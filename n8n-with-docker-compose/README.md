# n8n with Docker Compose

Before you start this tutorial, make sure you have create a ct on your Proxmox server, and installed docker, docker-compose, and cron.

## Installation

### 1. Create docker-compose.yml File

Copy this.

```yaml
version: "3"

services:
  traefik:
    image: "traefik"
    restart: always
    command:
      - "--api=true"
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entrypoint.scheme=https"
      - "--entrypoints.websecure.address=:443"
      # - "--certificatesresolvers.mytlschallenge.acme.tlschallenge=true"
      - "--certificatesresolvers.mytlschallenge.acme.dnschallenge.provider=cloudflare" # For Cloudflare
      - "--certificatesresolvers.mytlschallenge.acme.email=${SSL_EMAIL}"
      - "--certificatesresolvers.mytlschallenge.acme.storage=/letsencrypt/acme.json"
    environment:
      - CF_API_EMAIL=${SSL_EMAIL} # For Cloudflare
      - CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN} # For Cloudflare
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ${DATA_FOLDER}/letsencrypt:/letsencrypt
      - /var/run/docker.sock:/var/run/docker.sock:ro

  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    labels:
      - traefik.enable=true
      - traefik.http.routers.n8n.rule=Host(`${SUBDOMAIN}.${DOMAIN_NAME}`)
      - traefik.http.routers.n8n.tls=true
      - traefik.http.routers.n8n.entrypoints=web,websecure
      - traefik.http.routers.n8n.tls.certresolver=mytlschallenge
      - traefik.http.middlewares.n8n.headers.SSLRedirect=true
      - traefik.http.middlewares.n8n.headers.STSSeconds=315360000
      - traefik.http.middlewares.n8n.headers.browserXSSFilter=true
      - traefik.http.middlewares.n8n.headers.contentTypeNosniff=true
      - traefik.http.middlewares.n8n.headers.forceSTSHeader=true
      - traefik.http.middlewares.n8n.headers.SSLHost=${DOMAIN_NAME}
      - traefik.http.middlewares.n8n.headers.STSIncludeSubdomains=true
      - traefik.http.middlewares.n8n.headers.STSPreload=true
      - traefik.http.routers.n8n.middlewares=n8n@docker
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER
      - N8N_BASIC_AUTH_PASSWORD
      - N8N_HOST=${SUBDOMAIN}.${DOMAIN_NAME}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://${SUBDOMAIN}.${DOMAIN_NAME}/
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
    volumes:
      - ${DATA_FOLDER}/.n8n:/home/node/.n8n
      - ${DATA_FOLDER}/backup:/home/node/backup # Backup your workflow and credentials
    user: "0:0"
```

### 2. Create .env File

```env
DATA_FOLDER=/path/to/n8n/
DOMAIN_NAME=example.com
SUBDOMAIN=n8n
N8N_BASIC_AUTH_USER=mytest
N8N_BASIC_AUTH_PASSWORD=password_change_me
GENERIC_TIMEZONE=Asia/Taipei
SSL_EMAIL=no-reply@example.com
CF_DNS_API_TOKEN=your_cf_token
```

### 4. Setup Your Cloudflare

### 3. Run docker-compose

```bash
docker-compose up -d
```

Now you can go to n8n.example.com to use your n8n, enjoy!

## Backup and Import Workflow and Credentials

If you re-create your n8n container, the data inside the container will gone, and you can use cli tool to backup and import it.

### 1. Create Backup Folder

### 2. Create Shell for Backup

```bash
#!/bin/bash

# back dir
BACKUP_DIR="/home/node/backup"
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M")

# check folder exists
mkdir -p "$BACKUP_DIR/workflow"
mkdir -p "$BACKUP_DIR/credentials"

# run backup cmd with docker-compose
docker-compose exec n8n n8n export:workflow --backup --output="$BACKUP_DIR/workflow"
docker-compose exec n8n n8n export:credentials --backup --output="$BACKUP_DIR/credentials"

# delete the backup over 7 days
find "$BACKUP_DIR" -type f -mtime +7 -name "*.json" -delete

echo "Backup completed at $TIMESTAMP and saved in $BACKUP_DIR"
```

Make sure this shell is executable.
```bash
chmod +x ./backup_script.sh
```

### 3. Create Shell for Import

It will be used on container re-created

```bash
#!/bin/bash

BACKUP_DIR="/home/node/backup"

docker-compose exec n8n n8n import:workflow --separate --input="$BACKUP_DIR/workflow"
docker-compose exec n8n n8n import:credentials --separate --input="$BACKUP_DIR/credentials"

echo "Import completed"
```

Same, make sure this shell is executable.
```bash
chmod +x ./import_script.sh
```

### 4. Create Cron Job for Backup

Edit Cron Config
```
crontab -e
```

Add those line
```
30 3 * * * /path/to/backup_script.sh
```
I set run `backup_script.sh` at 3:30 everyday, the time is up to you.

Enable and start cron service
```bash
systemctl enable cron
systemctl start cron
```

Now, it will run `backup_script.sh` at 3:30 everyday automatically, enjoy!

## Refs

- https://www.ichiayi.com/tech/n8n_docker_install
- https://docs.n8n.io/hosting/cli-commands/
