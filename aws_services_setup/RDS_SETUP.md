# AWS RDS Database Setup Guide

## What is Amazon RDS?

**Amazon RDS (Relational Database Service)** is a fully managed database service that makes it easy to set up, operate, and scale relational databases in the cloud. Think of it as having a professional database administrator (DBA) working 24/7 to manage your database, without actually hiring one.

RDS automates time-consuming tasks like hardware provisioning, database setup, patching, backups, and recovery, allowing you to focus on building your application instead of managing infrastructure.

### Key Features:
- **Fully Managed**: AWS handles maintenance, backups, and updates automatically
- **High Availability**: Built-in redundancy and failover capabilities
- **Scalability**: Easily scale compute and storage resources
- **Security**: Encryption at rest and in transit, network isolation
- **Automated Backups**: Point-in-time recovery and automated snapshots
- **Multiple Engines**: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora

## Why Use RDS Instead of Self-Managed Databases?

### Traditional Self-Managed Database vs RDS:

**❌ Self-Managed Database (on EC2 or your own server):**
- You handle all OS updates and security patches
- Manual backup configuration and management
- No automatic failover - downtime during hardware failures
- Complex scaling requires downtime
- You're responsible for performance tuning
- Need to monitor disk space manually
- Database crashes = you wake up at 3 AM

**✅ Using RDS:**
- Automatic OS updates and security patches
- Automated backups with point-in-time recovery
- Multi-AZ deployment for automatic failover (99.95% SLA)
- Scale compute/storage with a few clicks (minimal downtime)
- AWS handles performance optimization
- Storage auto-scaling available
- AWS monitors and alerts you proactively

### Cost Comparison:

**Self-Managed**: EC2 instance + your time (hours of maintenance) + risk of downtime

**RDS**: Slightly higher per-hour cost but includes management, backups, monitoring, and peace of mind

### Real-World Scenarios:

✅ **Use RDS when:**
- You want to focus on application development
- You need high availability and automatic backups
- You want predictable performance without DBA expertise
- You're building production applications
- You need compliance features (encryption, audit logs)

❌ **Don't use RDS when:**
- You need full control over the database engine internals
- You're using unsupported database engines
- You need custom OS-level configurations
- Budget is extremely tight and you have DBA expertise in-house

---

## Supported Database Engines

RDS supports six popular database engines:

| Engine | Best For | Free Tier |
|--------|----------|-----------|
| **MySQL** | Web applications, general purpose | ✅ Yes |
| **PostgreSQL** | Advanced features, complex queries | ✅ Yes |
| **MariaDB** | MySQL alternative, open-source | ✅ Yes |
| **Oracle** | Enterprise applications, legacy systems | ❌ No |
| **SQL Server** | Microsoft ecosystem, .NET apps | ❌ No |
| **Aurora** | High performance, cloud-native | ❌ No |

For this guide, we'll use **MySQL** as it's the most popular choice for web applications and offers the best free tier support.

---

## Phase 1: Create the RDS Database Instance

### Step 1: Navigate to RDS Service

1. Log in to your **AWS Console**
2. In the search bar at the top, type **RDS**
3. Click on **Amazon RDS** from the search results
4. **Important**: Check the AWS Region in the top-right corner
   - Select the same region as your other resources (S3, IAM users)
   - Example: `us-east-1` (N. Virginia)
   - Keeping everything in the same region reduces latency and costs

### Step 2: Initiate Database Creation

1. In the left sidebar, click on **Databases**
2. Click the **Create database** button (orange button on the right)

### Step 3: Choose Database Creation Method

You'll see two options:

**Standard Create** (Recommended for learning)
- ✅ Full control over all configuration options
- ✅ Learn what each setting does
- ✅ Better for production environments
- **Select this option**

**Easy Create**
- Simplified setup with default values
- Good for quick testing but less educational

> **💡 Recommendation**: Choose **Standard Create** so you understand each configuration decision.

### Step 4: Select Database Engine

1. **Engine type**: Choose **MySQL**
   - Most popular open-source database
   - Excellent documentation and community support
   - Works great with Python, Node.js, PHP, Java

2. **Version**: Select the latest stable version
   - Example: `MySQL 8.0.39`
   - Newer versions have better performance and security
   - Unless you have a specific compatibility requirement, use the latest

### Step 5: Choose Template

AWS offers three templates that pre-configure settings for different use cases:

| Template | When to Use | Multi-AZ | Instance Size |
|----------|-------------|----------|---------------|
| **Production** | Live applications serving real users | Yes (high availability) | Larger instances |
| **Dev/Test** | Development and testing environments | No (single instance) | Medium instances |
| **Free Tier** | Learning, experimentation, small projects | No | db.t3.micro |

**Select: Free Tier** (if available)
- Perfect for learning and development
- 750 hours/month free for 12 months
- 20 GB of storage included
- No credit card charges within limits

> **Note**: If you don't see "Free Tier," it means either:
> - Your account is older than 12 months
> - You've exhausted the free tier
> - Select "Dev/Test" as the next best option

### Step 6: Configure Settings

#### DB Instance Identifier
**Name**: `newgate-app-database`
- This is how you'll identify your database in the AWS console
- Choose something descriptive
- Examples: `myapp-prod-db`, `ecommerce-database`, `blog-mysql-db`
- Can only contain alphanumeric characters and hyphens

#### Credentials Settings

1. **Master username**: `admin`
   - This is the superuser account for your database
   - Can be any name, but `admin` or `root` is common
   - You'll use this to create other database users later

2. **Credentials management**: Select **Self managed**
   - You set the password yourself
   - Alternative: AWS Secrets Manager (for production, costs extra)

3. **Master password**: Create a strong password
   - Must be 8-41 characters
   - Include uppercase, lowercase, numbers, and symbols
   - **CRITICAL**: Save this password securely - you'll need it to connect
   - Example format: `MySecurePass123!@#`

4. **Confirm password**: Re-enter the password

> 🔒 **Security Tip**: Use a password manager to generate and store this password. Losing it means recreating the database!

### Step 7: Configure Instance

#### DB Instance Class

The instance class determines your database's CPU, memory, and network performance.

**For Free Tier**:
- `db.t3.micro` or `db.t2.micro` (automatically selected)
- 1 vCPU, 1 GB RAM
- Sufficient for small applications and learning

**For Production** (examples):
- `db.t3.small`: 2 vCPU, 2 GB RAM - small apps
- `db.m5.large`: 2 vCPU, 8 GB RAM - medium apps
- `db.r5.xlarge`: 4 vCPU, 32 GB RAM - large apps with heavy queries

> **💡 Note**: You can change the instance class later with minimal downtime (usually 2-5 minutes).

### Step 8: Configure Storage

#### Storage Type

**General Purpose SSD (gp3)** - Recommended
- ✅ Best balance of price and performance
- ✅ Suitable for most workloads
- 3000 IOPS baseline

**Provisioned IOPS (io1)** - For high performance
- More expensive
- Use when you need consistent high I/O performance

**Select**: `General Purpose SSD (gp3)`

#### Allocated Storage

**For Free Tier**: 20 GB (free tier limit)
**For Production**: Start with what you need, can increase later

> **Important**: You can increase storage anytime without downtime, but you **cannot** decrease it.

#### Storage Autoscaling

✅ **Enable storage autoscaling** (Recommended)
- AWS automatically increases storage when you reach a threshold
- Prevents "disk full" errors
- **Maximum storage threshold**: Set to `100 GB` (or your budget limit)

### Step 9: Configure Availability & Durability

#### Multi-AZ Deployment

**Single DB instance** (Free Tier / Dev)
- Database runs in one Availability Zone
- Lower cost
- Acceptable downtime during maintenance

**Multi-AZ deployment** (Production)
- Standby replica in another Availability Zone
- Automatic failover (typically under 60 seconds)
- 99.95% availability SLA
- Costs ~2x but worth it for production

**Select**: `Single DB instance` (for learning)

> **Production Tip**: Always use Multi-AZ for production databases. The cost is worth the peace of mind.

### Step 10: Configure Connectivity

This is one of the most important sections - it determines how your application connects to the database.

#### Compute Resource

**Don't connect to an EC2 compute resource** (Select this)
- You'll manually configure connections later
- More flexible for Python backends, Lambda functions, etc.

#### Network Type

**IPv4** (Default and recommended)

#### Virtual Private Cloud (VPC)

**Select**: `Default VPC`
- If you have a custom VPC, select that instead
- VPC provides network isolation for your database

#### DB Subnet Group

**Select**: `default` (or your custom subnet group)
- Ensures your database can be reached from your application

#### Public Access

This is a critical security decision:

**🚨 For Production / Public Internet Access**:
- **Publicly accessible**: `Yes`
- Allows connections from your Python backend on external servers
- Still secured by username/password and security groups
- Use when:
  - Your backend is on Heroku, DigitalOcean, or other cloud providers
  - You need to connect from your local machine during development
  - You're using serverless functions (AWS Lambda)

**🔒 For Maximum Security / Private Network Only**:
- **Publicly accessible**: `No`
- Only accessible from within the VPC
- More secure but requires VPN or EC2 bastion host
- Use when:
  - Your backend is on AWS EC2 in the same VPC
  - You want maximum security and have VPN access

**Select**: `Yes` (for learning and external backend access)

#### VPC Security Group

Security groups act as a firewall for your database.

**Create new** (Recommended for first time)
- AWS creates a security group automatically
- Name: `rds-sg-newgate-app`

**Choose existing** (If you have configured security groups)
- Select your pre-configured security group

**Important**: We'll configure inbound rules in Phase 2.

#### Availability Zone

**No preference** (Recommended)
- AWS automatically chooses the best zone
- Only specify if you have specific requirements

#### Database Port

**Default**: `3306` (for MySQL)
- PostgreSQL: `5432`
- SQL Server: `1433`
- Oracle: `1521`

**Keep the default unless you have a reason to change it.**

### Step 11: Database Authentication

**Password authentication** (Select this)
- Traditional username/password authentication
- Simplest to set up and use

**IAM database authentication** (Advanced)
- Uses AWS IAM credentials
- More secure but more complex setup

**Kerberos authentication** (Enterprise)
- For organizations using Active Directory

**Select**: `Password authentication`

### Step 12: Additional Configuration

Click **Additional configuration** to expand this section.

#### Initial Database Name

**Database name**: `newgate_app_db`
- This creates a database automatically on launch
- You can create more databases later
- Use underscores, not hyphens
- **Important**: If you don't specify this, no database is created, and you'll have to create one manually later

> **💡 Tip**: Create the initial database - it saves you a manual step later.

#### DB Parameter Group

**Default**: `default.mysql8.0` (or your version)
- Contains engine configuration settings
- Keep default unless you need custom configurations

#### Option Group

**Default**: `default:mysql-8-0`
- Advanced database features
- Keep default for now

#### Backup

**Enable automatic backups**: ✅ Yes (Highly recommended)
- **Backup retention period**: `7 days` (Free tier allows up to 7 days)
- Point-in-time recovery available
- **Backup window**: `No preference` (AWS chooses low-traffic times)

**Copy tags to snapshots**: ✅ Yes
- Helps organize your backups

> **Production Tip**: Set retention to 30 days for production databases.

#### Encryption

**Enable encryption**: ✅ Yes (Recommended)
- Encrypts your database at rest
- Uses AWS KMS (Key Management Service)
- **Free tier compatible**
- No performance impact
- Keep the default `(Default) aws/rds` key

#### Performance Insights

**Enable Performance Insights**: Optional
- Advanced performance monitoring
- Free tier: 7 days retention
- Useful for troubleshooting slow queries
- **Select**: `Yes` (if available in free tier)

#### Monitoring

**Enable Enhanced Monitoring**: Optional
- Detailed OS-level metrics
- Requires IAM role creation
- Costs a bit more
- **Select**: `No` (for free tier)

#### Log Exports

**Select logs to publish to CloudWatch**:
- ✅ Error log (Recommended)
- ✅ Slow query log (Recommended for optimization)
- General log (Very verbose, usually not needed)
- Audit log (For compliance requirements)

#### Maintenance

**Enable auto minor version upgrade**: ✅ Yes
- AWS automatically applies security patches
- Applied during maintenance window
- **Maintenance window**: `No preference`

#### Deletion Protection

**Enable deletion protection**: ❌ No (for learning)
- For production: ✅ **Yes** (prevents accidental deletion)
- You must disable this before you can delete the database

### Step 13: Estimate Cost

Scroll down to see **Estimated monthly costs**
- Free tier: $0.00 (if within limits)
- Otherwise: Shows approximate cost

### Step 14: Create Database

1. Review all settings carefully
2. Click **Create database** (orange button)
3. You'll see a success message

**Creation Time**: 5-15 minutes
- Status shows: `Creating`
- AWS is provisioning compute, storage, and network resources
- Be patient - grab a coffee! ☕

---

## Phase 2: Configure Security Group Rules

Even though we created the database, we need to configure the security group to allow connections from your application.

### Why Configure Security Groups?

Think of security groups as bouncers at a club - they decide who gets in. By default, RDS security groups block **all inbound traffic**. We need to explicitly allow connections from your Python backend.

### Step 1: Find Your Database Endpoint

1. Go to **RDS Console** → **Databases**
2. Wait for status to change from `Creating` to `Available`
3. Click on your database name: `newgate-app-database`
4. In the **Connectivity & security** tab, find:
   - **Endpoint**: Something like `newgate-app-database.abc123xyz.us-east-1.rds.amazonaws.com`
   - **Port**: `3306`

> 📝 **IMPORTANT**: Copy and save these values - you'll need them to connect!

### Step 2: Access Security Group

1. On the same page, scroll to **Security** section
2. Under **VPC security groups**, click on the security group link
   - Example: `rds-sg-newgate-app (sg-0abc123def456)`
3. You'll be taken to the EC2 Security Groups page

### Step 3: Edit Inbound Rules

1. Click on the **Inbound rules** tab
2. Click **Edit inbound rules**

### Step 4: Add Rules for Your Application

You'll need to add rules based on where your Python backend is hosted:

#### Option A: Allow Your Local Computer (For Development)

**Best for**: Testing from your laptop

1. Click **Add rule**
2. Configure:
   - **Type**: `MySQL/Aurora` (auto-fills port 3306)
   - **Source**: `My IP` (AWS auto-detects your current IP)
   - **Description**: `My development machine`

> ⚠️ **Warning**: Your home IP may change. If you lose connection, come back and update this rule.

#### Option B: Allow Specific IP Address/Range

**Best for**: Server with static IP

1. Click **Add rule**
2. Configure:
   - **Type**: `MySQL/Aurora`
   - **Source**: `Custom`
   - **CIDR**: Enter your server's IP address in CIDR format
     - Single IP: `203.0.113.25/32`
     - IP range: `203.0.113.0/24`
   - **Description**: `Production backend server`

#### Option C: Allow From Anywhere (NOT RECOMMENDED)

**⚠️ SECURITY RISK** - Only use for testing

1. Click **Add rule**
2. Configure:
   - **Type**: `MySQL/Aurora`
   - **Source**: `Anywhere-IPv4` (`0.0.0.0/0`)
   - **Description**: `Temporary - REMOVE AFTER TESTING`

> 🚨 **NEVER** use `0.0.0.0/0` in production! Your database will be exposed to the entire internet. Rely on strong passwords, but don't make it easy for attackers.

#### Option D: Allow From AWS Services (Lambda, EC2)

**Best for**: Backend running on AWS

1. If your backend is on EC2:
   - **Type**: `MySQL/Aurora`
   - **Source**: `Custom`
   - **Source**: Select your EC2 instance's security group
   - **Description**: `Backend EC2 instances`

2. If your backend is on Lambda:
   - Ensure Lambda is in the same VPC as RDS
   - Add Lambda's security group as the source

### Step 5: Save Rules

1. Review all rules
2. Click **Save rules**
3. Rules take effect immediately

---

## Implementation: Connecting to RDS from Your Code

### Python - Using PyMySQL

Install the MySQL connector:

```bash
pip install pymysql
```

**Connection Code**:

```python
import pymysql
import os

# Database connection settings
DB_HOST = 'newgate-app-database.abc123xyz.us-east-1.rds.amazonaws.com'
DB_PORT = 3306
DB_USER = 'admin'
DB_PASSWORD = os.environ.get('DB_PASSWORD')  # Store securely!
DB_NAME = 'newgate_app_db'

def get_db_connection():
    """
    Create and return a database connection
    """
    try:
        connection = pymysql.connect(
            host=DB_HOST,
            port=DB_PORT,
            user=DB_USER,
            password=DB_PASSWORD,
            database=DB_NAME,
            charset='utf8mb4',
            cursorclass=pymysql.cursors.DictCursor,
            connect_timeout=10
        )
        print("✅ Database connection successful!")
        return connection
    except pymysql.Error as e:
        print(f"❌ Error connecting to database: {e}")
        return None

# Example: Query data
def get_users():
    connection = get_db_connection()
    if connection:
        try:
            with connection.cursor() as cursor:
                cursor.execute("SELECT * FROM users")
                users = cursor.fetchall()
                return users
        finally:
            connection.close()
    return []

# Example: Insert data
def create_user(name, email):
    connection = get_db_connection()
    if connection:
        try:
            with connection.cursor() as cursor:
                sql = "INSERT INTO users (name, email) VALUES (%s, %s)"
                cursor.execute(sql, (name, email))
                connection.commit()
                return cursor.lastrowid
        finally:
            connection.close()
    return None
```

### Python - Using SQLAlchemy (Recommended for Large Apps)

Install dependencies:

```bash
pip install sqlalchemy pymysql
```

**Connection Code**:

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

# Database connection string
DB_USER = 'admin'
DB_PASSWORD = os.environ.get('DB_PASSWORD')
DB_HOST = 'newgate-app-database.abc123xyz.us-east-1.rds.amazonaws.com'
DB_PORT = 3306
DB_NAME = 'newgate_app_db'

DATABASE_URL = f"mysql+pymysql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"

# Create engine
engine = create_engine(
    DATABASE_URL,
    pool_pre_ping=True,  # Verify connections before using
    pool_recycle=3600,   # Recycle connections after 1 hour
    echo=True            # Log SQL queries (disable in production)
)

# Create session factory
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Base class for models
Base = declarative_base()

# Example model
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(100), unique=True, nullable=False)

# Create tables
Base.metadata.create_all(bind=engine)

# Usage example
def create_user(name, email):
    db = SessionLocal()
    try:
        user = User(name=name, email=email)
        db.add(user)
        db.commit()
        db.refresh(user)
        return user
    finally:
        db.close()

def get_all_users():
    db = SessionLocal()
    try:
        users = db.query(User).all()
        return users
    finally:
        db.close()
```

### Flask Application Example

```python
from flask import Flask, jsonify, request
import pymysql
import os

app = Flask(__name__)

# Database configuration
DB_CONFIG = {
    'host': 'newgate-app-database.abc123xyz.us-east-1.rds.amazonaws.com',
    'port': 3306,
    'user': 'admin',
    'password': os.environ.get('DB_PASSWORD'),
    'database': 'newgate_app_db'
}

def get_db():
    return pymysql.connect(**DB_CONFIG)

@app.route('/api/users', methods=['GET'])
def get_users():
    try:
        conn = get_db()
        cursor = conn.cursor(pymysql.cursors.DictCursor)
        cursor.execute("SELECT id, name, email FROM users")
        users = cursor.fetchall()
        cursor.close()
        conn.close()
        return jsonify({'success': True, 'users': users})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

@app.route('/api/users', methods=['POST'])
def create_user():
    try:
        data = request.json
        conn = get_db()
        cursor = conn.cursor()
        cursor.execute(
            "INSERT INTO users (name, email) VALUES (%s, %s)",
            (data['name'], data['email'])
        )
        conn.commit()
        user_id = cursor.lastrowid
        cursor.close()
        conn.close()
        return jsonify({'success': True, 'user_id': user_id}), 201
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
```

### Environment Variables Setup

**Never hardcode credentials!** Use environment variables:

**Linux/Mac (.bashrc or .zshrc)**:
```bash
export DB_PASSWORD="YourSecurePassword123!@#"
export DB_HOST="newgate-app-database.abc123xyz.us-east-1.rds.amazonaws.com"
export DB_NAME="newgate_app_db"
export DB_USER="admin"
```

**Windows (Command Prompt)**:
```cmd
set DB_PASSWORD=YourSecurePassword123!@#
set DB_HOST=newgate-app-database.abc123xyz.us-east-1.rds.amazonaws.com
```

**Using .env file** (recommended for development):

Install python-dotenv:
```bash
pip install python-dotenv
```

Create `.env` file:
```env
DB_HOST=newgate-app-database.abc123xyz.us-east-1.rds.amazonaws.com
DB_PORT=3306
DB_USER=admin
DB_PASSWORD=YourSecurePassword123!@#
DB_NAME=newgate_app_db
```

Load in your code:
```python
from dotenv import load_dotenv
import os

load_dotenv()  # Load .env file

DB_HOST = os.getenv('DB_HOST')
DB_PASSWORD = os.getenv('DB_PASSWORD')
```

**🚨 CRITICAL**: Add `.env` to `.gitignore` to never commit credentials!

---

## Creating Tables and Initial Setup

### Connect via MySQL Client

**Option 1: MySQL Workbench** (GUI)
1. Download from mysql.com
2. Create new connection:
   - Hostname: Your RDS endpoint
   - Port: 3306
   - Username: admin
   - Password: Your master password

**Option 2: Command Line**
```bash
mysql -h newgate-app-database.abc123xyz.us-east-1.rds.amazonaws.com -P 3306 -u admin -p
```

### Create Initial Tables

```sql
-- Create users table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Create images table (for AI-generated images)
CREATE TABLE images (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    image_url VARCHAR(500) NOT NULL,
    prompt TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Create indexes for better query performance
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_image_user ON images(user_id);
CREATE INDEX idx_image_created ON images(created_at);

-- Insert test data
INSERT INTO users (name, email) VALUES 
    ('John Doe', 'john@example.com'),
    ('Jane Smith', 'jane@example.com');
```

---

## Security Best Practices

### ✅ DO:

1. **Use strong passwords**:
   - Minimum 12 characters
   - Mix of uppercase, lowercase, numbers, symbols
   - Use a password generator

2. **Store credentials securely**:
   - Use environment variables
   - Use AWS Secrets Manager (production)
   - Never commit to Git

3. **Restrict security group rules**:
   - Only allow specific IPs
   - Remove `0.0.0.0/0` rules
   - Review rules monthly

4. **Enable encryption**:
   - At rest: RDS encryption
   - In transit: SSL/TLS connections

5. **Enable automated backups**:
   - Set retention period
   - Test restore procedures

6. **Use least privilege**:
   - Create separate database users for applications
   - Don't use the master user in production code
   ```sql
   -- Create application-specific user
   CREATE USER 'app_user'@'%' IDENTIFIED BY 'strong_password';
   GRANT SELECT, INSERT, UPDATE, DELETE ON newgate_app_db.* TO 'app_user'@'%';
   FLUSH PRIVILEGES;
   ```

7. **Monitor your database**:
   - Enable CloudWatch alarms
   - Set up alerts for high CPU, low storage
   - Monitor slow queries

8. **Regular maintenance**:
   - Review parameter groups
   - Update to latest minor versions
   - Clean up unused connections

### ❌ DON'T:

1. ❌ Use weak passwords
2. ❌ Make database publicly accessible with `0.0.0.0/0` in production
3. ❌ Hardcode credentials in code
4. ❌ Ignore security patches and updates
5. ❌ Skip backups for production databases
6. ❌ Use the master user for application connections
7. ❌ Store sensitive data unencrypted
8. ❌ Forget to delete test databases

---

## Connecting with SSL/TLS (Recommended)

For production, always use encrypted connections:

```python
import pymysql
import ssl

ssl_context = ssl.create_default_context()
ssl_context.check_hostname = False
ssl_context.verify_mode = ssl.CERT_NONE

connection = pymysql.connect(
    host=DB_HOST,
    port=DB_PORT,
    user=DB_USER,
    password=DB_PASSWORD,
    database=DB_NAME,
    ssl=ssl_context
)
```

For production, download the RDS CA certificate and verify:
```python
import pymysql

connection = pymysql.connect(
    host=DB_HOST,
    port=DB_PORT,
    user=DB_USER,
    password=DB_PASSWORD,
    database=DB_NAME,
    ssl={'ca': '/path/to/rds-ca-2019-root.pem'}
)
```

Download certificate:
```bash
wget https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
```

---

## Troubleshooting

### "Can't connect to MySQL server"

**Problem**: Connection times out or refused

**Solutions**:
1. ✅ Verify database status is "Available" in RDS console
2. ✅ Check security group allows inbound traffic from your IP
3. ✅ Confirm you're using the correct endpoint and port
4. ✅ Verify your IP hasn't changed (update security group)
5. ✅ Check if database is publicly accessible (if connecting from outside VPC)
6. ✅ Ensure your firewall allows outbound connections on port 3306

### "Access denied for user"

**Problem**: Wrong username or password

**Solutions**:
1. ✅ Verify username (check RDS console)
2. ✅ Verify password (no extra spaces or special characters issues)
3. ✅ Check if user has proper permissions
4. ✅ Try connecting with MySQL Workbench to verify credentials

### "Unknown database"

**Problem**: Database name doesn't exist

**Solutions**:
1. ✅ Verify you created an initial database during setup
2. ✅ Connect without specifying database name, then run `CREATE DATABASE`
3. ✅ Check available databases: `SHOW DATABASES;`

### Connection works but queries are slow

**Problem**: Performance issues

**Solutions**:
1. ✅ Check CloudWatch metrics for CPU and memory usage
2. ✅ Enable slow query log to identify problem queries
3. ✅ Add indexes to frequently queried columns
4. ✅ Consider upgrading instance class
5. ✅ Use connection pooling

### "Too many connections"

**Problem**: Max connections reached

**Solutions**:
1. ✅ Close connections properly in your code
2. ✅ Use connection pooling
3. ✅ Increase max_connections parameter (requires reboot)
4. ✅ Check for connection leaks in your application

---

## Monitoring and Maintenance

### CloudWatch Metrics to Monitor

1. **CPUUtilization**: Should be < 80% normally
2. **DatabaseConnections**: Watch for spikes
3. **FreeStorageSpace**: Set alarm before running out
4. **ReadLatency / WriteLatency**: Monitor query performance
5. **FreeableMemory**: Low memory
