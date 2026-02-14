# 🚀 Unified DevOps & CI/CD Documentation

This guide outlines the end-to-end process of setting up a Linux-based DevOps environment, containerizing applications with Docker, and automating deployment using Jenkins.

## 1. System Environment Setup
The foundation requires updating the system and installing core communication and version control tools.

- **System Update:** Ensure all packages are current using:
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```
- **SSH Configuration:** Install and enable the OpenSSH server to allow remote management and firewall access.
- **Git Installation:** Install `git` and `openssh-client` to manage source code.
- **Java Environment:** Install OpenJDK (version 17 or 21) as it is a prerequisite for running Jenkins.

## 2. Docker Infrastructure & Manual Deployment
Docker is used to package applications into portable containers.

- **Installation:** Install Docker Engine and associated plugins (`buildx`, `compose`).
- **Permissions:** Add the current user and the Jenkins user to the `docker` group to execute commands without `sudo`.
- **Nginx Static Hosting:** You can quickly host static sites by mounting a local directory to the Nginx container.

```bash
docker run -d --name web-container -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx:alpine
```

## 3. Application Preparation
Before automation, applications must be verified locally.

- **Node.js:** Install the LTS version of Node.js and `build-essential` for compiling dependencies.
- **Local Build:** Clone the repository, run `npm install`, and test the development server or build production assets (e.g., `npm run build`).

## 4. Jenkins CI/CD Pipeline Architectures
Two primary methods for automation were established.

### Method A: Chained Freestyle Jobs
This approach uses three separate jobs that trigger one another in a sequence.
- **Clone Job:** Pulls the latest source code from the Git repository.
- **Build Image Job:** Triggered by the Clone Job; it enters the workspace and runs:
  ```bash
docker build -t app-image .
``` 
- **Run Container Job:** Triggered by the Build Job; it stops/removes old containers and starts a new one:
  ```bash
docker run -d -p 80:3000 --name running-app app-image
``` 

### Method B: Unified Multi-Stage Pipeline
A single pipeline script can handle the entire lifecycle, providing a "SUCCESS" or "FAILURE" result for the whole flow.
- **Stages:** Clean Workspace → Clone → Verify Environment → Install/Build → Docker Image Build → Container Deployment.
- **Deployment:** The final result is an application accessible via the server IP on a designated port (e.g., port 3000).

## 5. Validation & Maintenance
- **Verification:** Use `docker ps` to ensure container status is "Up" and port mappings are correct.
- **Logs:** Review Jenkins console output to confirm "Finished: SUCCESS" for each pipeline run.
- **Cleanup:** Use `docker stop` and `docker rm` to manage container lifecycles during updates.
