# 📃 Outline Service

Running Services:


:::info

1. Outline               → outline
2. Postgres              → postgres 
3. Redis                 → redis
4. Minio                 → minio-storage-1
5. Nginx                 → https-portal

:::


* ***Outline Stack Docker Compose***

```yaml
services:

  outline:
    image: outlinewiki/outline
    env_file: ./docker.env
    hostname: outline
    container_name: outline
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - redis
    networks:
      - outline

  redis:
    image: redis
    env_file: ./docker.env
    hostname: redis
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - ./redis.conf:/redis.conf
    command: ["redis-server", "/redis.conf"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 30s
      retries: 3
    networks:
      - outline

  postgres:
    image: postgres
    hostname: postgres
    container_name: postgres
    env_file: ./docker.env
    ports:
      - "5432:5432"
    volumes:
      - database-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 30s
      timeout: 20s
      retries: 3
    environment:
      POSTGRES_USER: 'POSTGRES_USER'
      POSTGRES_PASSWORD: 'POSTGRES_PASSWORD'
      POSTGRES_DB: 'outline'
    networks:
      - outline

volumes:
  database-data:

networks:
  outline:
    name: outline
    external: true
```


* ***Minio Stack Docker Compose***

```yaml
services:
  storage:
    image: quay.io/minio/minio
    ports:
      - "9000:9000"
      - "9001:9001"
    entrypoint: sh
    command: -c 'minio server --console-address ":9001" /data/minio'
    deploy:
      restart_policy:
        condition: on-failure
    volumes:
      - storage-data:/data
    environment:
      MINIO_ROOT_USER: 'ADMIN_USER_NAME'
      MINIO_ROOT_PASSWORD: 'ADMIN_PASSWORD'
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
    networks:
      - outline

volumes:
  storage-data:

networks:
  outline:
    name: outline
```


* ***NGINX Stack Docker Compose***

```yaml
services:

  https-portal:
    image: steveltn/https-portal
    hostname: https-portal
    container_name: https-portal
    env_file: ./docker.env
    ports:
      - '80:80'
      - '443:443'
    restart: always
    volumes:
      - https-portal-data:/var/lib/https-portal
    healthcheck:
      test: ["CMD", "service", "nginx", "status"]
      interval: 30s
      timeout: 20s
      retries: 3
    environment:
      DOMAINS: 'wiki.example.com -> http://outline:3000, storage.example.com -> http://storage:9000'
      STAGE: 'production'
      WEBSOCKET: 'true'
    networks:
      - outline

volumes:
  https-portal-data:

networks:
  outline:
    name: outline
    external: true
```


* ***ENV File***


:::warning
Replace the below parameters with their correct value in ENV file and store it as `docker.env`.

```markup
SECRET_KEY
UTILS_SECRET
DB_USER
DB_PASSWORD
MINIO_ACCESS_KEY
MINIO_SECRET_KEY
AWS_S3_UPLOAD_BUCKET_URL
SMTP_USER_PASSWORD
URL

```

:::

```none
# –––––––––––––––– REQUIRED ––––––––––––––––

NODE_ENV=production

# Generate a hex-encoded 32-byte random key. You should use `openssl rand -hex 32`
# in your terminal to generate a random value.
SECRET_KEY=GENERATE_RANDOM_VALUE

# Generate a unique random key. The format is not important but you could still use
# `openssl rand -hex 32` in your terminal to produce this.
UTILS_SECRET=GENERATE_RANDOM_VALUE

# For production point these at your databases, in development the default
# should work out of the box.
DATABASE_URL=postgres://DB_USER:DB_PASSWORD@postgres:5432/outline
DATABASE_URL_TEST=postgres://DB_USER:DB_PASSWORD@postgres:5432/outline-test
DATABASE_CONNECTION_POOL_MIN=
DATABASE_CONNECTION_POOL_MAX=
# Uncomment this to disable SSL for connecting to Postgres
PGSSLMODE=disable

# For redis you can either specify an ioredis compatible url like this
REDIS_URL=redis://redis:6379
# or alternatively, if you would like to provide additional connection options,
# use a base64 encoded JSON connection option object. Refer to the ioredis documentation
# for a list of available options.
# Example: Use Redis Sentinel for high availability
# {"sentinels":[{"host":"sentinel-0","port":26379},{"host":"sentinel-1","port":26379}],"name":"mymaster"}
# REDIS_URL=ioredis://eyJzZW50aW5lbHMiOlt7Imhvc3QiOiJzZW50aW5lbC0wIiwicG9ydCI6MjYzNzl9LHsiaG9zdCI6InNlbnRpbmVsLTEiLCJwb3J0IjoyNjM3OX1dLCJuYW1lIjoibXltYXN0ZXIifQ==

# URL should point to the fully qualified, publicly accessible URL. If using a
# proxy the port in URL and PORT may be different.
URL=https://wiki.example.com
PORT=

# See [documentation](docs/SERVICES.md) on running a separate collaboration
# server, for normal operation this does not need to be set.
COLLABORATION_URL=

# To support uploading of images for avatars and document attachments an
# s3-compatible storage must be provided. AWS S3 is recommended for redundancy
# however if you want to keep all file storage local an alternative such as
# minio (https://github.com/minio/minio) can be used.

# A more detailed guide on setting up S3 is available here:
# => https://wiki.generaloutline.com/share/125de1cc-9ff6-424b-8415-0d58c809a40f
#
AWS_ACCESS_KEY_ID=MINIO_ACCESS_KEY
AWS_SECRET_ACCESS_KEY=MINIO_SECRET_KEY
AWS_REGION=local
AWS_S3_ACCELERATE_URL=
AWS_S3_UPLOAD_BUCKET_URL=https://storage.example.com
AWS_S3_UPLOAD_BUCKET_NAME=outline
AWS_S3_UPLOAD_MAX_SIZE=26214400
AWS_S3_FORCE_PATH_STYLE=true
AWS_S3_ACL=private


# –––––––––––––– AUTHENTICATION ––––––––––––––

# Third party signin credentials, at least ONE OF EITHER Google, Slack,
# or Microsoft is required for a working installation or you'll have no sign-in
# options.

# To configure Slack auth, you'll need to create an Application at
# => https://api.slack.com/apps
#
# When configuring the Client ID, add a redirect URL under "OAuth & Permissions":
# https://<URL>/auth/slack.callback
SLACK_CLIENT_ID=get_a_key_from_slack
SLACK_CLIENT_SECRET=get_the_secret_of_above_key

# To configure Google auth, you'll need to create an OAuth Client ID at
# => https://console.cloud.google.com/apis/credentials
#
# When configuring the Client ID, add an Authorized redirect URI:
# https://<URL>/auth/google.callback
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# To configure Microsoft/Azure auth, you'll need to create an OAuth Client. See
# the guide for details on setting up your Azure App:
# => https://wiki.generaloutline.com/share/dfa77e56-d4d2-4b51-8ff8-84ea6608faa4
AZURE_CLIENT_ID=
AZURE_CLIENT_SECRET=
AZURE_RESOURCE_APP_ID=

# To configure generic OIDC auth, you'll need some kind of identity provider.
# See documentation for whichever IdP you use to acquire the following info:
# Redirect URI is https://<URL>/auth/oidc.callback
OIDC_CLIENT_ID=
OIDC_CLIENT_SECRET=
OIDC_AUTH_URI=
OIDC_TOKEN_URI=
OIDC_USERINFO_URI=

# Specify which claims to derive user information from
# Supports any valid JSON path with the JWT payload
OIDC_USERNAME_CLAIM=preferred_username

# Display name for OIDC authentication
OIDC_DISPLAY_NAME=OpenID

# Space separated auth scopes.
OIDC_SCOPES=openid profile email


# –––––––––––––––– OPTIONAL ––––––––––––––––

# Base64 encoded private key and certificate for HTTPS termination. This is only
# required if you do not use an external reverse proxy. See documentation:
# https://wiki.generaloutline.com/share/1c922644-40d8-41fe-98f9-df2b67239d45
SSL_KEY=
SSL_CERT=

# If using a Cloudfront/Cloudflare distribution or similar it can be set below.
# This will cause paths to javascript, stylesheets, and images to be updated to
# the hostname defined in CDN_URL. In your CDN configuration the origin server
# should be set to the same as URL.
CDN_URL=

# Auto-redirect to https in production. The default is true but you may set to
# false if you can be sure that SSL is terminated at an external loadbalancer.
FORCE_HTTPS=true

# Have the installation check for updates by sending anonymized statistics to
# the maintainers
ENABLE_UPDATES=true

# How many processes should be spawned. As a reasonable rule divide your servers
# available memory by 512 for a rough estimate
WEB_CONCURRENCY=1

# Override the maximum size of document imports, could be required if you have
# especially large Word documents with embedded imagery
MAXIMUM_IMPORT_SIZE=5120000

# You can remove this line if your reverse proxy already logs incoming http
# requests and this ends up being duplicative
DEBUG=http

# Configure lowest severity level for server logs. Should be one of
# error, warn, info, http, verbose, debug and silly
LOG_LEVEL=info

# For a complete Slack integration with search and posting to channels the
# following configs are also needed, some more details
# => https://wiki.generaloutline.com/share/be25efd1-b3ef-4450-b8e5-c4a4fc11e02a
#
# SLACK_VERIFICATION_TOKEN=
# SLACK_APP_ID=
# SLACK_MESSAGE_ACTIONS=true

# Optionally enable google analytics to track pageviews in the knowledge base
GOOGLE_ANALYTICS_ID=

# Optionally enable Sentry (sentry.io) to track errors and performance,
# and optionally add a Sentry proxy tunnel for bypassing ad blockers in the UI:
# https://docs.sentry.io/platforms/javascript/troubleshooting/#using-the-tunnel-option)
SENTRY_DSN=
SENTRY_TUNNEL=

# To support sending outgoing transactional emails such as "document updated" or
# "you've been invited" you'll need to provide authentication for an SMTP server
SMTP_HOST=mail.example.com
SMTP_PORT=587
SMTP_USERNAME=wiki@example.com
SMTP_PASSWORD=SMTP_USER_PASSWORD
SMTP_FROM_EMAIL=wiki@example.com
SMTP_REPLY_EMAIL=wiki@example.com
SMTP_TLS_CIPHERS=TLSv1.2
SMTP_SECURE=false
ENABLED_SIGNIN_EMAIL=true

# The default interface language. See translate.getoutline.com for a list of
# available language codes and their rough percentage translated.
DEFAULT_LANGUAGE=en_US

# Optionally enable rate limiter at application web server
RATE_LIMITER_ENABLED=true

# Configure default throttling parameters for rate limiter
RATE_LIMITER_REQUESTS=1000
RATE_LIMITER_DURATION_WINDOW=60
```


# Installing Steps



1.  Install Minio Service with below command

```bash
docker compose up -d
```


2. Go through Minio Console using `{SERVER_IP}:9001` and create a new **Bucket** and named it `outline`
3. Go to ***Access Keys***  section and create a new ***access*** and ***secret*** key for outline.
4. Navigate throw ***Adminstrator → Buckets*** and select **outline** bucket. In **Summary** page, change **Access Policy** to **public. (This handles bucket public folder permissions for outline)**
5. Replace ENV parameters with correct values
6. Migrate outline DB using the below command

```bash
docker compose run --rm outline yarn db:migrate --env=production-ssl-disabled
```


7.  Start outline stack with below command

```bash
docker compose up -d
```


8. Install NGINX stack with the below command

```bash
docker compose up -d
```


9. Wait for the Service to generate SSL certs for specified domains for both outline and minio storage


:::tip
You can watch the process via `docker logs -f https-portal`. If the process passed. you can step through the process.

:::

:::tip
Change `wiki.example.com` and `storage.example.com` with your correct domain name in both `docker.env` and 
`nginx` docker-compose file.

:::

:::warning
Ensure that CDN Proxy is turend off in generating ssl certs part. 

Cloudflare    → Proxy Button

Arvan            → نماد ابر

After All certs being generated successfully, you can again turn them ON from CDN panel

:::


10. Use the below command to create ***ADMIN USER***

```bash
docker exec -it outline node build/server/scripts/seed.js ADMIN_EMAIL_ADDRESS

e.g -> docker exec -it outline node build/server/scripts/seed.js iman@example.com
```


11. Navigate Through the given link from the above command and login to outline for the first time with one time generated token.
12. Invite Team members :grinning: