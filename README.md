# Smart Intrusion Detection System (Smart IDS)

#### SQL Injection Detection System

#### Generative AI-Powered Intrusion Detection System

**Project Type:** Cybersecurity Web Application | **Framework:** Streamlit | **Model:** Qwen2.5 via Ollama

\---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Key Features](#key-features)
3. [Prerequisites](#prerequisites)
4. [Repository Structure](#repository-structure)
5. [Phase 1 — Environment Setup](#phase-1--environment-setup)
6. [Phase 2 — Ollama Model Setup](#phase-2--ollama-model-setup)
7. [Phase 3 — Running the Application](#phase-3--running-the-application)
8. [Phase 4 — Using the Smart IDS Dashboard](#phase-4--using-the-smart-ids-dashboard)
9. [Detection Logic](#detection-logic)
10. [Security Logging](#security-logging)
11. [Troubleshooting](#troubleshooting)
12. [Limitations and Future Improvements](#limitations-and-future-improvements)

\---

## Project Overview

This project builds a **Smart Intrusion Detection System (Smart IDS)** for detecting **SQL Injection (SQLi)** attempts in web request inputs. The system combines traditional rule-based detection with a local Large Language Model (LLM) running through **Ollama**.

The application provides a simple Streamlit interface where users can test login inputs, manually analyze suspicious HTTP payloads, and review security alerts through an admin dashboard.

The system uses a hybrid detection approach:

* **Regex-based rules** to quickly identify common SQL Injection patterns
* **Local LLM analysis** using `qwen2.5:7b-instruct` through Ollama
* **RAG-lite reference notes** to provide the LLM with defensive SQL Injection knowledge
* **Security event logging** into a CSV file
* **Admin dashboard** for reviewing blocked or challenged requests
* **AI assistant** to explain alerts and recommend mitigation steps

\---

## Key Features

|Feature|Description|
|-|-|
|Login Simulation|Simulates a login form and analyzes username/password input before allowing the request|
|Raw Request Analyzer|Allows manual testing of arbitrary HTTP request payloads|
|Rule-Based SQLi Detection|Uses regex rules to detect SQL comments, tautologies, UNION SELECT, and dangerous SQL keywords|
|Local LLM Analysis|Sends suspicious input to a locally running Ollama model for deeper analysis|
|RAG-lite Support|Uses SQL Injection defensive notes as additional context for the LLM|
|Decision Engine|Classifies requests as `ALLOW`, `CHALLENGE`, or `BLOCK`|
|Admin Dashboard|Shows total events, blocked events, challenged events, and recent alerts|
|AI Alert Explanation|Lets the admin ask the LLM to explain an alert and suggest mitigations|
|CSV Logging|Stores processed security events in `security\\\\\\\_events.csv`|

\---

## Prerequisites

### Tools Required

|Tool|Version|Purpose|
|-|-|-|
|Python|3.10+ recommended|Run the Streamlit application|
|pip|Latest recommended|Install Python dependencies|
|Git|2.x|Clone or manage the repository|
|Ollama|Latest|Run the local LLM model|
|Streamlit|From requirements.txt|Build the web dashboard|

### Python Libraries

The project requires the following main Python packages:

```text
streamlit
pandas
requests
watchdog
```

Install them using:

```bash
pip install -r requirements.txt
```

### Ollama Model Required

The application is configured to use:

```text
qwen2.5:7b-instruct
```

A lighter alternative can be used on machines with lower resources:

```text
qwen2.5:3b-instruct
```

\---

## Repository Structure

```text
Smart-Intrusion-Detection-System-Smart-IDS-/
├── README.md                 # Main project documentation
├── app.py                    # Main Streamlit application and IDS detection pipeline
├── requirements.txt          # Python dependencies required to run the project
├── .gitignore                # Files and folders ignored by Git
├── security\\\\\\\_events.csv       # Generated automatically after requests are analyzed
│
└── refs/
    └── sqli\\\\\\\_notes.txt        # SQL Injection defensive notes used for RAG-lite context
```

> Note: `security\\\\\\\_events.csv` is created automatically when the system starts analyzing requests.

\---

## Phase 1 — Environment Setup

### 1.1 Clone the Repository

```bash
git clone <your-repository-url>
cd Smart-Intrusion-Detection-System-Smart-IDS-
```

### 1.2 Create a Virtual Environment

#### Windows PowerShell

```powershell
python -m venv .venv
.\\\\\\\\.venv\\\\\\\\Scripts\\\\\\\\Activate.ps1
```

#### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 1.3 Install Dependencies

```bash
pip install -r requirements.txt
```

\---

## Phase 2 — Ollama Model Setup

### 2.1 Install Ollama

Download and install Ollama from the official website:

```text
https://ollama.com
```

### 2.2 Pull the Required Model

```bash
ollama pull qwen2.5:7b-instruct
```

If your device has limited RAM or CPU resources, use the smaller model:

```bash
ollama pull qwen2.5:3b-instruct
```

If you use the smaller model, update this line in `app.py`:

```python
MODEL\\\\\\\_NAME = "qwen2.5:3b-instruct"
```

### 2.3 Verify Ollama is Running

```bash
ollama list
```

You can also test the API:

```bash
curl http://localhost:11434/api/tags
```

The application expects Ollama to run locally on:

```text
http://localhost:11434
```

\---

## Phase 3 — Running the Application

Start the Streamlit application:

```bash
streamlit run app.py
```

Streamlit will open the application in your browser, usually at:

```text
http://localhost:8501
```

\---

## Phase 4 — Using the Smart IDS Dashboard

The application contains three main tabs.

### 4.1 Login Demo

This tab simulates a login endpoint. The user enters a username and password, and the IDS analyzes the combined payload before allowing the login attempt.

Example malicious input:

```text
' OR '1'='1 --
```

Possible decisions:

|Decision|Meaning|
|-|-|
|ALLOW|The request appears normal|
|CHALLENGE|The request is suspicious but not confidently malicious|
|BLOCK|The request is highly likely to be malicious|

\---

### 4.2 Raw Request Analyzer

This tab allows manual testing of any HTTP request input or payload.

Example payloads:

```text
id=1 UNION SELECT username,password FROM users
```

```text
admin' OR '1'='1 --
```

```text
name=Ahmed\\\\\\\&comment=Hello this is a normal message
```

The user can optionally enable:

```text
Use reference notes (RAG-lite)
```

This adds SQL Injection defensive notes into the LLM prompt to improve contextual analysis.

\---

### 4.3 Admin Dashboard

This tab displays security monitoring information, including:

* Total analyzed events
* Number of blocked requests
* Number of challenged requests
* Recent alerts
* Full event log
* AI assistant for explaining selected alerts

The AI assistant can answer questions such as:

```text
Explain this alert and suggest mitigations.
```

\---

## Detection Logic

The Smart IDS uses a hybrid pipeline.

### Step 1 — Rule-Based Detection

The system first checks the input using fast regex and keyword rules. It looks for common SQL Injection indicators such as:

|Pattern Type|Example|
|-|-|
|Boolean-based injection|`' OR '1'='1`|
|SQL comments|`--`, `/\\\\\\\* \\\\\\\*/`|
|UNION-based injection|`UNION SELECT`|
|Dangerous keywords|`DROP TABLE`, `INSERT INTO`, `WAITFOR DELAY`|
|Tautologies|`1=1`, `2=2`|

Each matched pattern increases the risk score and adds detection tags.

### Step 2 — LLM Analysis

After rule checking, the request is sent to the local Ollama model. The model returns structured JSON containing:

```json
{
  "label": "malicious",
  "confidence": 0.92,
  "risk\\\\\\\_level": "high",
  "reasons": \\\\\\\["Possible UNION-based SQL Injection"],
  "recommended\\\\\\\_mitigations": \\\\\\\["Use parameterized queries"],
  "tags": \\\\\\\["union\\\\\\\_select"]
}
```

### Step 3 — Decision Policy

The application converts the model output into a final decision:

|Condition|Decision|
|-|-|
|label = normal|ALLOW|
|label = malicious and confidence >= 0.80|BLOCK|
|label = malicious and confidence < 0.80|CHALLENGE|
|unexpected output|CHALLENGE|

\---

## Security Logging

Every analyzed request is logged to:

```text
security\\\\\\\_events.csv
```

The log includes:

|Field|Description|
|-|-|
|timestamp\_utc|Time of the analyzed request|
|client\_id|Client identifier or IP address|
|endpoint|Simulated endpoint|
|input\_preview|Short preview of the request input|
|label|Model classification|
|confidence|Model confidence score|
|risk\_level|Low, medium, high, or unknown|
|decision|ALLOW, CHALLENGE, or BLOCK|
|tags|Detection tags such as `union\\\\\\\_select` or `sql\\\\\\\_comment`|

\---

## Troubleshooting

### Ollama is Offline

If the sidebar shows:

```text
Ollama Offline
```

Start or restart Ollama:

```bash
ollama serve
```

Then test:

```bash
curl http://localhost:11434/api/tags
```

\---

### Model Not Found

If you see an error that the model does not exist, pull it again:

```bash
ollama pull qwen2.5:7b-instruct
```

Or update `MODEL\\\\\\\_NAME` in `app.py` to match an installed model:

```bash
ollama list
```

\---

### Streamlit Command Not Found

Install the dependencies again:

```bash
pip install -r requirements.txt
```

Or run Streamlit with Python:

```bash
python -m streamlit run app.py
```

\---

### Reference Notes Not Loading

The app expects the reference file at:

```text
refs/sqli\\\\\\\_notes.txt
```

If your file is currently in the root folder, create a `refs` folder and move it:

```bash
mkdir refs
mv sqli\\\\\\\_notes.txt refs/sqli\\\\\\\_notes.txt
```

On Windows PowerShell:

```powershell
mkdir refs
move sqli\\\\\\\_notes.txt refs\\\\\\\\sqli\\\\\\\_notes.txt
```

\---

## Limitations and Future Improvements

### Current Limitations

* The application is a prototype and is not a production-ready WAF.
* LLM responses may sometimes be inconsistent or fail to return valid JSON.
* Detection accuracy depends on the local model quality and available machine resources.
* The current logging system uses CSV instead of a database.
* The login page is a simulation and does not connect to a real authentication backend.

### Future Improvements

* Add a real backend API using FastAPI or Flask
* Store logs in SQLite or PostgreSQL
* Add authentication for the admin dashboard
* Add charts for attack trends and detection categories
* Add Docker support for easier deployment
* Improve prompt hardening against prompt injection
* Add more SQLi test cases and unit tests
* Integrate with a real WAF or SIEM system

\---

## Educational Purpose

This project is designed for educational and research purposes. It demonstrates how traditional IDS rules and local generative AI can be combined to detect and explain SQL Injection attempts.

It should not be used as the only security control in a production environment. Production systems should still use secure coding practices such as parameterized queries, input validation, least privilege database accounts, safe error handling, and Web Application Firewalls.

\---

## Author

**Naira Albattra**

**Email: `naira\\\_albattra@outlook.com`**

