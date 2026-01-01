Here is a complete, industry-standard documentation guide for solving the **Mixed Content** issue. You can save this as `MIXED_CONTENT_FIX.md` in your repository or share it with your team.

---

# 🛡️ Guide: Fixing "Mixed Content" (HTTPS to HTTP) Errors

## 🚨 The Problem
You have deployed your application using the following architecture:
*   **Frontend:** Hosted on **Vercel** (Served over **HTTPS** ✅).
*   **Backend:** Hosted on **AWS EC2 IP** (Served over **HTTP** ❌).

When the Frontend tries to call the Backend, the browser blocks the request and shows this error in the console:
> *Mixed Content: The page at 'https://my-app.vercel.app/' was loaded over HTTPS, but requested an insecure XMLHttpRequest endpoint 'http://13.52.10.10/api/login'. This request has been blocked; the content must be served over HTTPS.*

## 💡 The Solution: Vercel Rewrites (Proxy)
Instead of buying a domain and SSL certificate for your EC2 instance (which costs money/time), we use **Vercel Rewrites**.

Vercel acts as a **Secure Middleman**:
1.  **Browser** sends a request to Vercel (HTTPS 🔒).
2.  **Vercel** forwards the request to EC2 (HTTP 🔓) in the background.
3.  **Result:** The browser sees only HTTPS, so the error disappears.

---

## 🛠️ Implementation Steps

### Step 1: Create the Rewrite Configuration
Create a file named `vercel.json` in the root of your Frontend folder (e.g., `frontend/vercel.json`).

**`frontend/vercel.json`**
```json
{
  "rewrites": [
    {
      "source": "/api/:match*",
      "destination": "http://YOUR_EC2_IP_ADDRESS/api/:match*"
    }
  ]
}
```
*   **Replace** `YOUR_EC2_IP_ADDRESS` with your actual AWS Public IP (e.g., `54.123.45.67`).
*   **Note:** Do not include a port (like `:8000`) if you mapped Docker port 80 to 8000. Use standard HTTP port 80.

### Step 2: Update Frontend Environment Variables
Your React app must stop calling the EC2 IP directly. It should now call *itself* (Vercel).

1.  Go to **Vercel Dashboard** $\to$ **Settings** $\to$ **Environment Variables**.
2.  Find your API URL variable (e.g., `VITE_API_URL`).
3.  Change the value to a **relative path**:
    *   ❌ Old Value: `http://54.123.45.67/api`
    *   ✅ New Value: `/api`
4.  **Save**.

### Step 3: Update Backend CORS
Since the requests will now be proxied through Vercel, the backend sees the request originating from your Vercel Domain.

**`backend/app/main.py`**
```python
origins = [
    "http://localhost:5173",            # Localhost
    "https://your-project.vercel.app",  # <--- Add your Vercel Domain here
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    # ... rest of config
)
```

### Step 4: Redeploy Everything
Changes to Vercel Environment variables do not apply automatically. You must force a redeploy.

1.  **Push Code:** Push the `vercel.json` and `main.py` changes to GitHub.
    *   *This automatically updates the EC2 Backend.*
2.  **Redeploy Vercel:**
    *   Go to Vercel Dashboard $\to$ **Deployments**.
    *   Click the **three dots (...)** next to the latest deployment.
    *   Click **Redeploy**.

---

## ✅ Verification
1.  Open your Vercel App (`https://...`).
2.  Open **Developer Tools (F12)** $\to$ **Network Tab**.
3.  Perform an API action (e.g., Login).
4.  Inspect the Request URL.
    *   It should look like: `https://your-project.vercel.app/api/login`
    *   **Status:** 200 OK.
    *   **No Mixed Content Error** in the Console.

---

## ❓ Troubleshooting

**Q: I still see 404 Errors.**
*   **Check:** Does your backend route prefix match `vercel.json`?
*   If your backend uses `/api/v1/login`, your rewrite destination must handle that.
*   *Recommended:* Keep `destination` as `.../api/:match*` and Vercel will pass the full path.

**Q: I get 502 Bad Gateway.**
*   **Check:** Is your EC2 instance running?
*   **Check:** Is your Docker container running? (`docker ps`)
*   **Check:** Is Port 80 open in your AWS Security Group?

**Q: I get CORS Errors.**
*   **Check:** Did you add the **https** version of your Vercel domain to `main.py`?
*   **Check:** Did you redeploy the backend after changing `main.py`?