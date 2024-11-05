
# SigVid

**SigVid** is a Signal-based video downloader and responder system. This tool listens for incoming messages on Signal, detects video links, downloads the videos, and sends them back to the original sender. Designed for secure media automation, SigVid offers options for data logging with either MySQL or SQLite for security and debugging.

## Features
- **Signal Integration**: Listens for Signal messages to detect video URLs.
- **Automated Video Downloads**: Downloads videos from links sent via Signal.
- **Responsive Media Sharing**: Sends the downloaded videos back to the original sender through Signal.
- **Logging and Data Storage**: Logs requests, responses, and errors to either MySQL or SQLite for secure data retention.
- **Storage Management**: Stores downloaded videos in structured directories for easy management.

## Requirements
- **Operating System**: Ubuntu 22.04 LTS
- **Tools**: Docker, Docker-Compose, Python, Signal-CLI REST API
- **Database**: MySQL or SQLite

## Installation

### 1. Initial Server Setup
1. Update and upgrade the system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. Install essential tools:
   ```bash
   sudo apt install -y curl wget git vim ufw
   ```

### 2. User and Docker Setup
1. Create a non-privileged user and add to the Docker group:
   ```bash
   sudo adduser nvd
   sudo usermod -aG sudo nvd
   sudo usermod -aG docker nvd
   ```

2. Install Docker and Docker-Compose:
   ```bash
   curl -fsSL https://get.docker.com | sudo bash
   sudo curl -L "https://github.com/docker/compose/releases/download/<latest_version>/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

### 3. Clone and Run the Signal-CLI REST API
1. Clone the Signal-CLI REST API:
   ```bash
   git clone https://github.com/bbernhard/signal-cli-rest-api.git /opt/NVD/signal-cli-rest-api
   cd /opt/NVD/signal-cli-rest-api
   docker-compose up -d
   ```

### 4. Set Up SigVid

1. Create directories and the Python script:
   ```bash
   mkdir -p /opt/NVD/data
   touch /opt/NVD/signal_listener.py
   ```

2. Create and populate `signal_listener.py`:
   - Paste the code that listens for Signal messages, detects URLs, downloads videos, and logs data.

3. Create a Dockerfile for SigVid:
   ```dockerfile
   FROM python:3.9-slim
   RUN apt-get update && apt-get install -y ffmpeg wget sqlite3 &&        pip install requests youtube_dl

   WORKDIR /opt/NVD
   COPY signal_listener.py /opt/NVD/signal_listener.py
   EXPOSE 8080
   CMD ["python", "/opt/NVD/signal_listener.py"]
   ```

4. Build and run the custom Docker image:
   ```bash
   cd /opt/NVD
   docker build -t custom-signal-listener .
   docker run -d      --name signal_listener      -v /opt/NVD:/opt/NVD      -v /tmp:/tmp      custom-signal-listener
   ```

### 5. Database Configuration
- Configure either MySQL or SQLite as per your logging and data retention needs.
- Update `signal_listener.py` to connect to the database of choice.

## Usage
1. **Signal Listener**: The container listens for messages containing video links on Signal.
2. **Video Download and Response**: Upon receiving a link, it downloads the video, logs the request, and sends the video back.
3. **Logging**: Logs requests, downloads, and response statuses to MySQL or SQLite for security and debugging.

## License
SigVid is licensed under the [MIT License](LICENSE).
