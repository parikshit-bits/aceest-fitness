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

## API cURLs

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
python3 -m pytest test_app.py -v

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

**To configure in Jenkins:**
1. Create a new **Pipeline** project.
2. Under *Pipeline Definition*, select **Pipeline script from SCM**.
3. Set SCM to **Git** and provide your repository URL.
4. Set the script path to `Jenkinsfile`.
5. Save and click **Build Now**.

The `post` block cleans up the Docker image after every run and logs overall pass/fail status.

---

## Git Branching Strategy

| Branch     | Purpose                              |
|------------|--------------------------------------|
| `main`     | Production-ready, protected branch   |
| `develop`  | Integration branch for features      |
| `feature/` | Individual feature branches          |
| `fix/`     | Bug fix branches                     |

Commit message format: `<type>(<scope>): <short description>`  
Example: `feat(clients): add DELETE endpoint for client removal`# trigger
