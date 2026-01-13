# Testing and Security Scanning with Pytest and Trivy

This step describes how the GitHub Actions pipeline ensures code correctness and container security before deployment. The two main components are **Pytest** for functional testing of the Flask application and **Trivy** for vulnerability scanning of Docker images.

## Pytest: automated testing

I created a "**tests**" folder in my repository and a **test_app.py** inside, with this code:

```python
import sys
from pathlib import Path

# Adds the project root (where app.py is located) to the PYTHONPATH
ROOT = Path(__file__).resolve().parent.parent
sys.path.append(str(ROOT))

from app import app

def test_index_returns_200():
    client = app.test_client()
    response = client.get("/")
    assert response.status_code == 200
```

 Test case for the root endpoint ("/") of the Flask application.

    Steps:
    1. Create a test client using Flask's built-in test_client().
    2. Send a GET request to the "/" route.
    3. Assert that the HTTP status code of the response is 200 (OK).

**Note: In the future, this test file will be expanded with additional tests for
    other endpoints, functionality, and edge cases to ensure complete coverage.**

This test verifies that the main page of the web application is reachable and responds correctly.
    """
    client = app.test_client()      # Instantiate a test client to simulate requests
    response = client.get("/")      # Perform a GET request to the root URL
    assert response.status_code == 200  # Check that the response is HTTP 200 OK

# Container Security Scanning with Trivy
## Why I used Trivy

- Detects **high and critical vulnerabilities** before deployment.
- Works with **Docker Hub images** or local builds.
- Integrates seamlessly with **GitHub Actions**.
- Can enforce security policies in CI/CD pipelines.

## GitHub Actions integration (already present in deploy.yaml)

The `scan-image` job in GitHub Actions performs:

1. **Docker login** (so Trivy can pull private images if necessary):
```yaml
- name: Docker login for Trivy
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

2. **Scan the image:**
```yaml
- name: Scan image with Trivy
  uses: aquasecurity/trivy-action@0.28.0
  with:
    scan-type: image
    image-ref: enrisox/quizapp:sha-${{ github.sha }}
    severity: HIGH,CRITICAL
    ignore-unfixed: true
    format: table
    exit-code: "1"
```
**Explanation**

- scan-type: image → scans a container image.
- image-ref → points to the image built in the previous step.
- severity: HIGH,CRITICAL → only considers high and critical vulnerabilities for pipeline failure.
- ignore-unfixed: true → ignores vulnerabilities with no available fix to reduce noise.
- format: table → human-readable table output.
- exit-code: "1" → fails the CI job if vulnerabilities are found, preventing deployment of unsafe images.

### Do you need local Trivy?

**No, not for the GitHub Actions pipeline.**
- The pipeline uses the Trivy Action which includes and runs Trivy for you.

**Yes, if you want to:**

- run manual scans locally (e.g., before opening a PR),
- scan images stored locally or in other registries,
- check additional things like filesystem or IaC config, not just container images.

Manual local scan of a Docker image on a server:
```yaml
sudo apt install trivy
trivy image dockerhub-username/image-name:latest   #or image tag
```

