# Preparing the AWS QuizApp on Raspberry Pi

- This tutorial starts with my Raspberry Pi environment already set up, with Ubuntu Server, Docker, and Docker Compose installed and ready to use.
- To learn more about Ubuntu server and Docker setup, check my other project: **https://github.com/Enrisox/Secure-Home-Lab-Docker**
- I will focus on preparing the AWS QuizApp project, creating the directory structure, Dockerfile, and docker-compose.yml for local testing before setting up CI/CD in step2.md.

## Create Project directory

```bash
mkdir ~/app1-test
cd ~/app1-test
```

## Create the Dockerfile in the Project directory
- The Dockerfile defines the container for the AWS Quiz App. It starts by setting the base image to python:3.11-slim, a lightweight Python 3.11 image that reduces the final size while providing a complete Python environment. Using the slim variant also minimizes the number of pre-installed packages, reducing the attack surface and improving security. The working directory inside the container is set to /app, so all subsequent commands, such as copying files or running commands, are executed relative to this folder.
- The requirements.txt file is copied from the host into the container’s /app directory, enabling Docker to install the Python dependencies needed for the application. All packages listed in requirements.txt are installed using pip install --no-cache-dir -r requirements.txt; the --no-cache-dir option prevents caching, keeping the image small.
- For security, a non-root system user named appuser is created with no login shell. Running the application as a non-root user is a best practice to reduce potential security risks. The main application files (app.py), the dataset (exam.txt), and the HTML templates (templates/) are copied into the container, providing all the code and resources the app needs to run.
- File ownership is changed to appuser using chown -R appuser:appuser /app, ensuring the non-root user has access to all application files. File permissions are then set to 550 recursively with chmod -R 550 /app, allowing read and execute access for the owner and group while preventing modification inside the container. The container is then switched to run as appuser with the USER command, so all subsequent commands and the application itself execute with restricted permissions.
- Port 9000 is exposed inside the container to allow communication from other containers or a reverse proxy like Caddy. Finally, the container starts the application using Gunicorn, a production-grade WSGI server. The command CMD ["gunicorn", "-b", "0.0.0.0:9000", "app:app"] binds the server to all network interfaces on port 9000 and tells Gunicorn to run the app object defined in app.py. This configuration ensures the Flask application runs securely, efficiently, and is ready to be accessed via the container network.

```bash
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

RUN useradd -r appuser

COPY app.py .
COPY exam.txt .
COPY templates/ ./templates/

RUN chown -R appuser:appuser /app
RUN chmod -R 550 /app
USER appuser

EXPOSE 9000

CMD ["gunicorn", "-b", "0.0.0.0:9000", "app:app"]
```

## Create Docker Compose Configuration

The docker-compose.yml file defines how the Quiz App container runs, which network it uses, port exposure, and security constraints. It allows easy management of the container and integration with other services like the reverse proxy (Caddy) and Cloudflare DDNS.
```bash
version: "3.8"
services:
  quizapp:
    image: enrisox/quizapp:${IMAGE_TAG}
    container_name: quizapp
    restart: unless-stopped

    networks:
      - ReverseProxy_net
    expose:
      - "9000"

    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    tmpfs:
      - /tmp

networks:
  ReverseProxy_net:
    external: true
```

- **image: enrisox/quizapp:${IMAGE_TAG}**: This dynamically uses the image tag set in the environment, allowing immutable deploys or latest for continuous updates.
- **restart: unless-stopped**: Ensures the container will auto-restart on failures but won’t restart if manually stopped.
- **networks: ReverseProxy_net**: Connects the container to a dedicated network shared with the reverse proxy (Caddy) and other internal services.
- **expose: "9000"**: Makes the application port accessible only to other containers; the reverse proxy handles public access.
- **security_opt and cap_drop**: Hardens the container by removing unnecessary Linux capabilities and preventing privilege escalation.
- **tmpfs: /tmp**: Stores temporary files in memory, improving security by avoiding sensitive data on disk.
- **networks.external: true**: The network is created outside Compose and can be shared with multiple services.

**Using Immutable Docker Image Tags vs latest – Security Perspective**

In Docker, an image can be referenced by a tag, like latest or a specific version/hash

**Why immutable tags are better for security:**

1. Reproducibility & auditability:
Using a tag tied to the commit hash or version (sha-<GIT_COMMIT>), the exact same image is deployed every time. This allows you to audit the image for vulnerabilities and know exactly what is running.
2. Vulnerability management:
Tools like Trivy scan a specific image tag. If you always pull latest, a new image could contain untested or vulnerable packages. With immutable tags, you scan once per build and know the deployed version is safe.
3. CI/CD consistency:
Immutable tags prevent “drift” in production. If a new image is pushed to latest with a bug or vulnerability, the pipeline could fail unexpectedly or deploy unsafe code.
4. Compliance & traceability:
Security policies often require traceable, auditable deployments. Tags like commit SHA give a clear chain of custody: which code, which image, which deployment.

**Example workflow:**

1. Build: CI pipeline builds the image with docker build -t enrisox/quizapp:sha-${GITHUB_SHA}.
2. Push: Push the tagged image to Docker Hub.
3. Scan: Trivy scans the exact tag.
4. Deploy: Self-hosted runner on Raspberry pulls the specific SHA-tagged image, ensuring only scanned, secure code is deployed.


## Application Files

- Create the templates/ directory:
This folder will contain HTML templates for the Flask app.

- Create index.html inside templates/:
This is the main page of the web app.

- Create app.py
This is the Python Flask application entry point
