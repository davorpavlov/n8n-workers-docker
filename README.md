
# 🚀Production-Ready n8n with Workers using Docker Compose
<p align="center">
  <img src="https://i.ibb.co/Df5QBX4T/Untitled-diagram-Mermaid-Chart-2025-08-22-211114.png" alt="n8n architecture diagram" width="600"/>
</p>
This repository provides a robust, scalable, and production-ready setup for n8n using Docker Compose. The architecture is designed for high performance by separating the main n8n instance (UI/editor) from the workflow execution engine, which is handled by one or more dedicated worker services.

This setup uses:

  * **n8n Main**: The primary service for the web interface and workflow management.
  * **n8n Worker(s)**: Dedicated services that execute the workflows, ensuring the main UI remains fast and responsive.
  * **PostgreSQL**: A powerful and reliable database for storing all your n8n data.
  * **Redis**: Used as a high-speed message broker for the execution queue.

[](https://opensource.org/licenses/MIT)

## Features

✅ **Scalable Architecture** – Easily scale workers for heavy workloads  
⚡ **High Performance** – Main instance stays fast while workers handle executions  
💾 **Data Persistence** – All data persists in Docker volumes  
🔒 **Secure by Default** – Secrets in `.env`, not hardcoded  
🩺 **Health Checks** – Services monitored for correct startup and health  
🚀 **Plug-and-Play** – Default settings work out-of-the-box


## Prerequisites

Before you begin, ensure you have the following installed on your server:

  * [Docker](https://docs.docker.com/engine/install/)
  * [Docker Compose](https://docs.docker.com/compose/install/)

## 🚀 Quickstart Guide

Getting started is simple. Follow these steps:

### 1️⃣ Clone the Repository

Clone this repository to your server:

```bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
```

### 2️⃣ Create Your Environment File

Copy the example environment file. This is where you'll store all your secrets and custom configurations.

```bash
cp .env.example .env
```

### 3️⃣ Generate an Encryption Key

The `N8N_ENCRYPTION_KEY` is the most critical variable. It is used to encrypt your credentials. **You must set this variable.**

Generate a secure, 32-character hexadecimal string with the following command:

```bash
# On Linux or macOS
openssl rand -hex 16
```


Click here for a Windows PowerShell command:

```powershell
$bytes = New-Object byte[] 16; [System.Security.Cryptography.RandomNumberGenerator]::Create().GetBytes($bytes); [BitConverter]::ToString($bytes).Replace('-', '').ToLower()
```



Copy the output of the command.

### 4️⃣ Customize Your .env File

Now, open the `.env` file you created and:

1.  Paste the generated key as the value for `N8N_ENCRYPTION_KEY`.
2.  Change the default passwords for `POSTGRES_PASSWORD` and `REDIS_PASSWORD` to something secure.
3.  For production, change `N8N_HOST` to your public-facing domain (e.g., `https://n8n.yourdomain.com`).

### 5️⃣ Deploy the Stack

You are now ready to launch the entire application stack with a single command:

```bash
docker compose up -d
```

Docker Compose will now pull the necessary images, create the volumes and network, and start all the services in the background. Your n8n instance will be available at `http://your_server_ip:5678` (or the port you defined in `.env`).

## ⚙️ Configuration (`.env` File)

All configuration is handled through the `.env` file. Here are the key variables:

| Variable | Description | Default Value |
| :--- | :--- | :--- |
| `N8N_ENCRYPTION_KEY` | **CRITICAL:** The secret key for encrypting credentials. **Must be set.** | `     ` |
| `POSTGRES_PASSWORD` | The password for the PostgreSQL database user. | `changeme_in_env_file` |
| `REDIS_PASSWORD` | The password for the Redis instance. | `changeme_in_env_file` |
| `N8N_HOST` | The public URL of your n8n instance. | `http://localhost:5678` |
| `POSTGRES_PORT` | The external port for the PostgreSQL database. | `5432` |
| `POSTGRES_USER` | The username for the PostgreSQL database. | `n8n` |
| `POSTGRES_DB` | The name of the PostgreSQL database. | `n8n` |
| `GENERIC_TIMEZONE` | The timezone for the n8n instance. | `Europe/Berlin` |
| `N8N_MAX_MEMORY` | Max memory (in MB) for the n8n main and worker nodes. | `8192` |

## 🛠️ Stack Management

Here are the basic commands to manage your n8n stack:

  * **Check Status**: See which containers are running.
    ```bash
    docker compose ps
    ```
  * **View Logs**: Follow the logs for a specific service (e.g., the main n8n instance).
    ```bash
    docker compose logs -f n8n-main
    ```
  * **Stop the Stack**: Stop and remove all containers.
    ```bash
    docker compose down
    ```
  * **Destroy the Stack⚠️**: Stop and remove containers, **and delete all associated volumes (ALL DATA WILL BE LOST)**.
    ```bash
    docker compose down -v
    ```

## 📈 Scaling Workers for Heavy Loads

This architecture is designed to be easily scaled. If you find your workflows are running slowly or you have a high volume of executions, you can add more workers with a single command.

To scale the number of workers to **3**, for example, run the following command:

```bash
docker compose up -d --scale n8n-worker=3
```

Docker Compose will automatically create and start additional `n8n-worker` containers. The new workers will connect to the same Redis queue and immediately start picking up jobs.

You can verify that the workers are running with `docker compose ps`:

```
NAME                IMAGE                            STATUS              PORTS
n8n-main            docker.n8n.io/n8nio/n8n:latest   running (healthy)   0.0.0.0:5678->5678/tcp
n8n-worker-1        docker.n8n.io/n8nio/n8n:latest   running (healthy)
n8n-worker-2        docker.n8n.io/n8nio/n8n:latest   running (healthy)
n8n-worker-3        docker.n8n.io/n8nio/n8n:latest   running (healthy)
postgres            postgres:17-alpine               running (healthy)   0.0.0.0:35432->5432/tcp
redis               redis:7-alpine                   running (healthy)
```

To scale back down, simply change the number (e.g., `--scale n8n-worker=1`).

## 💾 Data Persistence and Backups

All critical data is stored in Docker-managed volumes on your host machine. The locations are typically found in `/var/lib/docker/volumes/`.

  * `postgres_data`: Contains all your workflows, credentials, and execution logs.
  * `n8n_data`: Contains n8n configuration files.
  * `redis_data`: Contains queued job data.

**🛡️ Tip: Always implement a regular backup strategy for these volumes.**

## License

This project is licensed under the MIT License. See the [LICENSE](https://www.google.com/search?q=LICENSE) file for details.
