# ACEest Fitness & Gym — CI/CD Pipeline Project

A Flask-based REST API for fitness and gym client management, fully containerised with
Docker and integrated with a CI/CD pipeline via GitHub Actions and Jenkins.

---

## Project Structure

```text
aceest-fitness/
├── app.py                        # Flask application
├── test_app.py                   # Pytest test suite
├── requirements.txt              # Python dependencies
├── Dockerfile                    # Container definition
├── Jenkinsfile                   # Jenkins pipeline
└── .github/
    └── workflows/
        └── main.yml              # GitHub Actions CI/CD pipeline
```

---

## API Endpoints

| Method | Route                  | Description                     |
|--------|------------------------|---------------------------------|
| GET    | `/`                    | Health check                    |
| GET    | `/programs`            | List all available programs      |
| GET    | `/programs/<name>`     | Get details of a program         |
| GET    | `/clients`             | List all clients                 |
| POST   | `/clients`             | Add or update a client           |
| GET    | `/clients/<name>`      | Get a specific client            |
| DELETE | `/clients/<name>`      | Delete a client                  |
| POST   | `/calories`            | Calculate daily calorie target   |
| POST   | `/progress`            | Log weekly adherence             |
| GET    | `/progress/<name>`     | Get a client's progress history  |

---

## API cURL Examples

### Health Check
```bash
curl http://localhost:5000/
```

### Programs

```bash
# List all programs
curl http://localhost:5000/programs

# Get details of a specific program
curl http://localhost:5000/programs/Beginner%20(BG)

# Other valid program names (URL-encoded):
# Fat%20Loss%20(FL)%20-%203%20day
# Muscle%20Gain%20(MG)%20-%20PPL
```

### Clients

```bash
# Add / update a client
curl -X POST http://localhost:5000/clients \
  -H "Content-Type: application/json" \
  -d '{"name": "Arjun", "age": 28, "weight": 75.0, "program": "Beginner (BG)"}'

# Add a fat loss client
curl -X POST http://localhost:5000/clients \
  -H "Content-Type: application/json" \
  -d '{"name": "Priya", "age": 24, "weight": 60.0, "program": "Fat Loss (FL) - 3 day"}'

# Add a muscle gain client
curl -X POST http://localhost:5000/clients \
  -H "Content-Type: application/json" \
  -d '{"name": "Ravi", "age": 30, "weight": 85.0, "program": "Muscle Gain (MG) - PPL"}'

# List all clients
curl http://localhost:5000/clients

# Get a specific client
curl http://localhost:5000/clients/Arjun

# Delete a client
curl -X DELETE http://localhost:5000/clients/Arjun
```

### Calorie Calculator

```bash
# Calculate calories for fat loss
curl -X POST http://localhost:5000/calories \
  -H "Content-Type: application/json" \
  -d '{"weight": 70.0, "program": "Fat Loss (FL) - 3 day"}'

# Calculate calories for muscle gain
curl -X POST http://localhost:5000/calories \
  -H "Content-Type: application/json" \
  -d '{"weight": 80.0, "program": "Muscle Gain (MG) - PPL"}'

# Calculate calories for beginner
curl -X POST http://localhost:5000/calories \
  -H "Content-Type: application/json" \
  -d '{"weight": 65.0, "program": "Beginner (BG)"}'
```

### Progress Tracking

```bash
# Log weekly adherence for a client
curl -X POST http://localhost:5000/progress \
  -H "Content-Type: application/json" \
  -d '{"client_name": "Arjun", "week": "Week 01", "adherence": 85}'

# Log another week
curl -X POST http://localhost:5000/progress \
  -H "Content-Type: application/json" \
  -d '{"client_name": "Arjun", "week": "Week 02", "adherence": 90}'

# Get all progress entries for a client
curl http://localhost:5000/progress/Arjun
```

---

## Local Setup & Execution

### Prerequisites
- Python 3.11+
- Docker Desktop

### Run locally (without Docker)
```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/aceest-fitness.git
cd aceest-fitness

# 2. Install dependencies
pip3 install -r requirements.txt

# 3. Start the Flask server
python3 app.py
# API available at http://localhost:5000
```

### Run with Docker
```bash
# Build the image
docker build -t aceest-fitness .

# Run the container
docker run -p 5000:5000 aceest-fitness

# API available at http://localhost:5000
```

---

## Running Tests Manually

```bash
# Without Docker
pytest test_app.py -v

# Inside Docker
docker run --rm aceest-fitness python -m pytest test_app.py -v --tb=short
```

---

## GitHub Actions — CI/CD Pipeline Overview

The pipeline is defined in `.github/workflows/main.yml` and is triggered automatically
on every **push** or **pull request** to the `main` or `develop` branch.

**Stages:**

1. **Build & Lint** — Installs Python dependencies and runs `flake8` to catch syntax
   errors and style violations before any build begins.
2. **Docker Image Assembly** — Builds the Docker image and verifies it was created
   successfully. This stage only runs after the lint job passes.
3. **Automated Testing in Container** — Rebuilds the Docker image and executes the
   full Pytest suite *inside the container*, confirming the app behaves correctly in
   the production-equivalent environment. This stage only runs after a successful Docker
   build.

Each stage is a dependency of the next, so a failure in any stage halts the pipeline.

---

## Jenkins BUILD Integration

The `Jenkinsfile` defines a declarative pipeline with six stages:

1. **Checkout** — Pulls latest source code from the connected GitHub repository.
2. **Build Environment** — Installs Python dependencies via `pip`.
3. **Lint** — Runs `flake8` as a quality gate for code style.
4. **Unit Tests** — Executes `pytest` directly on the host build agent.
5. **Docker Build** — Builds the Docker image tagged with the Jenkins build number.
6. **Quality Gate** — Runs the full Pytest suite inside the freshly built container as
   a final integration check.

The `post` block cleans up the Docker image after every run and logs overall pass/fail status.

---

## Jenkins Local Setup

### Step 1: Run Jenkins in Docker

```bash
docker run -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

Open `http://localhost:8080` in your browser.

### Step 2: Unlock Jenkins

Get the initial admin password:

```bash
docker exec $(docker ps -q --filter ancestor=jenkins/jenkins:lts) \
  cat /var/jenkins_home/secrets/initialAdminPassword
```

Paste it in the browser → click **Install suggested plugins** → create your admin user.

### Step 3: Install Python, pip and Docker CLI inside Jenkins container

Jenkins runs inside its own container and does not have Python or Docker by default.
Run these commands to install them:

```bash
# Get the Jenkins container ID
docker ps
# Copy the container ID, e.g. e76f1ce5cf47

# Install Python and pip
docker exec -u root <container_id> apt-get update
docker exec -u root <container_id> apt-get install -y python3 python3-pip docker.io

# Create symlinks so `pip` and `python` commands work
docker exec -u root <container_id> ln -s /usr/bin/python3 /usr/bin/python
docker exec -u root <container_id> ln -s /usr/bin/pip3 /usr/bin/pip

# Allow Jenkins to run Docker commands
docker exec -u root <container_id> chmod 666 /var/run/docker.sock

# Verify
docker exec <container_id> pip --version
docker exec <container_id> docker --version
```

Then restart Jenkins so the changes take effect:

```bash
docker restart <container_id>
```

> **Note:** pip installs packages to `/var/jenkins_home/.local/bin/` which is not on
> PATH by default. The Jenkinsfile uses the full path
> `/var/jenkins_home/.local/bin/flake8` to call flake8 directly.

### Step 4: Create the Pipeline job in Jenkins

1. Click **New Item**
2. Enter name `aceest-fitness` → select **Pipeline** → click **OK**
3. Under **Build Triggers** → check **GitHub hook trigger for GITScm polling**
4. Scroll to **Pipeline** section
5. Set **Definition** → `Pipeline script from SCM`
6. Set **SCM** → `Git`
7. Enter your repo URL → `https://github.com/<your-username>/aceest-fitness.git`
8. Set **Branch** to `*/main`
9. Set **Script Path** → `Jenkinsfile`
10. Click **Save**

### Step 5: Expose Jenkins via ngrok (for GitHub Webhooks)

Jenkins runs on localhost and is not reachable by GitHub by default.
ngrok creates a public tunnel to your local machine.

```bash
# Install ngrok
brew install ngrok

# In a new terminal tab, expose Jenkins port
ngrok http 8080
```

Copy the public URL ngrok gives you, e.g.:
```
https://abc123.ngrok-free.app
```

> Keep this terminal tab open — closing it stops the tunnel.

### Step 6: Add GitHub Webhook

1. Go to your GitHub repo → **Settings → Webhooks → Add webhook**
2. Fill in:
   - **Payload URL:** `https://abc123.ngrok-free.app/github-webhook/`
   - **Content type:** `application/json`
   - **Trigger:** Just the push event
3. Click **Add webhook**
4. GitHub sends a ping — you should see a ✅ green tick next to the webhook

### Step 7: Trigger the pipeline

Make any small commit and push to `main`:

```bash
echo "# trigger" >> README.md
git add .
git commit -m "ci: trigger Jenkins pipeline test"
git push origin main
```

Jenkins will auto-trigger via the webhook. Go to `http://localhost:8080` →
click your job → **Console Output** to watch it live.

### Expected successful output

```
Stage: Checkout           ✅
Stage: Build Environment  ✅
Stage: Lint               ✅
Stage: Unit Tests         ✅
Stage: Docker Build       ✅
Stage: Quality Gate       ✅
Finished: SUCCESS
```

---

## Git Branching Strategy

| Branch     | Purpose                              |
|------------|--------------------------------------|
| `main`     | Production-ready, protected branch   |
| `develop`  | Integration branch for features      |
| `feature/` | Individual feature branches          |
| `fix/`     | Bug fix branches                     |

Commit message format: `<type>(<scope>): <short description>`  
Example: `feat(clients): add DELETE endpoint for client removal`