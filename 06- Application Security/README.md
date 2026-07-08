# 🔐 Application Security (AppSec) - SAST, SCA & DAST

## 📖 Overview
Application Security (AppSec) focuses on identifying and fixing vulnerabilities throughout the Software Development Life Cycle (SDLC). No single security tool is sufficient, which is why organizations combine **SAST**, **SCA**, and **DAST** to build a secure DevSecOps pipeline.

---

# Security Testing Approaches

| Testing | What it Scans | Access Required | Best Stage | Example Tool |
|----------|---------------|----------------|------------|--------------|
| **SAST** | Source Code | Source Code | Development | SonarQube |
| **SCA** | Dependencies | Dependency Files | Build Phase | pip-audit |
| **DAST** | Running Application | No Source Code | Runtime | OWASP ZAP |

> **Remember:** SAST + SCA + DAST together provide better application security coverage.

---

# 1️⃣ Static Application Security Testing (SAST)

Scans the application's **source code** without executing it and detects insecure coding practices early.

### Common Issues Detected
- Hardcoded Secrets
- SQL Injection Patterns
- Command Injection
- Insecure Deserialization (`pickle`)
- Arbitrary Code Execution (`eval()`, `exec()`)
- Unsafe YAML Loading (`yaml.load`)
- Path Traversal (`tarfile.extractall`)
- Insecure SSL (`verify=False`)
- Weak Cryptography (MD5 / SHA1)
- Insecure Temporary Files (`tempfile.mktemp`)
- Debug Mode Enabled

### Why SAST?
- Shift Security Left
- Detect vulnerabilities before deployment
- Faster & cheaper to fix
- Integrates with Pull Requests and CI/CD

**Tool Used:** SonarQube

---

# 2️⃣ Software Composition Analysis (SCA)

Scans third-party libraries and dependencies for publicly known vulnerabilities (CVEs).

### Detects
- Vulnerable packages
- Outdated dependencies
- Known CVEs
- License issues

Example:

```
Flask==1.0
Requests==2.19.1
```

Both contain known vulnerabilities.

**Tool Used:** pip-audit

---

# 3️⃣ Dynamic Application Security Testing (DAST)

Tests a **running application** from an attacker's perspective without accessing the source code.

### Detects
- SQL Injection
- XSS
- Missing Security Headers
- Authentication Issues
- Session Management Issues
- Sensitive Information Disclosure
- Command Injection

**Tool Used:** OWASP ZAP

---

# Lab Workflow

```
Write Code
      │
      ▼
 SAST (SonarQube)
      │
      ▼
 SCA (pip-audit)
      │
      ▼
 Build Application
      │
      ▼
 Deploy Application
      │
      ▼
 DAST (OWASP ZAP)
```

---

# Commands Used

### Run SonarQube

```bash
docker run -d -p 9000:9000 --name sonarqube sonarqube:9.9-community
```

### Run Sonar Scanner

```bash
sonar-scanner \
-Dsonar.host.url=http://localhost:9000 \
-Dsonar.login=<TOKEN>
```

### Run pip-audit

```bash
pip install pip-audit
pip-audit -r requirements.txt
```

### Start Vulnerable Application

```bash
pip install -r requirements.txt
python app.py
```

### Run Juice Shop

```bash
docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop
```

### Run OWASP ZAP Baseline Scan (CLI)

```bash
docker run --rm \
-v "$(pwd):/zap/wrk" \
-t ghcr.io/zaproxy/zaproxy:stable \
zap-baseline.py \
-t http://host.docker.internal:3000 \
-r zap-report.html
```

Generates an HTML report (`zap-report.html`) containing the identified security findings. This mode is commonly used in CI/CD pipelines.

---

### Run OWASP ZAP GUI (WebSwing)

```bash
docker run -u zap \
-p 8080:8080 \
-p 8090:8090 \
-i ghcr.io/zaproxy/zaproxy:stable \
zap-webswing.sh
```

Open the browser:

```
http://localhost:8080/zap
```

> **Note (Windows/Git Bash):** If you experience issues with the WebSwing interface, the native OWASP ZAP Desktop application is generally more reliable for interactive testing. For automated DevSecOps pipelines, prefer the CLI baseline scan.

---

# Comparison

| Vulnerability | SAST | SCA | DAST |
|---------------|:----:|:---:|:----:|
| Hardcoded Secrets | ✅ | ❌ | ❌ |
| Vulnerable Dependencies | ❌ | ✅ | ❌ |
| SQL Injection | ✅ | ❌ | ✅ |
| Command Injection | ✅ | ❌ | ✅ |
| Runtime Misconfigurations | ❌ | ❌ | ✅ |

---

# Best Practices

- Use parameterized SQL queries
- Remove `eval()` and `exec()`
- Store secrets in AWS Secrets Manager / Vault / Environment Variables
- Keep dependencies updated
- Disable Debug Mode in Production
- Integrate SAST, SCA & DAST into CI/CD
- Scan continuously and remediate vulnerabilities early

---

## 🚀 Key Takeaway

> **Secure applications require multiple layers of testing.**
>
> - **SAST** secures your code.
> - **SCA** secures your dependencies.
> - **DAST** secures your running application.
>
> Combining all three creates a strong foundation for building secure applications in a modern DevSecOps pipeline.
