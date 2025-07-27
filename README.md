# N8N Docker Compose Setup with Traefik (SSL) + PostgreSQL

🚀 **Production-ready N8N setup** dengan PostgreSQL database, Traefik reverse proxy, dan SSL certificates otomatis dari Let's Encrypt.

## 📋 Prerequisites

- Docker & Docker Compose terinstall
- Domain yang sudah pointing ke server Anda
- Port 80 dan 443 terbuka di firewall
- Email address untuk Let's Encrypt notifications

## 🏗️ Architecture

```
Internet → Traefik (Port 80/443) → N8N (Port 5678)
                ↓
            PostgreSQL Database
```

**Components:**
- **Traefik v3.0** - Reverse proxy dengan SSL termination
- **N8N Latest** - Workflow automation platform
- **PostgreSQL 15** - Database untuk N8N
- **Let's Encrypt** - SSL certificates otomatis

## 🚀 Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/alfalaah404/N8N-Docker-Compose-Setup-with-Traefik-SSL-PostgreSQL.git
cd N8N-Docker-Compose-Setup-with-Traefik-SSL-PostgreSQL
```

### 2. Configure Environment

Copy dan edit file environment:

```bash
cp .env.example .env
nano .env
```

Edit konfigurasi sesuai kebutuhan:

```env
# Domain configuration
DOMAIN=your.domain.com

# Let's Encrypt email for SSL certificates
ACME_EMAIL=your.email@domain.com

# PostgreSQL Database configuration
POSTGRES_DB=n8n
POSTGRES_USER=n8n_user
POSTGRES_PASSWORD=your_secure_password_here

# N8N Basic Auth (optional, hapus setelah setup initial)
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your_admin_password_here
```

### 3. Deploy

```bash
# Start semua services
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f
```

### 4. Access N8N

Tunggu beberapa menit untuk SSL certificate generation, kemudian akses:

- **N8N Interface**: `https://your.domain.com`

Login menggunakan credentials dari `N8N_BASIC_AUTH_USER` dan `N8N_BASIC_AUTH_PASSWORD`.

## 🔧 Configuration Details

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `DOMAIN` | Your domain name | `n8n.example.com` |
| `ACME_EMAIL` | Email for Let's Encrypt | `admin@example.com` |
| `POSTGRES_DB` | Database name | `n8n` |
| `POSTGRES_USER` | Database username | `n8n_user` |
| `POSTGRES_PASSWORD` | Database password | `secure_password` |
| `N8N_BASIC_AUTH_USER` | N8N login username | `admin` |
| `N8N_BASIC_AUTH_PASSWORD` | N8N login password | `admin_password` |

### Security Features

✅ **HTTPS Only** - Automatic HTTP to HTTPS redirect  
✅ **SSL Certificates** - Auto-renewal via Let's Encrypt  
✅ **Secure Cookies** - N8N cookies dengan secure flag  
✅ **Network Isolation** - Separated networks untuk security  
✅ **Basic Authentication** - Optional protection untuk N8N

### Data Persistence

Data disimpan dalam Docker volumes yang persistent:

- **`postgres_data`** - Database PostgreSQL
- **`n8n_data`** - N8N workflows, credentials, dan settings
- **`traefik_letsencrypt`** - SSL certificates Let's Encrypt

## 🛠️ Management Commands

### Start/Stop Services

```bash
# Start semua services
docker compose up -d

# Stop semua services (data tetap aman)
docker compose down

# Restart services
docker compose restart

# Stop satu service
docker compose stop n8n
```

### View Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f n8n
docker compose logs -f traefik
docker compose logs -f postgres
```

### Database Operations

```bash
# Access PostgreSQL
docker compose exec postgres psql -U n8n_user -d n8n

# Create database backup
docker compose exec postgres pg_dump -U n8n_user n8n > backup_$(date +%Y%m%d).sql

# Restore backup
docker compose exec -T postgres psql -U n8n_user -d n8n < backup.sql
```

## 🔒 Security Recommendations

### Production Hardening

1. **Disable N8N Basic Auth** (setelah setup user accounts)
   ```env
   # Comment out di .env
   # N8N_BASIC_AUTH_ACTIVE=true
   # N8N_BASIC_AUTH_USER=admin
   # N8N_BASIC_AUTH_PASSWORD=password
   ```

2. **Restrict PostgreSQL Access**
   ```yaml
   # Hapus port exposure di postgres service
   # ports:
   #   - "127.0.0.1:5432:5432"
   ```

3. **Strong Passwords**
   - Gunakan password generator untuk semua credentials
   - Minimal 16 karakter dengan kombinasi huruf, angka, dan simbol

### Firewall Configuration

```bash
# UFW Example
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable
```

## 🔄 Backup & Restore

### Automated Backup Script

Buat script backup otomatis:

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup database
docker compose exec -T postgres pg_dump -U n8n_user n8n > $BACKUP_DIR/n8n_db_$DATE.sql

# Backup N8N data volume
docker run --rm -v n8n-docker-setup_n8n_data:/data -v $BACKUP_DIR:/backup alpine tar czf /backup/n8n_data_$DATE.tar.gz /data

# Backup SSL certificates
docker run --rm -v n8n-docker-setup_traefik_letsencrypt:/data -v $BACKUP_DIR:/backup alpine tar czf /backup/letsencrypt_$DATE.tar.gz /data

# Cleanup old backups (keep last 7 days)
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $DATE"
```

### Restore Process

```bash
# Stop services
docker compose down

# Restore database
docker compose up postgres -d
sleep 10
docker compose exec -T postgres psql -U n8n_user -d n8n < backup.sql

# Restore volumes (if needed)
docker run --rm -v n8n-docker-setup_n8n_data:/data -v $(pwd):/backup alpine tar xzf /backup/n8n_data_backup.tar.gz -C /

# Start all services
docker compose up -d
```

## 🐛 Troubleshooting

### Common Issues

**SSL Certificate Issues**
```bash
# Check Traefik logs
docker compose logs traefik | grep -i error

# Verify domain DNS
nslookup your.domain.com

# Test Let's Encrypt staging
# Tambahkan ke traefik command: --certificatesresolvers.letsencrypt.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory
```

**Database Connection Issues**
```bash
# Check PostgreSQL logs
docker compose logs postgres

# Test database connection
docker compose exec postgres psql -U n8n_user -d n8n -c "SELECT version();"
```

**N8N Not Accessible**
```bash
# Check N8N logs
docker compose logs n8n | tail -50

# Verify network connectivity
docker compose exec n8n wget -O- http://localhost:5678/healthz
```

### Reset Everything

```bash
# ⚠️ PERINGATAN: Ini akan menghapus semua data!
docker compose down -v
docker system prune -a
```

## 📚 Additional Resources

- [N8N Documentation](https://docs.n8n.io/)
- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ⭐ Support

Jika setup ini membantu Anda, berikan ⭐ pada repository ini!

Untuk pertanyaan atau bantuan, silakan buka [GitHub Issues](issues).

---

**Dibuat dengan ❤️ untuk komunitas N8N Indonesia**
