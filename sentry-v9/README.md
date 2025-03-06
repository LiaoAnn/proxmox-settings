# Sentry v9

Sentry is a cross-platform crash reporting and aggregation platform. It provides real-time error tracking and exception logging for your web and mobile applications.

Ensure your server installed with Docker and Docker Compose.

## Install & Setup

### Step 1: Clone Sentry self-hosted repo

```bash
git clone https://github.com/getsentry/self-hosted.git
```

### Step 2: Create a `.env` file

```bash
cd self-hosted
cp .env.example .env
```

### Step 3: Create Sentry Key

```bash
docker-compose run web config generate-secret-key
```

And copy the key to the `.env` file.

### Step 4: Upgrade Sentry

```bash
docker-compose run --rm web upgrade
```

### Step 5: Create Volumes

```bash
docker volume create --name=sentry-data
docker volume create --name=sentry-postgres
```

### Step 6: Start Sentry

```bash
docker-compose up -d
```

### Step 7: Access Sentry

Open your browser and go to `http://<your-ip>:9000`.
