<!-- TOC -->

- [Learning-GitHub-Actions](#learning-github-actions)
  - [Workflow Overview](#workflow-overview)
  - [GitHub Actions shown in this workflow](#github-actions-shown-in-this-workflow)
  - [Getting Started](#getting-started)
  - [Deep Dive into GitHub Actions Workflow](#deep-dive-into-github-actions-workflow)
    - [Job1: `sast_scan`](#job1-sast_scan)
      - [Step 1: Checkout code](#step-1-checkout-code)
      - [Step 2: Set up Python](#step-2-set-up-python)
      - [Step 3: Install Bandit](#step-3-install-bandit)
      - [Step 4: Run Bandit Scan](#step-4-run-bandit-scan)
      - [Step 5: Upload Bandit Scan Artifact](#step-5-upload-bandit-scan-artifact)
    - [Job2: `image_scan`](#job2-image_scan)
      - [Step 1: Checkout code](#step-1-checkout-code-1)
      - [Step 2: Set up Docker](#step-2-set-up-docker)
      - [Step 3: Build Docker Image](#step-3-build-docker-image)
      - [Step 4: Docker Scout Scan](#step-4-docker-scout-scan)
      - [Step 5: Upload Docker Scout Scan Artifact](#step-5-upload-docker-scout-scan-artifact)
  - [Acknowledgment](#acknowledgment)

<!-- /TOC -->

# Learning-GitHub-Actions
Welcome to the "Learning-GitHub-Actions" project! This repository is designed for beginners to understand and practice GitHub Actions, utilizing a sample workflow inspired by the OWASP PyGoat intentionally vulnerable web project.

## Workflow Overview

The GitHub Actions workflow defined in `.github/workflows/main.yml` consists of two jobs:

1. **sast_scan:** This job checks the codebase for security issues using Bandit, a Python static analysis tool. The findings are stored as an artifact.

2. **image_scan:** This job builds a Docker image using the provided Dockerfile and performs security scans using Docker Scout. The scan results are saved as an artifact.

## GitHub Actions shown in this workflow
- [checkout](https://github.com/actions/checkout)
- [setup-python](https://github.com/actions/setup-python)
- [Bandit](https://github.com/marketplace/actions/bandit-action)
- [upload-artifact](https://github.com/actions/upload-artifact)
- [setup-docker](https://github.com/docker-practice/actions-setup-docker)
- [docker-scout-action](https://github.com/docker/scout-action)



## Getting Started

1. **Clone the Repository:**
   ```bash
    git clone https://github.com/your-username/Learning-GitHub-Actions.git
    cd Learning-GitHub-Actions
   ```

2. **Run GitHub Actions Workflow:**
   - Manually trigger the workflow by navigating to the "Actions" tab on GitHub and selecting "Run workflow" for the desired workflow.

3. **Explore Results:**
   - View Bandit scan results in the bandit-findings artifact.
   - Check Docker Scout scan findings in the docker-scout-findings artifact.

## Deep Dive into GitHub Actions Workflow
Let's break down the YAML code by each job level, then explain each steps inside it:

### Job1: `sast_scan`

```yaml
sast_scan:
  name: Run Bandit Scan
  runs-on: ubuntu-latest

  steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v3
      with:
        python-version: 3.8

    - name: Install Bandit
      run: pip install bandit

    - name: Run Bandit Scan
      run: bandit -ll -ii -r . -f json -o bandit-report.json

    - name: Upload Bandit Scan Artifact
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: bandit-findings
        path: bandit-report.json
```

Let's break down the `sast_scan` job into individual steps:

#### Step 1: Checkout code
```yaml
- name: Checkout code
  uses: actions/checkout@v3
```
- **Explanation:** This step uses the `actions/checkout` action to fetch the repository code. It ensures that the latest code from the repository is available for the subsequent steps.

#### Step 2: Set up Python
```yaml
- name: Set up Python
  uses: actions/setup-python@v3
  with:
    python-version: 3.8
```
- **Explanation:** This step uses the `actions/setup-python` action to set up Python version 3.8. It ensures that the correct Python version is available for the Bandit tool.

#### Step 3: Install Bandit
```yaml
- name: Install Bandit
  run: pip install bandit
```
- **Explanation:** This step installs the Bandit tool using the `pip` package manager. It ensures that Bandit is available for static analysis of Python code.

#### Step 4: Run Bandit Scan
```yaml
- name: Run Bandit Scan
  run: bandit -ll -ii -r . -f json -o bandit-report.json
```
- **Explanation:**
  - `-ll`: Log level set to low, displaying only critical issues.
  - `-ii`: Confidence level set to high, reporting only high-confidence issues.
  - `-r .`: Recursive scan of the current directory and its subdirectories.
  - `-f json`: Output the scan results in JSON format.
  - `-o bandit-report.json`: Save the scan report to a file named `bandit-report.json`. This command runs the Bandit tool to identify security issues in Python code.

#### Step 5: Upload Bandit Scan Artifact
```yaml
- name: Upload Bandit Scan Artifact
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: bandit-findings
    path: bandit-report.json
```
- **Explanation:**
  - `name: Upload Bandit Scan Artifact`: Descriptive name for the step.
  - `uses: actions/upload-artifact@v4`: Specifies the version of the `upload-artifact` action to use.
  - `if: always()`: Ensures that the artifact is uploaded even if previous steps fail.
  - `with` section:
    - `name: bandit-findings`: Name of the artifact. It will be used to identify the uploaded artifact.
    - `path: bandit-report.json`: Path to the file or directory to be uploaded. In this case, it's the Bandit scan report (`bandit-report.json`).

This step uses the `upload-artifact` action to upload the Bandit scan report as a build artifact named `bandit-findings`. The artifact can be later accessed and used for analysis or reporting purposes. The `if: always()` ensures that the artifact is uploaded regardless of the success or failure of previous steps.

These five steps together define the `sast_scan` job, providing a comprehensive overview of the Bandit scan workflow.



### Job2: `image_scan`

```yaml
image_scan:
  name: Build Image and Run Image Scan
  runs-on: ubuntu-latest

  steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker
      uses: docker-practice/actions-setup-docker@v1
      with:
        docker_version: "20.10.7"

    - name: Build Docker Image
      run: docker build -f Dockerfile -t myapp:latest ./Pygoat

    - name: Docker Scout Scan
      uses: docker/scout-action@v1.3.0
      with:
        dockerhub-user: ${{ secrets.REPO_USER }}
        dockerhub-password: ${{ secrets.REPO_PWD }}
        command: quickview,cves
        only-severities: critical,high
        sarif-file: scout-report.sarif

    - name: Upload Docker Scout Scan Artifact
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: docker-scout-findings
        path: scout-report.sarif
```
Let's break down the `image_scan` job into individual steps:

#### Step 1: Checkout code
```yaml
- name: Checkout code
  uses: actions/checkout@v3
```
**Explanation:**
- **Name**: Descriptive name for the step.
- **Uses**: Utilizes the `actions/checkout` action to fetch the repository code.

#### Step 2: Set up Docker
```yaml
- name: Set up Docker
  uses: docker-practice/actions-setup-docker@master
  with:
    docker_version: "20.10.7"
```
**Explanation:**
- **Name**: Descriptive name for the step.
- **Uses**: Utilizes the `docker-practice/actions-setup-docker` action to set up Docker with version 20.10.7.

#### Step 3: Build Docker Image
```yaml
- name: Build Docker Image
  run: docker build -f Dockerfile -t myapp:latest ./Pygoat
```
**Explanation:**
- **Name**: Descriptive name for the step.
- **Run**: Executes a shell command to build a Docker image using the provided Dockerfile and tags it as `myapp:latest`.

#### Step 4: Docker Scout Scan
```yaml
- name: Docker Scout Scan
  uses: docker/scout-action@v1.3.0
  with:
    dockerhub-user: ${{ secrets.REPO_USER }}
    dockerhub-password: ${{ secrets.REPO_PWD }}
    command: quickview,cves
    only-severities: critical,high
    sarif-file: scout-report.sarif
```
**Explanation:**
- **Name**: Descriptive name for the step.
- **Uses**: Utilizes the `docker/scout-action` action to perform security scans on the Docker image.
- **With**: Provides configuration options such as Docker Hub credentials, scan command, severity levels, and output SARIF file.

#### Step 5: Upload Docker Scout Scan Artifact
```yaml
- name: Upload Docker Scout Scan Artifact
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: docker-scout-findings
    path: scout-report.sarif
```
**Explanation:**
- **Name**: Descriptive name for the step.
- **Uses**: Utilizes the `actions/upload-artifact` action to upload the Docker Scout scan report as an artifact.
- **If**: Specifies that this step should always be executed, even if previous steps fail.
- **With**: Specifies the artifact name (`docker-scout-findings`) and the path of the SARIF file to be uploaded.

These steps collectively define the `image_scan` job, providing a clear sequence of actions to build a Docker image, perform security scans, and upload the findings as an artifact.

## Acknowledgment 

Special thanks to **Nana Janashia** for her valuable guidance and teachings. You can find her on [Techworld with Nana](https://www.youtube.com/c/techworldwithnana).



