

---

# 🛠️ Production Docker Setup with NGINX, SSL & Let's Encrypt

This guide sets up a **production-ready Docker environment** for the `sparrowapp-dev` application using:

- Docker Compose

---

## 📁 Project Structure

Ensure the following directory structure exists in your repository:

```
sparrowapp-dev/
│
├── docker-compose.yml
└── 
```

---

## ⚙️ `docker-compose.yml` for Production

```yaml
version: "3.8"

services:
  mongo:
    image: mongo:7.0
    container_name: mongo-db
    env_file:
      - .env.docker-setup
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    networks:
      - localnet

  kafka:
    image: bitnami/kafka:3.4.1
    container_name: kafka
    hostname: kafka
    ports:
      - "9092:9092" # Broker listener
      - "9093:9093" # Controller listener
      - "9094:9094" # External listener
    volumes:
      - kafka_data:/bitnami
    env_file:
      - .env.docker-setup
    network: 
      - localnet


  sparrow:
    image: sparrowapi/sparrow-api:v1
    ports:
      - "9000:9000"
      - "9001:9001"
      - "9002:9002"
    env_file:
      - .env.docker-setup
    networks:
      - localnet
    depends_on: 
      - mongo
      - kafka

  # Sparrow Auth Server
  sparrow-auth:
    image: sparrowapi/sparrow-auth:v1
    ports:
      - "1421:80"
    networks:
      - localnet
    depends_on:
      - sparrow-api
      - mongo
      - kafka
    env_file:
        - .env.docker-setup
    command:  >-
      sh -c "
        echo 'window.runtimeConfig = {
          VITE_API_URL: \"${VITE_API_URL}\",
          VITE_SPARROW_SUPPORT_EMAIL: \"${VITE_SPARROW_SUPPORT_EMAIL}\",
          VITE_SPARROW_OAUTH: \"${VITE_SPARROW_OAUTH}\",
          VITE_ENABLE_MIX_PANEL: \"${VITE_ENABLE_MIX_PANEL}\",
          VITE_MIX_PANEL_TOKEN: \"${VITE_MIX_PANEL_TOKEN}\",
          VITE_TERMS_OF_SERVICE: \"${VITE_TERMS_OF_SERVICE}\",
          VITE_SPARROW_WEB_URL: \"${VITE_SPARROW_WEB_URL}\",
          VITE_SPARROW_PRIVACY_POLICY: \"${VITE_SPARROW_PRIVACY_POLICY}\"
        }' > /usr/share/nginx/html/runtime-config.js && nginx -g 'daemon off;'
      "

  # Sparrow Proxy Service
  sparrow-proxy:
    image: sparrowapi/sparrow-proxy-service:v1
    platform: linux/amd64
    ports:
      - "3000:3000"
    networks:
      - localnet

  # Sparrow Web App
  sparrow-web:
    build:
      context: .
      dockerfile: Sparrow-Web.Dockerfile
      args:
        VITE_WEB_API_TIMEOUT: 5000
        VITE_WEB_API_URL: http://localhost:9000
        VITE_WEB_AUTH_URL: http://localhost:1421
        VITE_WEB_SOCKET_IO_API_URL: http://localhost:9001
        VITE_WEB_SPARROW_OAUTH: http://localhost:9000/api/auth/google/callback
        VITE_WEB_TERMS_OF_SERVICE: https://example.dev/termsandconditions
        VITE_WEB_BASE_URL: http://localhost:9000
        VITE_WEB_SPARROW_GITHUB: https://github.com/sparrowapp-dev
        VITE_WEB_SPARROW_LINKEDIN: https://www.linkedin.com/showcase/sparrow-app/
        VITE_WEB_SPARROW_DOWNLOAD_LINK: https://github.com/sparrowapp-dev/sparrow-app/releases
        VITE_WEB_RELEASE_NOTES_API: https://api.github.com/repos/sparrowapp-dev/sparrow-app/releases
        VITE_WEB_SPARROW_DOCS: https://docs.sparrowapp.dev/docs/intro
        VITE_WEB_PROXY_SERVICE: http://localhost:3000
        VITE_WEB_ENABLE_MIX_PANEL: false
        VITE_WEB_MIX_PANEL_TOKEN:
        VITE_WEB_SPARROW_AI_WEBSOCKET: "ws://localhost:9000/ai-assistant"

    ports:
      - "1422:80"
    networks:
      - localnet
    depends_on:
      - sparrow-proxy
      - sparrow-api
      - sparrow-auth
      - mongo
      - kafka

volumes:
  mongo_data:
  kafka_data:
    driver: local

networks:
  localnet:
    driver: bridge
```

---

## 🚀 Running the Setup

Make sure Docker and Docker Compose are installed on your system.

Then, run:

```bash
docker-compose up -d
```

---

## 📝 Notes

- ✅ Make sure DNS for `your_domain.com` points to your server's IP.
- 🔁 Run Certbot manually to generate initial certificates:
  ```bash
  docker run -it --rm \
    -v ./volumes/nginx/letsencrypt:/etc/letsencrypt \
    -v ./volumes/nginx/lib/letsencrypt:/var/lib/letsencrypt \
    -v ./volumes/nginx/www:/var/www/certbot \
    certbot/certbot certonly \
    --webroot --webroot-path=/var/www/certbot \
    --email your_email@example.com \
    --agree-tos --no-eff-email \
    -d your_domain.com
  ```

- ✅ Set up a cron job or use a script for **auto-renewing** SSL certificates.

---

