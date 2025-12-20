# Complete CI/CD Pipeline Guide - From Basics to Production

## What is CI/CD?

**CI/CD** stands for **Continuous Integration** and **Continuous Delivery/Deployment**. It's an automated approach to software development that allows teams to deliver code changes frequently, reliably, and with minimal manual intervention.

Think of CI/CD as an assembly line for software - code goes in one end, and fully tested, production-ready software comes out the other, all automatically.

### Step 2: Create Your First Pipeline

**For GitHub Actions**:

1. Create the workflow directory:
```bash
mkdir -p .github/workflows
```

2. Create a simple pipeline file `.github/workflows/ci.yml`:
```yaml
name: Simple CI Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
    
    - name: Run tests
      run: |
        pytest
```

3. Commit and push:
```bash
git add .github/workflows/ci.yml
git commit -m "Add CI pipeline"
git push
```

4. Watch it run:
   - Go to your GitHub repository
   - Click the "Actions" tab
   - See your pipeline running! 🎉

---

## CI/CD Best Practices

### 1. Keep Your Pipeline Fast ⚡

**Problem**: Slow pipelines = developers don't commit often

**Solutions**:
```yaml
# Cache dependencies
- uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}

# Run jobs in parallel
jobs:
  test-unit:
    runs-on: ubuntu-latest
    steps: [...]
  
  test-integration:
    runs-on: ubuntu-latest
    steps: [...]
  
  lint:
    runs-on: ubuntu-latest
    steps: [...]

# Use matrix testing
strategy:
  matrix:
    python-version: [3.9, 3.10, 3.11]
    os: [ubuntu-latest, macos-latest]
```

**Target**: Build + Test in under 10 minutes

---

### 2. Fail Fast 🚨

**Goal**: Catch problems early, give quick feedback

```yaml
# Run quick checks first
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: flake8 .  # Fast: 30 seconds
  
  unit-tests:
    needs: lint
    steps:
      - run: pytest tests/unit/  # Medium: 2 minutes
  
  integration-tests:
    needs: unit-tests
    steps:
      - run: pytest tests/integration/  # Slow: 5 minutes
  
  e2e-tests:
    needs: integration-tests
    steps:
      - run: npm run test:e2e  # Slowest: 10 minutes
```

---

### 3. Make Builds Deterministic 🎯

**Problem**: "It worked yesterday, why is it failing now?"

**Solutions**:

**Pin your dependencies**:
```txt
# ❌ Bad
flask

# ✅ Good
flask==3.0.0
```

**Use specific versions**:
```yaml
# ❌ Bad
- uses: actions/setup-python@v4

# ✅ Good
- uses: actions/setup-python@v4.5.0
```

**Lock your Docker base images**:
```dockerfile
# ❌ Bad
FROM python:3.11

# ✅ Good
FROM python:3.11.7-slim
```

---

### 4. Secure Your Secrets 🔒

**Never commit secrets to Git!**

**Use CI/CD platform's secret management**:

```yaml
# GitHub Actions
steps:
  - name: Deploy
    env:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
      API_KEY: ${{ secrets.API_KEY }}
    run: |
      echo "Deploying..."
```

**How to add secrets**:
- GitHub: Settings → Secrets → Actions
- GitLab: Settings → CI/CD → Variables
- Jenkins: Credentials → Add Credentials

**What to store as secrets**:
- Database passwords
- API keys
- SSH keys
- Docker registry credentials
- Cloud provider credentials

---

### 5. Test What You Deploy 📦

**Deploy the exact same artifact**:

```yaml
# Build once
build:
  steps:
    - run: docker build -t myapp:$VERSION .
    - run: docker push myapp:$VERSION

# Deploy the same image
deploy-staging:
  steps:
    - run: docker pull myapp:$VERSION
    - run: docker run myapp:$VERSION

deploy-production:
  steps:
    - run: docker pull myapp:$VERSION  # Same image!
    - run: docker run myapp:$VERSION
```

**Don't rebuild for each environment**:
```yaml
# ❌ Bad
deploy-staging:
  - run: npm run build  # Different build!
  - run: deploy

deploy-production:
  - run: npm run build  # Different build!
  - run: deploy

# ✅ Good
build:
  - run: npm run build
  - run: save artifacts

deploy-staging:
  - run: download artifacts
  - run: deploy

deploy-production:
  - run: download artifacts  # Same artifacts!
  - run: deploy
```

---

### 6. Monitor and Alert 📊

**Track pipeline metrics**:
- Build success rate
- Average build time
- Deployment frequency
- Mean time to recovery (MTTR)

**Set up notifications**:
```yaml
# Slack notifications
- name: Notify on failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    text: '❌ Build failed!'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}

# Email notifications
- name: Send email
  if: failure()
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    to: team@company.com
    subject: Build Failed
```

---

### 7. Version Everything 🏷️

**Tag your releases**:
```bash
git tag -a v1.2.3 -m "Release version 1.2.3"
git push origin v1.2.3
```

**Use semantic versioning**:
- v1.0.0 - Major release (breaking changes)
- v1.1.0 - Minor release (new features)
- v1.1.1 - Patch release (bug fixes)

**Trigger deployments on tags**:
```yaml
on:
  push:
    tags:
      - 'v*'  # Trigger on version tags

jobs:
  deploy:
    steps:
      - name: Get version
        run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_ENV
      
      - name: Deploy version ${{ env.VERSION }}
        run: deploy.sh
```

---

### 8. Implement Rollback Strategy 🔄

**Plan for failures**:

```yaml
deploy:
  steps:
    # Deploy new version
    - name: Deploy v2.0
      run: kubectl set image deployment/myapp myapp=myapp:v2.0
    
    # Wait and check health
    - name: Wait for deployment
      run: kubectl rollout status deployment/myapp --timeout=5m
    
    # Automatic rollback on failure
    - name: Rollback on failure
      if: failure()
      run: kubectl rollout undo deployment/myapp
```

**Keep previous versions available**:
```bash
# Don't delete old images
docker tag myapp:latest myapp:v2.0
docker tag myapp:v2.0 myapp:backup-$(date +%Y%m%d)
```

---

## Troubleshooting Common Issues

### Pipeline Fails: "Permission Denied"

**Problem**: CI/CD can't access resources

**Solutions**:
```yaml
# 1. Check file permissions
- run: chmod +x ./deploy.sh

# 2. Add SSH keys
- uses: webfactory/ssh-agent@v0.8.0
  with:
    ssh-private-key: ${{ secrets.SSH_KEY }}

# 3. Check secrets are set
- run: |
    if [ -z "${{ secrets.API_KEY }}" ]; then
      echo "API_KEY not set"
      exit 1
    fi
```

---

### Builds Are Too Slow

**Problem**: Pipeline takes 30+ minutes

**Solutions**:

**1. Use caching**:
```yaml
- uses: actions/cache@v3
  with:
    path: |
      ~/.cache/pip
      ~/.npm
      node_modules/
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
```

**2. Run jobs in parallel**:
```yaml
jobs:
  test-frontend:
    runs-on: ubuntu-latest
    steps: [...]
  
  test-backend:  # Runs simultaneously
    runs-on: ubuntu-latest
    steps: [...]
```

**3. Use faster runners**:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest-4-cores  # Faster machine
```

**4. Skip unnecessary steps**:
```yaml
- name: Run tests
  if: github.event_name == 'push'  # Skip on draft PRs
```

---

### Inconsistent Test Results

**Problem**: Tests pass locally, fail in CI (or vice versa)

**Solutions**:

**1. Use the same environment**:
```yaml
# Use Docker for consistency
- run: docker run --rm -v $(pwd):/app python:3.11 pytest
```

**2. Set explicit timezone**:
```yaml
env:
  TZ: America/New_York
```

**3. Avoid time-dependent tests**:
```python
# ❌ Bad
def test_timestamp():
    assert get_timestamp() == "2024-01-01"

# ✅ Good
def test_timestamp():
    timestamp = get_timestamp()
    assert isinstance(timestamp, str)
    assert len(timestamp) == 10
```

---

### Secrets Not Working

**Problem**: Environment variables are empty

**Solutions**:

**1. Check secret is set**:
- GitHub: Settings → Secrets and variables → Actions
- Ensure secret name matches exactly (case-sensitive!)

**2. Verify scope**:
```yaml
# ✅ Correct
env:
  API_KEY: ${{ secrets.API_KEY }}

# ❌ Wrong
run: echo $API_KEY  # Won't work without env:
```

**3. Don't print secrets**:
```yaml
# ❌ Bad - exposes secret in logs
- run: echo "API_KEY is ${{ secrets.API_KEY }}"

# ✅ Good
- run: |
    if [ -z "${{ secrets.API_KEY }}" ]; then
      echo "API_KEY not configured"
    else
      echo "API_KEY is set"
    fi
```

---

### Deployment Fails in Production

**Problem**: Works in staging, fails in production

**Solutions**:

**1. Make staging identical to production**:
```yaml
staging:
  environment:
    - DATABASE_URL: postgresql://...  # Same DB type
    - CACHE_URL: redis://...          # Same cache
    - NODE_ENV: production            # Same env mode

production:
  environment:
    - DATABASE_URL: postgresql://...
    - CACHE_URL: redis://...
    - NODE_ENV: production
```

**2. Use smoke tests**:
```yaml
deploy:
  steps:
    - run: deploy.sh
    - name: Smoke test
      run: |
        curl -f https://myapp.com/health || (rollback.sh && exit 1)
```

**3. Gradual rollout**:
```yaml
# Deploy to 10% of servers first
- run: deploy.sh --canary 10%
- run: sleep 300  # Wait 5 minutes
- run: check_error_rate.sh
- run: deploy.sh --full  # Deploy to all if no errors
```

---

## Advanced Topics

### Infrastructure as Code (IaC) in CI/CD

**Terraform Integration**:
```yaml
name: Terraform

on:
  push:
    branches: [ main ]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.6.0
      
      - name: Terraform Init
        run: terraform init
      
      - name: Terraform Plan
        run: terraform plan -out=tfplan
      
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve tfplan
```

---

### Database Migrations in CI/CD

**Automated migrations**:
```yaml
deploy:
  steps:
    # 1. Deploy new code
    - run: kubectl apply -f deployment.yaml
    
    # 2. Run migrations
    - name: Run database migrations
      run: |
        kubectl run migrations \
          --image=myapp:$VERSION \
          --restart=Never \
          --rm \
          --command -- python manage.py migrate
    
    # 3. Verify migrations
    - name: Check migration status
      run: |
        kubectl run check-migrations \
          --image=myapp:$VERSION \
          --restart=Never \
          --rm \
          --command -- python manage.py showmigrations
```

---

### Feature Flags

**Deploy without enabling**:
```python
# In code
if feature_flags.is_enabled('new_checkout_flow'):
    new_checkout()
else:
    old_checkout()
```

```yaml
# In CI/CD
deploy:
  steps:
    - run: deploy.sh
    - run: |
        # Feature is deployed but disabled
        feature-flag set new_checkout_flow false
    
    # Enable for 5% of users
    - run: feature-flag rollout new_checkout_flow 5%
    
    # Monitor...
    - run: sleep 3600
    
    # Enable for everyone if no issues
    - run: feature-flag rollout new_checkout_flow 100%
```

---

### Kubernetes Deployment

**Complete K8s CI/CD**:
```yaml
name: Deploy to Kubernetes

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Build Docker image
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
      
      # Push to registry
      - name: Push image
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push myapp:${{ github.sha }}
      
      # Setup kubectl
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
      
      # Update Kubernetes
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp \
            myapp=myapp:${{ github.sha }}
          
          kubectl rollout status deployment/myapp --timeout=5m
      
      # Rollback on failure
      - name: Rollback on failure
        if: failure()
        run: kubectl rollout undo deployment/myapp
```

---

## CI/CD Metrics to Track

### 1. Deployment Frequency 📈
**What**: How often you deploy to production

**Target**:
- Elite: Multiple times per day
- High: Once per day to once per week
- Medium: Once per week to once per month
- Low: Less than once per month

**Why it matters**: Higher frequency = faster feedback, smaller changes, less risk

---

### 2. Lead Time for Changes ⏱️
**What**: Time from code commit to code running in production

**Target**:
- Elite: Less than 1 hour
- High: 1 day to 1 week
- Medium: 1 week to 1 month
- Low: More than 1 month

**Why it matters**: Faster lead time = quicker value delivery

---

### 3. Mean Time to Recovery (MTTR) 🔧
**What**: Time to recover from a production failure

**Target**:
- Elite: Less than 1 hour
- High: Less than 1 day
- Medium: 1 day to 1 week
- Low: More than 1 week

**Why it matters**: Fast recovery = less user impact

---

### 4. Change Failure Rate ❌
**What**: Percentage of deployments that cause a failure

**Target**:
- Elite: 0-15%
- High: 16-30%
- Medium: 31-45%
- Low: More than 45%

**Why it matters**: Lower rate = more stable deployments

---

## Complete Project Setup Guide

### Project Structure with CI/CD

```
my-fullstack-app/
├── .github/
│   └── workflows/
│       ├── ci.yml              # CI pipeline
│       ├── cd-staging.yml      # Deploy to staging
│       └── cd-production.yml   # Deploy to production
├── backend/
│   ├── app/
│   ├── tests/
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .dockerignore
├── frontend/
│   ├── src/
│   ├── tests/
│   ├── package.json
│   ├── Dockerfile
│   └── .dockerignore
├── docker-compose.yml
├── k8s/                        # Kubernetes configs
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── terraform/                  # Infrastructure as Code
│   ├── main.tf
│   └── variables.tf
└── README.md
```

---

## Security in CI/CD 🔒

### 1. Scan Dependencies
```yaml
- name: Security scan
  run: |
    npm audit
    pip-audit
    snyk test
```

### 2. Scan Docker Images
```yaml
- name: Scan Docker image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: myapp:latest
    severity: 'CRITICAL,HIGH'
```

### 3. Static Application Security Testing (SAST)
```yaml
- name: SAST scan
  uses: github/codeql-action/analyze@v2
```

### 4. Secrets Scanning
```yaml
- name: Scan for secrets
  uses: trufflesecurity/trufflehog@main
  with:
    path: ./
```

### 5. License Compliance
```yaml
- name: Check licenses
  run: pip-licenses --fail-on="GPL;AGPL"
```

---

## Cost Optimization 💰

### GitHub Actions Free Tier

**Public repositories**: Unlimited

**Private repositories**:
- Free: 2,000 minutes/month
- Team: 3,000 minutes/month
- Enterprise: 50,000 minutes/month

**Multipliers by OS**:
- Linux: 1x
- Windows: 2x
- macOS: 10x

**Tips to save money**:

**1. Use Linux runners when possible**:
```yaml
# ✅ Costs 1 minute
runs-on: ubuntu-latest

# ❌ Costs 10 minutes
runs-on: macos-latest
```

**2. Use caching aggressively**:
```yaml
- uses: actions/cache@v3  # Saves rebuild time
```

**3. Run only necessary jobs**:
```yaml
jobs:
  test:
    if: github.event_name == 'push'  # Skip on draft PRs
```

**4. Self-host runners**:
```yaml
runs-on: self-hosted  # Use your own servers
```

---

## Summary

### What You've Learned

✅ **What CI/CD is**: Automated software delivery pipeline

✅ **Why it matters**: Faster delivery, fewer bugs, happier developers

✅ **How it works**: Source → Build → Test → Deploy

✅ **Pipeline creation**: Complete examples for multiple platforms

✅ **Best practices**: Fast builds, fail fast, secure secrets

✅ **Troubleshooting**: Common issues and solutions

✅ **Advanced topics**: Kubernetes, feature flags, metrics

---

### Your CI/CD Journey

**Week 1-2: Start Simple**
- Set up basic CI (build + test)
- Use GitHub Actions or GitLab CI
- Focus on getting tests running automatically

**Week 3-4: Add CD**
- Deploy to staging automatically
- Add manual approval for production
- Implement rollback strategy

**Month 2: Optimize**
- Add caching to speed up builds
- Implement parallel jobs
- Add security scanning

**Month 3+: Advanced**
- Blue-green deployments
- Feature flags
- Advanced monitoring
- Infrastructure as Code

---

### Key Takeaways

🎯 **Start small**: Don't try to implement everything at once

⚡ **Speed matters**: Fast feedback loops keep developers productive

🔒 **Security first**: Scan early, scan often

📊 **Measure everything**: Track metrics to improve

🔄 **Iterate constantly**: Your pipeline will evolve with your needs

---

### Next Steps

1. 🚀 **Choose a platform** (GitHub Actions recommended for beginners)
2. 📝 **Create your first pipeline** (start with build + test)
3. 🧪 **Add automated tests** (unit, integration, e2e)
4. 🚢 **Automate deployment** (staging first, then production)
5. 📊 **Monitor and improve** (track metrics, optimize)
6. 🎓 **Keep learning** (try advanced features)

---

## Quick Reference

### Essential CI/CD Commands

```bash
# GitHub Actions
gh workflow list
gh workflow view <workflow-name>
gh workflow run <workflow-name>

# GitLab CI
gitlab-ci-lint .gitlab-ci.yml
gitlab-runner verify

# Docker (for pipelines)
docker build -t myapp:test .
docker run --rm myapp:test pytest

# Kubernetes (for deployment)
kubectl apply -f deployment.yaml
kubectl rollout status deployment/myapp
kubectl rollout undo deployment/myapp
```

---

### Resources

**Documentation**:
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/)
- [Jenkins Docs](https://www.jenkins.io/doc/)

**Learning**:
- [GitHub Actions Learning Lab](https://lab.github.com/)
- [GitLab CI/CD Tutorial](https://docs.gitlab.com/ee/ci/quick_start/)
- [CI/CD Best Practices](https://docs.github.com/en/actions/learn-github-actions/best-practices-for-ci-cd)

**Community**:
- [DevOps Reddit](https://reddit.com/r/devops)
- [GitHub Community](https://github.com/community)
- [GitLab Forum](https://forum.gitlab.com/)

---

## Conclusion

CI/CD is not just a tool or a process - it's a fundamental shift in how we build and deliver software. By automating the repetitive, error-prone tasks, we free developers to focus on what they do best: solving problems and creating value.

**Remember**:
- CI/CD is a journey, not a destination
- Start simple and iterate
- Every team's pipeline is unique
- The goal is to deliver value faster and more reliably

Now go forth and automate! 🚀✨

**"The only way to go fast is to go well."** - Robert C. Martin Breaking Down the Acronym

**CI (Continuous Integration)**:
- Developers merge code changes into a central repository frequently (multiple times per day)
- Automated builds and tests run immediately after each merge
- Problems are detected early when they're easier and cheaper to fix

**CD (Continuous Delivery)**:
- Automatically prepares code for release to production
- Every change is production-ready but requires manual approval to deploy
- A human decides when to push to production

**CD (Continuous Deployment)**:
- Goes one step further than Continuous Delivery
- Automatically deploys every change that passes tests directly to production
- No human intervention required (fully automated)

---

## The Classic Problem: Manual Software Deployment 😰

### The Old Way (Without CI/CD)

**Monday Morning**:
```
Developer 1: "I finished the login feature!"
Developer 2: "I built the payment system!"
Developer 3: "I updated the database schema!"
```

**Friday Afternoon - Integration Day**:
```
Team: "Let's merge everything..."
*Merge conflicts everywhere*
*Tests failing*
*Database incompatible*
*Code doesn't compile*
```

**Next Week**:
```
Team: "Still fixing integration issues..."
Manager: "When can we release?"
Team: "Maybe next month... 😓"
```

### The Problems with Manual Deployment

1. **Integration Hell** 🔥
   - Multiple developers work in isolation for weeks
   - Merging code becomes a nightmare
   - Conflicts are hard to resolve
   - Testing happens too late

2. **"It Works On My Machine"** 💻
   - Code works on developer's laptop
   - Fails in testing environment
   - Breaks in production
   - Hard to reproduce bugs

3. **Slow Release Cycles** 🐌
   - Manual testing takes days/weeks
   - Deployment requires multiple teams
   - Releases happen monthly or quarterly
   - Bug fixes take too long to reach users

4. **High Risk Deployments** ⚠️
   - Big releases with many changes
   - If something breaks, hard to identify the cause
   - Rollbacks are complex and risky
   - Downtime affects many users

5. **Human Error** 🤦
   - Forgot to run a test
   - Deployed to wrong environment
   - Missed a configuration change
   - Copy-pasted wrong command

---

## How CI/CD Solves These Problems

### The Modern Way (With CI/CD)

**Every Day**:
```
Developer: Pushes code at 10 AM
CI/CD Pipeline: 
  ✅ Build: Success (2 min)
  ✅ Tests: All 1,247 tests passed (5 min)
  ✅ Deploy to staging: Success (1 min)
  ✅ Deploy to production: Success (1 min)

Developer: Feature is live by 10:10 AM! 🎉
```

### Benefits of CI/CD

**1. Fast Feedback** ⚡
- Know within minutes if your code works
- Catch bugs immediately
- Fix issues while context is fresh

**2. Reduced Risk** 🛡️
- Small, frequent changes are easier to debug
- Automated tests catch issues early
- Easy rollbacks if something goes wrong

**3. Faster Time to Market** 🚀
- Deploy multiple times per day
- Features reach users immediately
- Quick bug fixes
- Competitive advantage

**4. Better Collaboration** 🤝
- Everyone works on the same codebase
- Integration happens continuously
- Less merge conflicts
- Clear visibility into what's deployed

**5. Higher Quality** ✨
- Automated testing ensures consistency
- No "forgot to test" scenarios
- Security scans catch vulnerabilities
- Code quality checks enforced

**6. Developer Happiness** 😊
- Less manual, repetitive work
- More time for creative problem-solving
- Faster deployment means faster feedback
- Reduced stress around releases

---

## Real-World Impact: The Numbers

Organizations that have mastered CI/CD deploy 208 times more frequently and have a lead time that is 106 times faster compared to those without it.

**Before CI/CD**:
- Deploy: Once per month
- Lead time: 1-3 months
- Failure rate: 20-30%
- Recovery time: Days

**After CI/CD**:
- Deploy: Multiple times per day
- Lead time: Hours to days
- Failure rate: 0-5%
- Recovery time: Minutes

---

## Understanding the CI/CD Pipeline

A CI/CD pipeline guides the process of software development through a path of building, testing, and deploying code.

### The Pipeline Stages

Think of a CI/CD pipeline as a quality gate - your code must pass through multiple checkpoints before reaching production.

```
┌──────────────────────────────────────────────────────────────────┐
│                       CI/CD PIPELINE                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. SOURCE    →   2. BUILD   →   3. TEST   →   4. DEPLOY        │
│                                                                   │
│  Developer        Compile       Unit Tests      Staging          │
│  pushes code      Package       Integration     Production       │
│                   Create        Security        Monitoring       │
│                   artifacts     Performance                      │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

Let's explore each stage in detail:

---

### Stage 1: Source (Code Repository)

**What Happens**: Developer commits code to a version control system.

**Tools**: Git, GitHub, GitLab, Bitbucket

**Triggers**:
- Push to main/master branch
- Pull request created
- Tag created (for releases)
- Scheduled (nightly builds)

**Example**:
```bash
# Developer workflow
git add .
git commit -m "Add user authentication feature"
git push origin main

# This push automatically triggers the CI/CD pipeline
```

**Best Practices**:
- Commit small, focused changes frequently
- Write clear commit messages
- Use feature branches for development
- Merge to main branch often (trunk-based development)

---

### Stage 2: Build

**What Happens**: The CI/CD system compiles the source code into executable or deployable artifacts.

**Activities**:
- Compile source code
- Resolve dependencies
- Run linters and code quality checks
- Create Docker images
- Generate build artifacts

**Example - Python Flask App**:
```yaml
build:
  script:
    - pip install -r requirements.txt
    - python -m py_compile app.py
    - docker build -t myapp:$VERSION .
```

**Example - Node.js App**:
```yaml
build:
  script:
    - npm ci
    - npm run build
    - npm run lint
```

**Example - React App**:
```yaml
build:
  script:
    - npm install
    - npm run build
    - # Creates production-ready static files
```

**Why It Matters**:
- Catches syntax errors immediately
- Verifies all dependencies are available
- Ensures code compiles before testing
- Creates reproducible build artifacts

---

### Stage 3: Test

**What Happens**: Continuous testing is typically performed automatically, with each code change triggering a series of tests.

**Types of Tests**:

**1. Unit Tests** 🧪
- Test individual functions/components
- Fast (milliseconds)
- Run on every commit
```python
def test_add_user():
    user = create_user("john@example.com")
    assert user.email == "john@example.com"
```

**2. Integration Tests** 🔗
- Test how components work together
- Moderate speed (seconds)
- Test database connections, API calls
```python
def test_user_login():
    response = client.post('/login', data={
        'email': 'test@example.com',
        'password': 'password123'
    })
    assert response.status_code == 200
```

**3. End-to-End (E2E) Tests** 🎭
- Test complete user workflows
- Slower (minutes)
- Simulate real user behavior
```javascript
test('user can complete checkout', async () => {
  await page.goto('/products')
  await page.click('.add-to-cart')
  await page.click('.checkout')
  await page.fill('#card-number', '4242424242424242')
  await page.click('.submit-payment')
  await expect(page).toHaveURL('/order-confirmation')
})
```

**4. Security Tests** 🔒
- Scan for vulnerabilities
- Check dependencies for known issues
- Test authentication and authorization
```bash
npm audit
snyk test
```

**5. Performance Tests** ⚡
- Load testing
- Stress testing
- Response time checks
```yaml
performance:
  script:
    - ab -n 1000 -c 10 http://localhost:8000/
```

**Example Test Stage**:
```yaml
test:
  script:
    # Unit tests
    - pytest tests/unit/
    
    # Integration tests
    - pytest tests/integration/
    
    # Code coverage
    - pytest --cov=app tests/
    
    # Security scan
    - bandit -r app/
    
    # E2E tests
    - npm run test:e2e
```

---

### Stage 4: Deploy

**What Happens**: Code is deployed to various environments.

**Environments**:

**1. Development** 💻
- Developer's local environment
- Used for active development
- May be unstable

**2. Staging/Testing** 🧪
- Mirror of production environment
- Used for QA testing
- Test with production-like data
- Final validation before production

**3. Production** 🌍
- Live environment serving real users
- Must be stable and reliable
- Monitored 24/7

**Deployment Strategies**:

**A. Blue-Green Deployment**
```
Blue (Current)     Green (New)
    ↓                  ↓
[v1.0 100%]      [v2.0 0%]
                      
Switch traffic →
                      
[v1.0 0%]        [v2.0 100%]
```
- Zero downtime
- Easy rollback
- Requires double infrastructure

**B. Canary Deployment**
```
Day 1: v2.0 → 5% of users
Day 2: v2.0 → 25% of users
Day 3: v2.0 → 100% of users
```
- Gradual rollout
- Monitor for issues
- Minimize blast radius

**C. Rolling Deployment**
```
Server 1: v1.0 → v2.0 ✅
Server 2: v1.0 → v2.0 ✅
Server 3: v1.0 → v2.0 ✅
```
- Update servers one by one
- No downtime
- Some users see old, some see new

---

## CI/CD Pipeline in Action: Complete Examples

### Example 1: GitHub Actions - Python Flask Application

**Project Structure**:
```
my-flask-app/
├── app.py
├── requirements.txt
├── tests/
│   ├── test_app.py
│   └── test_integration.py
├── Dockerfile
└── .github/
    └── workflows/
        └── ci-cd.yml
```

**.github/workflows/ci-cd.yml**:
```yaml
name: CI/CD Pipeline

# Trigger on push to main branch or pull request
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  DOCKER_IMAGE: myapp
  VERSION: ${{ github.sha }}

jobs:
  # Job 1: Build and Test
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    # Step 1: Checkout code
    - name: Checkout code
      uses: actions/checkout@v3
    
    # Step 2: Set up Python
    - name: Set up Python 3.11
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    # Step 3: Cache dependencies
    - name: Cache pip packages
      uses: actions/cache@v3
      with:
        path: ~/.cache/pip
        key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
    
    # Step 4: Install dependencies
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest pytest-cov flake8
    
    # Step 5: Lint code
    - name: Lint with flake8
      run: |
        flake8 app.py --max-line-length=120
    
    # Step 6: Run unit tests
    - name: Run unit tests
      run: |
        pytest tests/test_app.py -v
    
    # Step 7: Run integration tests
    - name: Run integration tests
      run: |
        pytest tests/test_integration.py -v
    
    # Step 8: Generate coverage report
    - name: Code coverage
      run: |
        pytest --cov=app tests/ --cov-report=xml
    
    # Step 9: Upload coverage to Codecov
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
  
  # Job 2: Security Scan
  security:
    runs-on: ubuntu-latest
    needs: build-and-test
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Run security scan
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'table'
  
  # Job 3: Build Docker Image
  docker-build:
    runs-on: ubuntu-latest
    needs: [build-and-test, security]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          ${{ secrets.DOCKER_USERNAME }}/${{ env.DOCKER_IMAGE }}:latest
          ${{ secrets.DOCKER_USERNAME }}/${{ env.DOCKER_IMAGE }}:${{ env.VERSION }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
  
  # Job 4: Deploy to Staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: docker-build
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.myapp.com
    
    steps:
    - name: Deploy to staging server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.STAGING_HOST }}
        username: ${{ secrets.STAGING_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          docker pull ${{ secrets.DOCKER_USERNAME }}/${{ env.DOCKER_IMAGE }}:latest
          docker stop myapp-staging || true
          docker rm myapp-staging || true
          docker run -d \
            --name myapp-staging \
            -p 8000:8000 \
            -e DATABASE_URL=${{ secrets.STAGING_DB_URL }} \
            ${{ secrets.DOCKER_USERNAME }}/${{ env.DOCKER_IMAGE }}:latest
    
    - name: Run smoke tests
      run: |
        sleep 10
        curl -f https://staging.myapp.com/health || exit 1
  
  # Job 5: Deploy to Production
  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.com
    
    steps:
    - name: Deploy to production server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.PROD_HOST }}
        username: ${{ secrets.PROD_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          docker pull ${{ secrets.DOCKER_USERNAME }}/${{ env.DOCKER_IMAGE }}:${{ env.VERSION }}
          docker stop myapp-prod || true
          docker rm myapp-prod || true
          docker run -d \
            --name myapp-prod \
            -p 80:8000 \
            -e DATABASE_URL=${{ secrets.PROD_DB_URL }} \
            --restart unless-stopped \
            ${{ secrets.DOCKER_USERNAME }}/${{ env.DOCKER_IMAGE }}:${{ env.VERSION }}
    
    - name: Verify deployment
      run: |
        sleep 15
        curl -f https://myapp.com/health || exit 1
    
    - name: Notify team
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: '🚀 Deployed to production successfully!'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      if: success()
```

---

### Example 2: GitLab CI - Node.js Application

**.gitlab-ci.yml**:
```yaml
# Define pipeline stages
stages:
  - build
  - test
  - security
  - deploy

# Global variables
variables:
  NODE_VERSION: "18"
  DOCKER_IMAGE: "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"

# Cache node_modules
cache:
  paths:
    - node_modules/

# Build stage
build:
  stage: build
  image: node:18
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
  only:
    - main
    - merge_requests

# Unit tests
test:unit:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run test:unit
  coverage: '/Statements\s*:\s*([^%]+)/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Integration tests
test:integration:
  stage: test
  image: node:18
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_DB: test_db
    POSTGRES_USER: test_user
    POSTGRES_PASSWORD: test_pass
    DATABASE_URL: "postgresql://test_user:test_pass@postgres:5432/test_db"
    REDIS_URL: "redis://redis:6379"
  script:
    - npm ci
    - npm run test:integration

# E2E tests
test:e2e:
  stage: test
  image: mcr.microsoft.com/playwright:latest
  script:
    - npm ci
    - npm run build
    - npm run test:e2e
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 1 week

# Code quality
code_quality:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run lint
    - npm run type-check

# Security scanning
security:dependency:
  stage: security
  image: node:18
  script:
    - npm audit --audit-level=moderate
  allow_failure: true

security:sast:
  stage: security
  image:
    name: returntocorp/semgrep
  script:
    - semgrep --config=auto .
  allow_failure: true

# Docker build
docker:build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main

# Deploy to staging
deploy:staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
  script:
    - |
      ssh -o StrictHostKeyChecking=no $STAGING_USER@$STAGING_HOST << EOF
        docker pull $DOCKER_IMAGE
        docker stop myapp-staging || true
        docker rm myapp-staging || true
        docker run -d \
          --name myapp-staging \
          -p 3000:3000 \
          -e NODE_ENV=staging \
          -e DATABASE_URL=$STAGING_DB_URL \
          --restart unless-stopped \
          $DOCKER_IMAGE
      EOF
    - sleep 10
    - curl -f http://$STAGING_HOST:3000/health || exit 1
  environment:
    name: staging
    url: https://staging.myapp.com
  only:
    - main

# Deploy to production (manual approval required)
deploy:production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  script:
    - |
      ssh -o StrictHostKeyChecking=no $PROD_USER@$PROD_HOST << EOF
        docker pull $DOCKER_IMAGE
        
        # Blue-green deployment
        docker stop myapp-prod-green || true
        docker rm myapp-prod-green || true
        docker run -d \
          --name myapp-prod-green \
          -p 3001:3000 \
          -e NODE_ENV=production \
          -e DATABASE_URL=$PROD_DB_URL \
          --restart unless-stopped \
          $DOCKER_IMAGE
        
        # Health check
        sleep 10
        curl -f http://localhost:3001/health || exit 1
        
        # Switch traffic
        docker stop myapp-prod-blue || true
        docker rm myapp-prod-blue || true
        docker rename myapp-prod-green myapp-prod-blue
      EOF
  environment:
    name: production
    url: https://myapp.com
  when: manual
  only:
    - main
```

---

### Example 3: Jenkins Pipeline - Java Spring Boot

**Jenkinsfile**:
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "myapp"
        VERSION = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = "docker.io"
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh './mvnw test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Integration Tests') {
            steps {
                sh './mvnw verify -Dskip.unit.tests=true'
            }
        }
        
        stage('Code Quality') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh './mvnw dependency-check:check'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${VERSION}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        docker.image("${DOCKER_IMAGE}:${VERSION}").push()
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                script {
                    sshagent(['staging-ssh-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no user@staging-server << 'EOF'
                            docker pull ${DOCKER_IMAGE}:${VERSION}
                            docker stop myapp-staging || true
                            docker rm myapp-staging || true
                            docker run -d \\
                                --name myapp-staging \\
                                -p 8080:8080 \\
                                -e SPRING_PROFILES_ACTIVE=staging \\
                                ${DOCKER_IMAGE}:${VERSION}
                        EOF
                        """
                    }
                }
            }
        }
        
        stage('Smoke Tests') {
            steps {
                sh 'sleep 15'
                sh 'curl -f http://staging-server:8080/actuator/health || exit 1'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                script {
                    sshagent(['prod-ssh-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no user@prod-server << 'EOF'
                            docker pull ${DOCKER_IMAGE}:${VERSION}
                            docker stop myapp-prod || true
                            docker rm myapp-prod || true
                            docker run -d \\
                                --name myapp-prod \\
                                -p 80:8080 \\
                                -e SPRING_PROFILES_ACTIVE=production \\
                                --restart unless-stopped \\
                                ${DOCKER_IMAGE}:${VERSION}
                        EOF
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            slackSend(
                color: 'good',
                message: "✅ Pipeline succeeded: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "❌ Pipeline failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }
        always {
            cleanWs()
        }
    }
}
```

---

## Popular CI/CD Tools Comparison

### 1. GitHub Actions

**Pros**:
- ✅ Tightly integrated with GitHub
- ✅ Free for public repositories
- ✅ 2,000 free minutes/month for private repos
- ✅ Huge marketplace of actions
- ✅ Easy to set up (YAML in repo)
- ✅ Matrix builds (test multiple versions)

**Cons**:
- ❌ Can get expensive for heavy usage
- ❌ Limited to GitHub
- ❌ Less flexible than Jenkins

**Best For**: GitHub-hosted projects, open source, startups

**Example**:
```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: npm test
```

---

### 2. GitLab CI/CD

**Pros**:
- ✅ Built into GitLab
- ✅ Excellent Docker support
- ✅ Free tier with 400 minutes/month
- ✅ Auto DevOps feature
- ✅ Strong security scanning
- ✅ Can use your own runners

**Cons**:
- ❌ Learning curve for complex pipelines
- ❌ Limited to GitLab

**Best For**: Teams using GitLab, enterprise, security-focused

**Example**:
```yaml
stages:
  - build
  - test

build:
  stage: build
  script:
    - npm install
    - npm run build

test:
  stage: test
  script:
    - npm test
```

---

### 3. Jenkins

**Pros**:
- ✅ Free and open source
- ✅ Highly customizable
- ✅ Huge plugin ecosystem (1,800+)
- ✅ Self-hosted (full control)
- ✅ Supports any VCS
- ✅ Industry standard

**Cons**:
- ❌ Requires setup and maintenance
- ❌ UI feels dated
- ❌ Steep learning curve
- ❌ Need to manage infrastructure

**Best For**: Enterprise, complex workflows, on-premise

---

### 4. CircleCI

**Pros**:
- ✅ Fast builds
- ✅ Excellent Docker support
- ✅ Free tier: 6,000 minutes/month
- ✅ Great documentation
- ✅ SSH debugging into builds

**Cons**:
- ❌ Can get expensive
- ❌ Less flexible than Jenkins

**Best For**: Docker-heavy workflows, performance-critical

---

### 5. Travis CI

**Pros**:
- ✅ Easy setup
- ✅ Free for open source
- ✅ Good GitHub integration
- ✅ Simple YAML configuration

**Cons**:
- ❌ Slower builds than competitors
- ❌ Limited free tier for private repos

**Best For**: Open source projects, simple workflows

---

### 6. Azure Pipelines

**Pros**:
- ✅ Free for open source (unlimited minutes)
- ✅ 1,800 free minutes/month for private
- ✅ Great Windows/Azure integration
- ✅ Supports any platform

**Cons**:
- ❌ Complex interface
- ❌ Azure-centric

**Best For**: Microsoft stack, Azure users, enterprise

---

## Setting Up Your First CI/CD Pipeline

### Step 1: Choose a CI/CD Platform

For beginners, I recommend **GitHub Actions** because:
- It's free to start
- Easy to set up (just add a YAML file)
- Great documentation
- Works with any language/framework

###