

```markdown
# Sparrow Deployment Notes (Docker)

## Overview
Working on the Sparrow deployment using Docker. Below are the key findings and configuration changes made during the setup process.

---

## 🔧 Changes & Configuration Updates

- ✅ **Modified `docker-compose.yml`** (fixed syntax issues).
- 🐳 **No need to build the Docker image manually** — the image is available on GitHub and can be pulled directly.
- ⚙️ **Updated `.env` file** with correct environment variables.
- ♻️ **Added restart policy** to ensure container resilience.
- 🔄 **Updated Kafka broker setting** in `.env`:
  ```env
  KAFKA_BROKER=kafka:9092
  ```
- 🔁 **Modified Kafka configuration in `docker-compose.yml`**:

  **Before:**
  ```yaml
  - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092,EXTERNAL://kafka:9094
  ```

  **After:**
  ```yaml
  - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094
  ```

---

## 📌 Reason for Kafka Configuration Changes

### Key Changes:
1. **Advertise Kafka's internal listener** as `PLAINTEXT://kafka:9092` for container-to-container communication.
2. **Advertise external listener** as `EXTERNAL://localhost:9094` to make Kafka accessible from the host machine.
3. **Expose both ports `9092` and `9094`** for internal and external connectivity.

---

## ⚠️ Architecture Compatibility Note

- ❌ **No prebuilt Docker images for AMD64** architecture are available on Docker Hub.
- This may lead to compatibility issues on AMD-based systems unless images are built locally or compatible images are published.

---

