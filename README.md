# Technical Documentation: Local AI-Powered Job Curator

## 1. System Overview
The Local AI-Powered Job Curator is an automated, containerized pipeline designed to scrape job listings from major boards and RSS feeds, evaluate them against a user's resume using a localized Large Language Model (LLM), and deliver high-matching opportunities via a daily email summary. 

### Core Components
*   **Ingestion Engine:** Utilizes `python-jobspy` to bypass anti-bot protections on major platforms (LinkedIn, Indeed, Glassdoor) and `feedparser` for standard RSS job boards.
*   **AI Evaluator:** Interfaces with a locally hosted Ollama instance to privately assess job descriptions against the user's resume without sending PII to external APIs.
*   **Notification System:** Leverages standard SMTP protocols to format and dispatch evaluation results.
*   **Containerized Runtime:** Managed via Docker Compose, utilizing volume mounts for dynamic updates to resumes and search criteria without requiring container rebuilds.

---

## 2. Infrastructure & Prerequisites 
Before deploying the application, the external host environment must be properly configured.

### A. Local LLM Configuration (Ollama)
The system relies on a pre-existing Ollama instance running on the host machine.
1. Install Ollama on the host system.
2. Pull the desired evaluation model (e.g., Llama 3) via the host terminal:
   ```bash
   ollama run llama3
   ```
3. Ensure Ollama is binding to a port accessible by Docker (default is `11434`). The application uses `host.docker.internal` to route traffic from the container to the host's Ollama service.

### B. Email SMTP Configuration (Gmail Example)
To allow the script to send emails automatically without triggering security blocks:
1. Navigate to your Google Account Security settings.
2. Ensure **2-Step Verification** is enabled.
3. Search for **App Passwords** and generate a new password designated for this application.
4. Save the 16-character password; it will be injected into the container environment variables.

### C. Resume Preparation
The system requires a plaintext version of your resume to pass into the LLM prompt.
1. Extract text from your existing PDF. You can accomplish this by opening the PDF in Google Docs or Microsoft Word and utilizing the **Save As -> Plain Text (.txt)** function.
2. Save the resulting file exactly as `resume.txt` in your project root directory.

---

## 3. Project File Structure
Create a dedicated directory for the project on your host machine. The final architecture requires the following six files:

### 1. `search_terms.txt`
A dynamic text file containing the job titles or keywords you are targeting. Place one term per line.
```text
Input your job search
descriptions here
and update as needed
```

### 2. `requirements.txt`
Defines the Python dependencies required for the container.
```text
requests
schedule
feedparser
python-jobspy
pandas
```

### 3. `Dockerfile`
Instructions for building the application image. The `-u` flag ensures Python outputs logs unbuffered to the Docker console.
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "-u", "main.py"]
```

### 4. `docker-compose.yml`
Orchestrates the container, maps host environment variables, and mounts the dynamic text files. Update the email and RSS variables accordingly.
---

### 5. `main.py`
The core application logic that handles scheduling, scraping, evaluation, and mailing.
---

## 4. Deployment and Operations

### Build and Run
From the root directory containing all the files above, execute the following command to build the image and start the container in detached mode:
```bash
docker-compose up -d --build
```

### Monitoring Logs
To monitor the system's execution, verify successful scraping, and watch the Ollama evaluation progress:
```bash
docker logs -f job_curator
```

### Updating Runtime Variables
Because `resume.txt` and `search_terms.txt` are mounted as volumes, you can edit these files directly on your host machine at any time. The script reads these files fresh on every execution, so the changes will automatically take effect on the next scheduled run without needing to restart or rebuild the container.
