# Sentry v9

Sentry is a cross-platform crash reporting and aggregation platform. It provides real-time error tracking and exception logging for your web and mobile applications.

Ensure your server installed with Docker and Docker Compose.

## Install & Setup

### Step 1: Clone Sentry self-hosted repo

```bash
git clone https://github.com/getsentry/self-hosted.git
```

### Step 2: Switch to the v9 tag

```bash
cd self-hosted
git checkout tags/9.1.2
```

### Step 3: Create a `.env` file

```bash
cp .env.example .env
```

### Step 4: Create Sentry Key

```bash
docker-compose run web config generate-secret-key
```

And copy the key to the `.env` file.

### Step 5: Upgrade Sentry

```bash
docker-compose run --rm web upgrade
```

### Step 6: Create Volumes

```bash
docker volume create --name=sentry-data
docker volume create --name=sentry-postgres
```

### Step 7: Start Sentry

```bash
docker-compose up -d
```

### Step 8: Access Sentry

Open your browser and go to `http://<your-ip>:9000`.
