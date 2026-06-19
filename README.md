# r_jobs: Database-Driven R Script Job Queue

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Status: Beta](https://img.shields.io/badge/Status-Beta-yellow.svg)

A powerful job queue system for managing and executing R scripts at scale. Submit jobs via a PHP web interface, store them in a database, and let a 24/7 Mac worker automatically process them.

## 🎯 Features

- **📝 Web Dashboard** - Submit and monitor jobs from any browser
- **🔄 24/7 Worker Service** - Automatic job execution on Mac
- **💾 Persistent Storage** - MySQL/PostgreSQL backend
- **📊 Real-time Monitoring** - Track job status, logs, and results
- **⚡ Priority Queue** - Jobs processed by priority and creation time
- **🔐 Secure** - Authentication, parameter validation, script isolation
- **📱 RESTful API** - Integrate from any application
- **🐛 Comprehensive Logging** - Full execution logs stored in database
- **⏱️ Timeout Support** - Prevent hanging jobs
- **🔄 Retry Failed Jobs** - Automatic or manual retry capability

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│  PHP Web Dashboard                              │
│  - Submit jobs                                  │
│  - Monitor status                               │
└─────────────────────────────┬───────────────────┘
                              │ INSERT/UPDATE
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Shared Database (MySQL/PostgreSQL)                                 │
│  - Job queue                                                        │
│  - Results & logs                                                   │
│  - Worker status                                                    │
└─────────────────────────────┬───────────────────────────────────────┘
                              │ POLL/SELECT
                              │
                    ┌─────────┴──────────┐
                    │                    │
                    │   Mac 24/7         │
                    │   ┌─────────────┐  │
                    │   │ R Worker    │  │
                    │   │ Service     │  │
                    │   └─────────────┘  │
                    │  (LaunchAgent)     │
                    └────────────────────┘
```

## 📦 Quick Start

### Prerequisites

- **Database**: MySQL 5.7+ or PostgreSQL 10+
- **Mac**: macOS 10.12+
- **R**: R 4.0+ with packages: DBI, RMySQL/RPostgreSQL, glue, callr, jsonlite
- **PHP**: 7.4+
- **Web Server**: Apache/Nginx for PHP

### 1. Database Setup

```bash
# Import the database schema
mysql -h your-db-host -u root -p < schema/init.sql

# Or for PostgreSQL
psql -h your-db-host -U postgres -d r_jobs -f schema/init.sql
```

### 2. PHP Setup

```bash
# Copy configuration
cp php/config.example.php php/config.php

# Edit with your database credentials
nano php/config.php

# Setup web server (Apache example)
cp -r php/* /var/www/r_jobs/
```

Access dashboard at: `http://your-domain/dashboard.php`

### 3. Mac Worker Setup

```bash
# Install R dependencies
Rscript -e 'install.packages(c("DBI", "RMySQL", "glue", "callr", "jsonlite"))'

# Setup LaunchAgent
cp macos/com.r_jobs.worker.plist ~/Library/LaunchAgents/
nano ~/Library/LaunchAgents/com.r_jobs.worker.plist
# Edit environment variables with your database credentials

# Load the service
launchctl load ~/Library/LaunchAgents/com.r_jobs.worker.plist

# Verify it's running
launchctl list | grep r_jobs
tail -f /tmp/r_jobs_worker.log
```

## 📚 Documentation

- [**DATABASE_SETUP.md**](docs/DATABASE_SETUP.md) - Database schema and configuration
- [**PHP_SETUP.md**](docs/PHP_SETUP.md) - PHP backend and web dashboard
- [**MAC_WORKER_SETUP.md**](docs/MAC_WORKER_SETUP.md) - Mac worker 24/7 service setup
- [**API_REFERENCE.md**](docs/API_REFERENCE.md) - Complete API documentation
- [**SECURITY.md**](docs/SECURITY.md) - Security best practices
- [**TROUBLESHOOTING.md**](docs/TROUBLESHOOTING.md) - Common issues and solutions
- [**EXAMPLES.md**](docs/EXAMPLES.md) - Code examples (R, Python, JavaScript, Bash)

## 🚀 Usage Examples

### Submit a Job via PHP Form

```php
<?php
$job_name = "Daily Analysis";
$script_content = "print('Hello from R!')";
$parameters = json_encode(['input_file' => 'data.csv']);
$priority = 5;

// Use the PHP API
include 'php/api/submit_job.php';
?>
```

### Check Job Status

```bash
curl http://your-domain/php/api/get_job_status.php?job_id=1
```

### Monitor in Real-time

Open the web dashboard in your browser:
```
http://your-domain/dashboard.php
```

You'll see:
- ✅ Live job queue
- 📊 Worker status and heartbeat
- 📈 Queue statistics
- 📋 Job logs and output

## 📋 Job States

```
pending ──→ running ──→ completed
              ↓
            failed ──→ (can retry)
              ↓
           cancelled
```

## 🔧 Configuration

### PHP Configuration

**File: `php/config.php`**

```php
define('DB_HOST', 'your-database-host.com');
define('DB_USER', 'your_db_user');
define('DB_PASS', 'your_db_password');
define('DB_NAME', 'r_jobs_db');
define('API_KEY', 'your-secret-api-key'); // For authentication
```

### R Worker Configuration

**File: `macos/com.r_jobs.worker.plist`**

```xml
<dict>
    <key>EnvironmentVariables</key>
    <dict>
        <key>R_JOBS_DB_HOST</key>
        <string>your-database-host.com</string>
        <key>R_JOBS_DB_USER</key>
        <string>db_user</string>
        <key>R_JOBS_DB_PASS</key>
        <string>db_password</string>
        <key>WORKER_ID</key>
        <string>mac-worker-01</string>
    </dict>
</dict>
```

## 📊 Monitoring

### View Worker Logs

```bash
# Real-time logs
tail -f /tmp/r_jobs_worker.log

# Or from system
log stream --predicate 'process == "Rscript"' --level debug
```

### Check Worker Status

```bash
# Is it running?
launchctl list | grep r_jobs

# Get PID
lsof -i :8080 | grep Rscript
```

### Database Queries

```sql
-- Pending jobs
SELECT * FROM jobs WHERE status = 'pending' ORDER BY priority DESC;

-- Worker status
SELECT * FROM workers;

-- Job history (today)
SELECT * FROM jobs WHERE DATE(created_at) = DATE(NOW());

-- Failed jobs
SELECT * FROM jobs WHERE status = 'failed';
```

## 🔐 Security

- ✅ API key authentication
- ✅ Parameter validation and sanitization
- ✅ Script execution in isolated processes
- ✅ Database connection pooling
- ✅ HTTPS support (for production)
- ✅ Rate limiting (configurable)
- ✅ Access control per user/role

See [SECURITY.md](docs/SECURITY.md) for detailed guidelines.

## 🐛 Troubleshooting

### Worker not running?

```bash
# Check if service is loaded
launchctl list | grep r_jobs

# Reload service
launchctl unload ~/Library/LaunchAgents/com.r_jobs.worker.plist
launchctl load ~/Library/LaunchAgents/com.r_jobs.worker.plist

# Check logs
tail -f /tmp/r_jobs_worker.log
```

### Database connection failed?

```bash
# Test connection
mysql -h your-db-host -u user -p -e "SELECT 1"

# Check credentials in config files
grep -r "DB_HOST" php/config.php
grep -r "R_JOBS_DB_HOST" ~/Library/LaunchAgents/com.r_jobs.worker.plist
```

### Jobs stuck in running state?

```sql
-- Check for zombie jobs
SELECT * FROM jobs 
WHERE status = 'running' 
AND updated_at < DATE_SUB(NOW(), INTERVAL 1 HOUR);

-- Manual reset (careful!)
UPDATE jobs SET status = 'failed', error_message = 'Zombie job reset' 
WHERE status = 'running' AND updated_at < DATE_SUB(NOW(), INTERVAL 1 HOUR);
```

See [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for more.

## 📝 License

MIT License - see [LICENSE](LICENSE) file

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Commit changes (`git commit -am 'Add my feature'`)
4. Push to branch (`git push origin feature/my-feature`)
5. Create a Pull Request

## 📞 Support

For issues, questions, or feature requests, please:

- 📖 Check [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)
- 🐛 [Open an issue](https://github.com/LukasKraiger/r_jobs/issues)
- 💬 Start a discussion

## 🗺️ Roadmap

- [ ] Web dashboard UI improvements
- [ ] Email notifications on job completion
- [ ] Multiple worker support (distributed execution)
- [ ] Job scheduling (cron-like)
- [ ] PostgreSQL full support
- [ ] Docker containerization
- [ ] Kubernetes support
- [ ] Advanced analytics and reporting

---

**Happy job queuing! 🎉**
