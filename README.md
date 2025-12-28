# visa_rescheduler — Kenya (U.S. Embassy Nairobi)

I've tailored this edition of `visa_rescheduler` to help me monitor and (optionally) reschedule U.S. visa appointment slots specifically for the U.S. Embassy in Nairobi, Kenya (usvisa-info.com). I use this to be notified of earlier slots — I avoid automated bookings unless I fully understand the legal and account risks.

I needed an earlier appointment at the Kenyan embassy because my university classes start before my currently scheduled appointment. This README gives Kenya-specific guidance, configuration examples, practical tips, and my AWS learnings so I can target Nairobi appointments efficiently and safely and show the work in a recruiter-friendly way.

---

## Quick overview (Kenya-specific)

- Target: U.S. Embassy — Nairobi (non-immigrant visa scheduling in Kenya).
- Timezone: East Africa Time (EAT, UTC+3). I convert site times to EAT when I set/compare dates.
- Language: English flow on the appointment site in Kenya; the generic `embassy.py` approach works if I supply the correct facility id(s).
- Recommended starting poll interval: 300–600 seconds (5–10 minutes) with jitter. I do not poll too frequently to reduce the chance of temporary blocks.

IMPORTANT: I never supply credentials in any public place. I keep `config.ini` out of source control.

---

## Features (what I get from this fork)

- Periodic checks of usvisa-info.com for earlier appointment slots.
- Alerts when earlier dates within my desired range become available.
- Notification options: Pushover, SendGrid (email), or the included `esender.php`.
- Extensible embassy list and facility IDs centralized in `embassy.py`.
- Easy command-line usage; can run locally or on a VPS.

---

## Prerequisites

- I already have an existing US visa appointment scheduled.
- Python 3.x (tested with Python 3.8+).
- Google Chrome (for Selenium browser automation).
- Optional: API tokens for Pushover and/or SendGrid for notifications.
- Optional: A website to host `esender.php` if I prefer server-side email sending.

Recommended Python packages (versions I tested):
```
pip install requests==2.27.1
pip install selenium==4.2.0
pip install webdriver-manager==3.7.0
pip install sendgrid==6.9.7
```

On Windows I can use the included `.bat` to install dependencies.

---

## How I find the correct facility id for Nairobi

I must add the Nairobi facility id(s) to `embassy.py` or my `config.ini`. I do NOT guess numeric IDs — I find them like this:

1. Log in to my usvisa-info.com account and go to the scheduling page where I choose the location (city/embassy/consulate).
2. In Google Chrome, I right‑click the location selector and choose "Inspect" (DevTools).
3. In the Elements pane I look for a `<select>` element or related element. Facility ids often appear as:
   - `value="12345"` on `<option>` elements, or
   - `data-facility-id="12345"` attributes.
4. I copy the numeric id(s) for "Nairobi", "Nairobi - US Embassy", or any related option I use, and add them to `embassy.py` or set them in `config.ini` as `facility_id`.
5. If there are multiple facility ids (different appointment centers), I add all that apply.

Illustration:
![Finding Facility id](https://github.com/Soroosh-N/us_visa_scheduler/blob/main/_img.png?raw=true)

---

## Example config (Kenya-focused)

Below is my recommended `config.ini` snippet for Nairobi. I replace placeholders with my real values.

```
[account]
email = my_email@example.com
password = my_password

[target]
# Replace with the numeric facility id(s) you found via Chrome DevTools
facility_id = 123456   ; <-- INSERT actual numeric ID for Nairobi
facility_name = U.S. Embassy Nairobi
visa_category = B2
earliest_acceptable_date = 2026-08-01  ; <-- set to a date before my class start date
poll_interval_seconds = 300
random_jitter_seconds = 30
max_consecutive_failures_before_backoff = 5

[notifications]
# Pushover example
pushover_token = my_pushover_api_token
pushover_user = my_pushover_user_key

# SendGrid example (email)
sendgrid_api_key = SG.xxxxx
notify_email = me@mydomain.tld

# Optional: host esender.php on my server and set URL
esender_url = https://mydomain.tld/esender.php
```

Notes:
- I use ISO date format `YYYY-MM-DD`.
- I keep `config.ini` out of version control and use environment variables or a secrets manager where possible.
- I set `earliest_acceptable_date` to the latest date that still lets me start university on time (with some buffer days).

---

## Installation & initial setup

1. Install Google Chrome: https://www.google.com/chrome/
2. Install Python 3.x: https://www.python.org/downloads/
3. Clone the repository and change directory:
   ```
   git clone <repo-url>
   cd <repo-directory>
   ```
4. Install the required Python packages (see Prerequisites).
5. Copy and edit the example config:
   ```
   cp config.ini.example config.ini
   # edit config.ini with my account credentials, Nairobi facility id, earliest date, and notification settings
   ```

---

## Usage

- Test a single run (check mode) to ensure login and parsing work:
  ```
  python3 visa.py --once
  ```
  If `--once` is unsupported, run and observe a single iteration then stop.

- Run in polling mode:
  ```
  python3 visa.py
  ```

Typical console output:
```
[2025-12-28T12:00:00Z] Checking availability for Nairobi (B2)...
[2025-12-28T12:00:02Z] Found earlier slot: 2026-07-25 — notifying via email and Pushover.
```

Recommended safe settings:
- poll_interval_seconds: 300–600 (5–10 minutes)
- random_jitter_seconds: 10–60
- Use exponential backoff after 3–5 consecutive failures
- Start with dry-run mode or notifications disabled to confirm parsing and login

---

## Notifications

I can configure notifications in `config.ini`:

- Pushover: mobile push notifications (API token + user key).
- SendGrid: email notifications (API key + target email).
- esender.php: send email from a hosted server endpoint (secure the endpoint before use).

---

## Running as a background service (example systemd)

I can run the script as a systemd service (example unit):

```
[Unit]
Description=Visa Rescheduler (Nairobi)
After=network.target

[Service]
User=myuser
WorkingDirectory=/home/myuser/visa_rescheduler
ExecStart=/usr/bin/python3 /home/myuser/visa_rescheduler/visa.py
Restart=on-failure
RestartSec=60

[Install]
WantedBy=multi-user.target
```

Commands:
```
sudo systemctl daemon-reload
sudo systemctl enable --now visa_rescheduler
```

---

## Practical tips for getting a slot before university starts

- Set `earliest_acceptable_date` several days before classes start so I have buffer time for travel and check-in.
- Check availability around midnight local time and early morning — some slots appear after daily refreshes.
- If applicable, consider multiple facility ids or nearby appointment centers.
- Enable immediate mobile/email notifications so I can act quickly when a slot appears.
- When I receive a notification, I log in immediately and confirm the slot manually unless I accept the risks of automated booking.

---

## Troubleshooting (Kenya-specific notes)

- Timezone mismatch: my server might run in UTC — convert to EAT (UTC+3) when interpreting dates.
- Login failures: verify credentials manually on usvisa-info.com, clear cookies, and ensure no CAPTCHA or extra verification is blocking automated login.
- Site layout changes: update selectors in `visa.py` if the appointment site changes.
- If I get blocked or rate-limited: stop polling, increase poll interval and jitter, and wait or change IP if appropriate.

---

## What I learned from running this project on AWS (recruiter-friendly)

Deploying and operating this bot on AWS taught me several practical, recruiter-friendly skills across development, operations and security:

- Containerization & CI/CD
  - I containerized the app (Docker) and pushed images to Amazon ECR.
  - I integrated a CI pipeline (GitHub Actions) to build and push images and run tests automatically.

- Orchestration & Serverless scheduling
  - I ran the service in a managed environment (ECS Fargate) so I could avoid managing OS-level updates and scale safely.
  - For scheduled checks I used EventBridge (CloudWatch Events) to trigger tasks or Lambda-based health checks.

- Secrets & least-privilege access
  - I moved sensitive configuration out of files and into AWS Secrets Manager (or SSM Parameter Store), and granted least-privilege IAM roles to the task to access them.
  - I rotated and audited secrets access to reduce exposure.

- Observability & alerts
  - I centralized logs in CloudWatch Logs, created metrics and dashboards for availability, error rates and notification deliveries.
  - I used CloudWatch Alarms and SNS to alert me on failures or unexpected increases in error rate.

- Reliability & cost-awareness
  - Using Fargate/ECR simplified maintenance and reduced idle cost compared to an always-on EC2 instance.
  - I implemented exponential backoff and jitter in the scraper to avoid service bans and reduce unnecessary runs, saving compute time and cost.

- Security & networking
  - I hardened the environment with proper security group rules, IAM policies and VPC placement where needed.
  - I ensured the `esender.php` endpoint (if used) required authentication and rate limits to avoid abuse.

- Infrastructure as Code & repeatability
  - I modeled the deployment using IaC (CloudFormation or Terraform) so the setup is reproducible, reviewable and version-controlled.
  - This made deployments to staging/production consistent and easy to roll back.

Recruiter-friendly summary: deploying this project gave me hands-on experience with containerization, CI/CD, secrets management, observability (logs/metrics/alerts), secure IAM practices, cost-optimized managed services (ECS Fargate / EventBridge), and making scraping workloads reliable and maintainable in the cloud.

---

## Safety, privacy & legal

- I never commit credentials or personal data to the repo.
- I use environment variables or a secrets manager in production.
- I follow the Terms of Service on usvisa-info.com — automating bookings or bypassing intended restrictions can lead to account suspension or legal consequences.
- This tool is provided "as-is"; I accept responsibility for its use.

---

## Acknowledgements

Thanks to contributors of the original project and the maintainers of Requests, Selenium, webdriver-manager, and the SendGrid library.
