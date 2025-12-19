# AWS IAM User Setup Guide for S3 Access

## What is IAM?

**IAM (Identity and Access Management)** is AWS's security service that helps you control who can access your AWS resources and what they can do with them. Think of it as a sophisticated security guard system for your cloud infrastructure.

Instead of sharing your main AWS account credentials (which would be like giving everyone the master key to your house), IAM lets you create separate users with specific permissions for specific tasks.

## Why Use IAM for Your Python Backend?

When your Python application needs to upload files to S3, it requires credentials to prove it has permission. Here's why IAM is essential:

- **Security**: Your Python code gets its own limited-access credentials instead of using your main AWS account
- **Control**: You can grant only the permissions needed (in this case, S3 access) and nothing more
- **Traceability**: All actions are logged under the IAM user's name, so you know who did what
- **Revocability**: If credentials are compromised, you can instantly disable just that IAM user without affecting anything else

### The Analogy

Think of your AWS account as a building:
- Your root account is the building owner with keys to everything
- IAM users are employees with keycards that only open specific doors
- Your Python backend is an employee that only needs access to the storage room (S3)

---

## Step-by-Step Guide: Creating an IAM User for S3 Upload

### Step 1: Navigate to IAM Service

1. Log in to your **AWS Console**
2. In the search bar at the top, type **IAM**
3. Click on **IAM** from the search results

### Step 2: Access Users Section

1. In the left sidebar, click on **Users**
2. Click the **Create user** button (orange button on the right)

### Step 3: Configure User Details

1. **User name**: Enter `user_name`
   - Use a descriptive name so you remember what this user is for
2. Click **Next**

### Step 4: Set Permissions (AS per requirements)

1. Under **Permissions options**, select **Attach policies directly**
2. In the search box, type `AmazonS3FullAccess`
3. Check the box next to **AmazonS3FullAccess**
   
   > **Note**: For production environments, it's recommended to create a custom policy with minimal permissions (principle of least privilege). However, `AmazonS3FullAccess` is standard for development and getting started quickly.

4. Click **Next**

### Step 5: Review and Create

1. Review your configuration:
   - User name: `user_name`
   - Permissions: `AmazonS3FullAccess`
2. Click **Create user**

### Step 6: Access the User

1. You'll see a success message
2. Click on the newly created user **user_name** from the users list
3. You'll be taken to the user's details page

### Step 7: Generate Access Keys

1. Click on the **Security credentials** tab
2. Scroll down to the **Access keys** section
3. Click **Create access key**

### Step 8: Select Use Case

1. Select **Local code** as the use case
   - This tells AWS you'll be using these credentials in your application code
2. Check the confirmation checkbox at the bottom
3. Click **Next**

### Step 9: Add Description (Optional)

1. You can add a description like "Python backend S3 upload credentials"
2. Click **Create access key**

### Step 10: Save Your Credentials

🚨 **CRITICAL STEP - READ CAREFULLY** 🚨

You'll now see two pieces of information:

- **Access key ID**: Something like `AKIAIOSFODNN7EXAMPLE`
- **Secret access key**: Something like `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`

**IMPORTANT**: 
- Copy both values immediately
- Save them in a secure location (password manager, environment variables file, etc.)
- **You cannot view the secret access key again after closing this page**
- If you lose it, you'll need to create a new access key

### Recommended: Download the CSV file containing these credentials for backup.

---

## Using These Credentials in Your Python Code

Once you have your credentials, you'll typically use them in your Python backend like this:

```python
import boto3

# Configure AWS credentials
s3_client = boto3.client(
    's3',
    aws_access_key_id='YOUR_ACCESS_KEY_ID',
    aws_secret_access_key='YOUR_SECRET_ACCESS_KEY',
    region_name='us-east-1'  # Change to your region
)

# Now you can upload files
s3_client.upload_file('local_file.txt', 'your-bucket-name', 'uploaded_file.txt')
```

### Best Practice: Use Environment Variables

Never hardcode credentials in your source code. Instead, use environment variables:

```python
import os
import boto3

s3_client = boto3.client(
    's3',
    aws_access_key_id=os.environ.get('AWS_ACCESS_KEY_ID'),
    aws_secret_access_key=os.environ.get('AWS_SECRET_ACCESS_KEY'),
    region_name=os.environ.get('AWS_REGION', 'us-east-1')
)
```

---

## Security Best Practices

✅ **DO:**
- Store credentials in environment variables or secrets management services
- Use IAM users with minimal required permissions
- Rotate access keys periodically (every 90 days recommended)
- Delete unused access keys
- Enable MFA (Multi-Factor Authentication) on your main AWS account

❌ **DON'T:**
- Commit credentials to Git repositories
- Share access keys via email or chat
- Use root account credentials in your code
- Give more permissions than necessary

---

## Troubleshooting

### "Access Denied" Errors
- Verify the IAM user has `AmazonS3FullAccess` policy attached
- Check that you're using the correct bucket name
- Ensure the bucket exists in the region you're connecting to

### "Invalid Credentials" Errors
- Double-check you copied the access key and secret key correctly
- Verify there are no extra spaces or characters
- Make sure the access key hasn't been deleted or deactivated

### Need to View/Reset Credentials
- You cannot view the secret key again after creation
- To get new credentials, create a new access key (you can have up to 2 active keys per user)
- Delete old access keys after rotating to new ones

---

## Next Steps

After setting up your IAM user:

1. Test the credentials in your Python code
2. Set up environment variables for secure credential storage
3. Configure your S3 bucket with appropriate permissions
4. Implement error handling for upload operations
5. Consider setting up CloudWatch logging to monitor S3 access

---

## Summary

You've successfully created an IAM user with S3 access permissions! This user acts as a secure bridge between your Python backend and AWS S3, allowing your application to upload files without exposing your main AWS account credentials.

Remember: Keep your credentials safe, rotate them regularly, and never commit them to version control.