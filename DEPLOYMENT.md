# JALDISEVA Deployment Guide

This guide provides comprehensive instructions for deploying the JALDISEVA telemedicine platform to production environments.

## 📋 Pre-Deployment Checklist

### System Requirements
- [ ] Ubuntu 20.04+ or CentOS 8+ server
- [ ] 4GB+ RAM (8GB recommended)
- [ ] 50GB+ storage space
- [ ] Node.js 18+ support
- [ ] PostgreSQL 15+ compatible
- [ ] SSL certificate for HTTPS
- [ ] Domain name configured

### Required Services
- [ ] Google Gemini API key
- [ ] SMTP service for email notifications
- [ ] Backup storage solution
- [ ] Monitoring service (optional)

## 🚀 Production Deployment

### Step 1: Server Setup

#### 1.1 Initial Server Configuration
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y curl wget git unzip nginx certbot python3-certbot-nginx

# Create application user
sudo adduser --system --group --home /opt/jaldiseva jaldiseva
sudo usermod -aG sudo jaldiseva
```

#### 1.2 Install Node.js 18+
```bash
# Install Node.js via NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version  # Should be 18.x or higher
npm --version
```

#### 1.3 Install PostgreSQL
```bash
# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Start and enable PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE jaldiseva;
CREATE USER jaldiseva WITH ENCRYPTED PASSWORD 'your_secure_password_here';
GRANT ALL PRIVILEGES ON DATABASE jaldiseva TO jaldiseva;
ALTER USER jaldiseva CREATEDB;
\q
EOF
```

#### 1.4 Install Process Manager
```bash
# Install PM2 globally
sudo npm install -g pm2

# Setup PM2 startup script
pm2 startup
# Follow the instructions provided by PM2
```

### Step 2: Application Deployment

#### 2.1 Clone and Setup Repository
```bash
# Switch to application user
sudo su - jaldiseva

# Clone repository
cd /opt/jaldiseva
git clone https://github.com/your-username/jaldiseva.git app
cd app

# Set proper permissions
sudo chown -R jaldiseva:jaldiseva /opt/jaldiseva
```

#### 2.2 Backend Deployment
```bash
cd /opt/jaldiseva/app/backend

# Install dependencies
npm ci --production

# Create production environment file
cat > .env << EOF
# Production Environment Configuration
NODE_ENV=production

# Database Configuration
DATABASE_URL=postgresql://jaldiseva:your_secure_password_here@localhost:5432/jaldiseva
DB_HOST=localhost
DB_PORT=5432
DB_NAME=jaldiseva
DB_USER=jaldiseva
DB_PASSWORD=your_secure_password_here

# JWT Configuration
JWT_SECRET=$(openssl rand -base64 64)
JWT_EXPIRES_IN=7d

# Google Gemini AI Configuration
GEMINI_API_KEY=AIzaSyCs-pxODTsSkxMWTW2GCtbzD2ichb4BFnU

# Server Configuration
PORT=3001
FRONTEND_URL=https://your-domain.com

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# File Upload Configuration
MAX_FILE_SIZE=10485760
UPLOAD_PATH=./uploads

# Email Configuration (Optional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
EOF

# Secure the environment file
chmod 600 .env

# Setup database schema
npm run db:setup

# Create uploads directory
mkdir -p uploads
chmod 755 uploads
```

#### 2.3 Frontend Deployment
```bash
cd /opt/jaldiseva/app/frontend

# Install dependencies
npm ci

# Create production environment file
cat > .env.local << EOF
NEXT_PUBLIC_API_URL=https://your-domain.com/api
NEXT_PUBLIC_SOCKET_URL=https://your-domain.com
NEXT_PUBLIC_APP_NAME=JALDISEVA
EOF

# Build for production
npm run build

# Verify build
ls -la .next/
```

### Step 3: Process Management with PM2

#### 3.1 Create PM2 Ecosystem File
```bash
cd /opt/jaldiseva/app

cat > ecosystem.config.js << EOF
module.exports = {
  apps: [
    {
      name: 'jaldiseva-backend',
      cwd: './backend',
      script: 'server.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3001
      },
      error_file: '/var/log/jaldiseva/backend-error.log',
      out_file: '/var/log/jaldiseva/backend-out.log',
      log_file: '/var/log/jaldiseva/backend-combined.log',
      time: true,
      max_memory_restart: '1G',
      node_args: '--max-old-space-size=1024'
    },
    {
      name: 'jaldiseva-frontend',
      cwd: './frontend',
      script: 'npm',
      args: 'start',
      instances: 1,
      exec_mode: 'fork',
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      },
      error_file: '/var/log/jaldiseva/frontend-error.log',
      out_file: '/var/log/jaldiseva/frontend-out.log',
      log_file: '/var/log/jaldiseva/frontend-combined.log',
      time: true,
      max_memory_restart: '512M'
    }
  ]
};
EOF
```

#### 3.2 Setup Logging Directory
```bash
# Create log directory
sudo mkdir -p /var/log/jaldiseva
sudo chown jaldiseva:jaldiseva /var/log/jaldiseva

# Setup log rotation
sudo tee /etc/logrotate.d/jaldiseva << EOF
/var/log/jaldiseva/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 644 jaldiseva jaldiseva
    postrotate
        pm2 reloadLogs
    endscript
}
EOF
```

#### 3.3 Start Applications
```bash
# Start applications with PM2
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Setup PM2 to start on boot
pm2 startup
# Follow the instructions provided
```

### Step 4: Nginx Configuration

#### 4.1 Create Nginx Configuration
```bash
sudo tee /etc/nginx/sites-available/jaldiseva << 'EOF'
# Rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

# Upstream servers
upstream backend {
    server 127.0.0.1:3001;
    keepalive 32;
}

upstream frontend {
    server 127.0.0.1:3000;
    keepalive 32;
}

server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    # SSL Configuration (will be updated by Certbot)
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # SSL Security Headers
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied expired no-cache no-store private must-revalidate auth;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

    # File Upload Size
    client_max_body_size 10M;

    # Backend API
    location /api {
        limit_req zone=api burst=20 nodelay;
        
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Authentication endpoints (stricter rate limiting)
    location /api/auth {
        limit_req zone=login burst=5 nodelay;
        
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket for real-time features
    location /socket.io/ {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket specific timeouts
        proxy_read_timeout 86400;
    }

    # Static files from backend (uploads, etc.)
    location /uploads {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        
        # Cache static files
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Frontend application
    location / {
        proxy_pass http://frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Next.js static files
    location /_next/static {
        proxy_pass http://frontend;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Favicon and other static assets
    location ~* \.(ico|css|js|gif|jpe?g|png|svg|woff|woff2|ttf|eot)$ {
        proxy_pass http://frontend;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
EOF
```

#### 4.2 Enable Site and Test Configuration
```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/jaldiseva /etc/nginx/sites-enabled/

# Remove default site
sudo rm -f /etc/nginx/sites-enabled/default

# Test Nginx configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
sudo systemctl enable nginx
```

### Step 5: SSL Certificate Setup

#### 5.1 Obtain SSL Certificate with Let's Encrypt
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate (replace with your domain)
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Test automatic renewal
sudo certbot renew --dry-run

# Setup automatic renewal cron job
echo "0 12 * * * /usr/bin/certbot renew --quiet" | sudo crontab -
```

### Step 6: Firewall Configuration

#### 6.1 Setup UFW Firewall
```bash
# Enable UFW
sudo ufw --force enable

# Allow SSH (adjust port if needed)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow PostgreSQL (only from localhost)
sudo ufw allow from 127.0.0.1 to any port 5432

# Check status
sudo ufw status verbose
```

### Step 7: Database Optimization

#### 7.1 PostgreSQL Performance Tuning
```bash
# Edit PostgreSQL configuration
sudo nano /etc/postgresql/15/main/postgresql.conf

# Add/modify these settings based on your server specs:
# shared_buffers = 256MB                    # 25% of RAM
# effective_cache_size = 1GB                # 75% of RAM
# work_mem = 4MB                            # RAM/max_connections/4
# maintenance_work_mem = 64MB               # RAM/16
# checkpoint_completion_target = 0.9
# wal_buffers = 16MB
# default_statistics_target = 100
# random_page_cost = 1.1
# effective_io_concurrency = 200

# Restart PostgreSQL
sudo systemctl restart postgresql
```

#### 7.2 Create Database Indexes
```bash
# Connect to database and create performance indexes
sudo -u postgres psql jaldiseva << EOF
-- Performance indexes
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_patients_phone ON patients(phone);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_patients_email ON patients(email);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_doctors_specialization ON doctors(specialization);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_doctors_city_state ON doctors(city, state);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_queue_doctor_status ON queue_entries(doctor_id, status);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_queue_patient_status ON queue_entries(patient_id, status);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_prescriptions_patient ON prescriptions(patient_id);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_prescriptions_doctor ON prescriptions(doctor_id);
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_prescriptions_created_at ON prescriptions(created_at);

-- Analyze tables for query optimization
ANALYZE;
\q
EOF
```

### Step 8: Monitoring and Logging

#### 8.1 Setup System Monitoring
```bash
# Install monitoring tools
sudo apt install -y htop iotop nethogs

# Create monitoring script
sudo tee /opt/jaldiseva/monitor.sh << 'EOF'
#!/bin/bash
# JALDISEVA System Monitor

LOG_FILE="/var/log/jaldiseva/system-monitor.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Check disk space
DISK_USAGE=$(df -h / | awk 'NR==2{print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo "[$DATE] WARNING: Disk usage is ${DISK_USAGE}%" >> $LOG_FILE
fi

# Check memory usage
MEM_USAGE=$(free | grep Mem | awk '{printf("%.0f", $3/$2 * 100.0)}')
if [ $MEM_USAGE -gt 85 ]; then
    echo "[$DATE] WARNING: Memory usage is ${MEM_USAGE}%" >> $LOG_FILE
fi

# Check PM2 processes
PM2_STATUS=$(pm2 jlist | jq -r '.[] | select(.pm2_env.status != "online") | .name')
if [ ! -z "$PM2_STATUS" ]; then
    echo "[$DATE] ERROR: PM2 process down: $PM2_STATUS" >> $LOG_FILE
fi

# Check database connection
sudo -u jaldiseva psql -d jaldiseva -c "SELECT 1;" > /dev/null 2>&1
if [ $? -ne 0 ]; then
    echo "[$DATE] ERROR: Database connection failed" >> $LOG_FILE
fi
EOF

chmod +x /opt/jaldiseva/monitor.sh

# Setup monitoring cron job
echo "*/5 * * * * /opt/jaldiseva/monitor.sh" | sudo crontab -u jaldiseva -
```

#### 8.2 Setup Application Health Checks
```bash
# Create health check script
sudo tee /opt/jaldiseva/health-check.sh << 'EOF'
#!/bin/bash
# JALDISEVA Health Check

API_URL="https://your-domain.com/api/health"
LOG_FILE="/var/log/jaldiseva/health-check.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Check API health
HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" $API_URL)
if [ $HTTP_STATUS -ne 200 ]; then
    echo "[$DATE] ERROR: API health check failed (HTTP $HTTP_STATUS)" >> $LOG_FILE
    # Restart backend if health check fails
    pm2 restart jaldiseva-backend
fi

# Check frontend
FRONTEND_STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://your-domain.com)
if [ $FRONTEND_STATUS -ne 200 ]; then
    echo "[$DATE] ERROR: Frontend health check failed (HTTP $FRONTEND_STATUS)" >> $LOG_FILE
    # Restart frontend if health check fails
    pm2 restart jaldiseva-frontend
fi
EOF

chmod +x /opt/jaldiseva/health-check.sh

# Setup health check cron job (every 2 minutes)
echo "*/2 * * * * /opt/jaldiseva/health-check.sh" | sudo crontab -u jaldiseva -
```

### Step 9: Backup Strategy

#### 9.1 Database Backup Script
```bash
# Create backup directory
sudo mkdir -p /opt/jaldiseva/backups
sudo chown jaldiseva:jaldiseva /opt/jaldiseva/backups

# Create database backup script
sudo tee /opt/jaldiseva/backup-db.sh << 'EOF'
#!/bin/bash
# JALDISEVA Database Backup

BACKUP_DIR="/opt/jaldiseva/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="jaldiseva_backup_$DATE.sql"

# Create backup
sudo -u postgres pg_dump jaldiseva > "$BACKUP_DIR/$BACKUP_FILE"

# Compress backup
gzip "$BACKUP_DIR/$BACKUP_FILE"

# Keep only last 7 days of backups
find $BACKUP_DIR -name "jaldiseva_backup_*.sql.gz" -mtime +7 -delete

echo "Database backup completed: $BACKUP_FILE.gz"
EOF

chmod +x /opt/jaldiseva/backup-db.sh

# Setup daily backup cron job
echo "0 2 * * * /opt/jaldiseva/backup-db.sh" | sudo crontab -u jaldiseva -
```

#### 9.2 Application Files Backup
```bash
# Create application backup script
sudo tee /opt/jaldiseva/backup-files.sh << 'EOF'
#!/bin/bash
# JALDISEVA Files Backup

BACKUP_DIR="/opt/jaldiseva/backups"
DATE=$(date +%Y%m%d_%H%M%S)
APP_DIR="/opt/jaldiseva/app"

# Backup uploads and configuration
tar -czf "$BACKUP_DIR/files_backup_$DATE.tar.gz" \
    -C "$APP_DIR" \
    backend/uploads \
    backend/.env \
    frontend/.env.local

# Keep only last 7 days of file backups
find $BACKUP_DIR -name "files_backup_*.tar.gz" -mtime +7 -delete

echo "Files backup completed: files_backup_$DATE.tar.gz"
EOF

chmod +x /opt/jaldiseva/backup-files.sh

# Setup weekly file backup
echo "0 3 * * 0 /opt/jaldiseva/backup-files.sh" | sudo crontab -u jaldiseva -
```

### Step 10: Final Verification

#### 10.1 Service Status Check
```bash
# Check all services
sudo systemctl status nginx
sudo systemctl status postgresql
pm2 status

# Check application logs
pm2 logs --lines 50

# Test database connection
sudo -u jaldiseva psql -d jaldiseva -c "SELECT COUNT(*) FROM patients;"
```

#### 10.2 Application Testing
```bash
# Test API endpoints
curl -k https://your-domain.com/api/health
curl -k https://your-domain.com/api/doctors

# Test frontend
curl -k https://your-domain.com

# Check SSL certificate
openssl s_client -connect your-domain.com:443 -servername your-domain.com
```

## 🔧 Post-Deployment Configuration

### Environment-Specific Settings

#### Production Optimizations
1. **Enable database connection pooling**
2. **Configure Redis for session storage** (optional)
3. **Setup CDN for static assets** (optional)
4. **Configure email notifications**
5. **Setup error tracking** (Sentry, etc.)

#### Security Hardening
1. **Change default PostgreSQL port**
2. **Setup fail2ban for intrusion prevention**
3. **Configure automated security updates**
4. **Setup VPN access for admin functions**
5. **Regular security audits**

### Maintenance Tasks

#### Daily
- [ ] Check application logs
- [ ] Verify backup completion
- [ ] Monitor system resources

#### Weekly
- [ ] Review security logs
- [ ] Update system packages
- [ ] Check SSL certificate expiry
- [ ] Performance monitoring review

#### Monthly
- [ ] Database maintenance (VACUUM, ANALYZE)
- [ ] Log rotation and cleanup
- [ ] Security patches review
- [ ] Backup restoration testing

## 🚨 Troubleshooting

### Common Issues

#### Application Won't Start
```bash
# Check PM2 logs
pm2 logs jaldiseva-backend --lines 100
pm2 logs jaldiseva-frontend --lines 100

# Check environment variables
pm2 env 0  # Replace 0 with process ID

# Restart applications
pm2 restart all
```

#### Database Connection Issues
```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Check database connectivity
sudo -u postgres psql -c "\l"

# Check database logs
sudo tail -f /var/log/postgresql/postgresql-15-main.log
```

#### SSL Certificate Issues
```bash
# Check certificate status
sudo certbot certificates

# Renew certificate manually
sudo certbot renew --force-renewal

# Check Nginx configuration
sudo nginx -t
```

#### Performance Issues
```bash
# Check system resources
htop
iotop
nethogs

# Check database performance
sudo -u postgres psql jaldiseva -c "SELECT * FROM pg_stat_activity;"

# Analyze slow queries
sudo -u postgres psql jaldiseva -c "SELECT query, mean_time, calls FROM pg_stat_statements ORDER BY mean_time DESC LIMIT 10;"
```

### Emergency Procedures

#### Application Recovery
1. Stop all services: `pm2 stop all`
2. Check logs for errors
3. Restore from backup if needed
4. Restart services: `pm2 start all`

#### Database Recovery
1. Stop applications
2. Restore database from backup
3. Verify data integrity
4. Restart applications

## 📞 Support Contacts

- **Technical Issues**: Create GitHub issue
- **Security Concerns**: security@jaldiseva.com
- **Emergency Contact**: +91-XXXX-XXXX-XX

---

**Deployment completed successfully! 🎉**

Your JALDISEVA telemedicine platform is now live and ready to serve rural healthcare needs.
