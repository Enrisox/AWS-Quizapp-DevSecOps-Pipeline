# Hardening and Deployment Strategy for the QuizApp Container

## Dockerfile Hardening Strategy

The Dockerfile has been optimized following two fundamental security pillars: reducing the attack surface and applying the Principle of Least Privilege.

1. **Principle of Least Privilege (appuser User)**
This is the most important defense against "Container Escape" attacks.
- **Non-Root User**: The Gunicorn application runs with an unprivileged user (instead of root).
- **Damage Mitigation**: If an attacker compromises my app, they only get the limited permissions of that user, preventing them from accessing or damaging the host operating system.
- **Permissions granted**: chmod -R 550: The code is set to read and execute only (550 = User: Read/Execute, Group: Read/Execute, Others: None) for the created user, protecting my code from modifications.
- **Attack Surface Reduction**:The Attack Surface is the set of all points where an unauthorized user can attempt to enter or extract data from the system. The larger the image, the more tools, libraries, and files there are, and the greater the attack surface.

Why choose Python:3.11-slim instead of the standard or latest version?

- **Vulnerability Reduction**: Standard Python or Debian images include hundreds of packages, utilities, and configuration files that the Flask application doesn't use. Each of these packages (e.g., a text editor, a graphics library, etc.) can contain vulnerabilities.
- **Reduced Size**: A slim or alpine image is much smaller, which speeds up the build and pull on your pipeline (particularly useful on the Raspberry Pi, which has limited resources).

The Principle of Least Privilege states that every entity (a user, a process, a container) must have only and exclusively the permissions necessary to perform its task and nothing else.

**The USER appuser Choice and Local Permissions**
By default, if you don't specify the user, Docker runs processes as root.

When a container runs as root:

- If an attacker exploits a vulnerability (e.g., a bug in your Flask code) and gains access, the attacker gets root permissions inside the container.
- With root permissions inside the container, it's much easier for the attacker to attempt to "escape" from the container and access the host operating system
- **Damage Mitigation**: If the attacker compromises the Gunicorn process, they only get the permissions of the appuser user. The appuser user doesn't have permissions to access system files, install software, or perform destructive actions on the Raspberry Pi.
- **Local Permissions (chmod -R 550 /app)**: This instruction ensures that appuser can only read and execute the necessary files (app.py, exam.txt, etc.), but cannot write to them. This prevents an attacker from modifying or injecting malicious code into the application files themselves. If an attack succeeds, the damage is confined only to the unprivileged user inside the container and does not propagate to the host server.

**Verifying the Running User**

sudo docker exec -it quizapp whoami

## Container Read-Only (--read-only)

docker run -d \
  --read-only \
  --name quizapp \
  -p 9000:9000 \
  enrisox/quizapp

