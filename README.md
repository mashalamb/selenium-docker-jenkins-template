# End-to-End Selenium CI/CD Test Automation Pipeline

An automated browser testing framework built with **Python**, **Selenium 4**, and **pytest**, containerized with **Docker**, and orchestrated via a **Jenkins** CI/CD pipeline.

---

## Architecture Flow

```text
+-------------------+       1. git push        +-------------------+

|                   | -----------------------> |                   |
|  Developer / QA   |                          |      GitHub       |
|   (Local Repo)    |                          |   (Source Code)   |
+-------------------+                          +-------------------+
                                                         |
                                                         | 2. Checkout SCM
                                                         v
+-------------------+     4. docker cp /       +-------------------+

|                   |        Archive Report    |                   |
|  Jenkins Build    | <----------------------- |      Jenkins      |
|   (report.html)   |                          |  (Pipeline Agent) |
+-------------------+                          +-------------------+
                                                         |
                                                         | 3. docker build & run
                                                         v
                                               +-------------------+

                                               |  Docker Container |
                                               |  - Python 3.12    |
                                               |  - pytest         |
                                               |  - Headless Chrome|
                                               +-------------------+
                                                         |
                                                         | Automated UI Tests
                                                         v
                                               +-------------------+

                                               |   Target Web App  |
                                               |  (Login Testing)  |
                                               +-------------------+
```

### How It Works
1. **Version Control (GitHub):** Stores the test suite (`tests/`), dependencies (`requirements.txt`), container recipe (`Dockerfile`), and pipeline definition (`Jenkinsfile`).
2. **CI Orchestration (Jenkins):** Pulls the latest code from GitHub, triggers a Docker image build, executes the test container, archives `reports/report.html`, and cleans up temporary containers/images.
3. **Isolated Execution (Docker):** Bundles Python, `pytest`, Selenium, and Google Chrome in headless mode (`--headless=new`) so tests run identically on any machine without driver or version conflicts.
4. **Automated UI Validation (Selenium + pytest):** Executes both **positive** (`test_valid_login`) and **negative** (`test_invalid_password`) test cases using explicit waits (`WebDriverWait`) and assertions.

---

## Project Structure

```text
selenium-docker-jenkins-template/
├── tests/
│   └── test_login_form.py   # Positive and negative login test cases + pytest fixture
├── Dockerfile               # Python 3.12 + Headless Google Chrome test runner image
├── Jenkinsfile              # Declarative CI/CD pipeline (Build, Test, Report, Cleanup)
├── requirements.txt         # Python dependencies (selenium, pytest, pytest-html)
├── .gitignore               # Excludes cache folders and generated reports
└── README.md                # Project documentation
```

---

## Running Locally

### 1. Run Directly on Host (Python + pytest)
```bash
# Run with visible browser window
pytest -v

# Run in headless mode (no GUI)
HEADLESS=true pytest -v
```

### 2. Run Inside Docker
```bash
docker build -t practice-login-tests .
docker run --rm --shm-size=2g -v "\$(pwd)/reports:/app/reports" practice-login-tests
```

---

## Test Coverage

| Test Case | Type | Description | Expected Result |
| :--- | :--- | :--- | :--- |
| `test_valid_login` | Positive | Submits valid credentials (`student` / `Password123`) | Redirects to `/logged-in-successfully/` and displays `Logged In Successfully` |
| `test_invalid_password` | Negative | Submits invalid password (`student` / `WrongPassword`) | Displays visible error banner reading `Your password is invalid!` |
