# visa_rescheduler — Kenya (U.S. Embassy Nairobi)

This fork/edition of `visa_rescheduler` is tailored to help monitor and (optionally) reschedule U.S. visa appointment slots specifically for the U.S. Embassy in Nairobi, Kenya (usvisa-info.com). Use this only to be notified of earlier slots — avoid automated bookings unless you fully understand the legal and account risks.

Why this tailored README?  
You mentioned you needed an earlier appointment at the Kenyan embassy because your university classes start before your currently scheduled appointment. This README gives Kenya-specific guidance, configuration examples, and practical tips so you can target Nairobi appointments efficiently and safely.

---

## Quick overview (Kenya-specific)

- Target: U.S. Embassy — Nairobi (most non-immigrant visa scheduling in Kenya happens here).
- Timezone: East Africa Time (EAT, UTC+3). When you set/compare dates/times, convert site times to EAT.
- Language: English flow on the appointment site in Kenya; the generic `embassy.py` approach should work but you must supply the correct facility id(s) for Nairobi.
- Recommended starting poll interval: 300–600 seconds (5–10 minutes) with jitter. Do not poll too frequently — this reduces the chance of temporary blocks.

IMPORTANT: Do not supply credentials into any public place. Keep `config.ini` out of source control.

---

## How to find the correct facility id for Nairobi

You must add the Nairobi facility id(s) to `embassy.py` or your `config.ini`. Do NOT guess numeric IDs — find them as follows:

1. Log in to your usvisa-info.com account and go to the scheduling page where you choose the location (city/embassy/consulate).
2. In Google Chrome, right‑click the location selector and choose "Inspect" (DevTools).
3. Look for a `<select>` element or other location element. Facility ids often appear as:
   - `value="12345"` on `<option>` elements, or
   - `data-facility-id="12345"` attributes.
4. Copy the numeric id(s) for "Nairobi", "Nairobi - US Embassy", or any related option you use, and add them to `embassy.py` (or put them in `config.ini` as `facility_id`).
5. If you see multiple facility ids (different appointment centers), add all that apply.

Illustration:
![Finding Facility id](https://github.com/Soroosh-N/us_visa_scheduler/blob/main/_img.png?raw=true)

---

## Example config (Kenya-focused)

Below is a recommended `config.ini` snippet tailored for Nairobi. Replace placeholders with your real values.

```
[account]
email = your_email@example.com
password = your_password

[target]
# Replace with the numeric facility id(s) you found via Chrome DevTools
facility_id = 123456   ; <-- INSERT actual numeric ID for Nairobi
facility_name = U.S. Embassy Nairobi
visa_category = B2
earliest_acceptable_date = 2026-08-01  ; <-- set to a date BEFORE your class start date
poll_interval_seconds = 300
random_jitter_seconds = 30
max_consecutive_failures_before_backoff = 5

[notifications]
# Pushover example
pushover_token = your_pushover_api_token
pushover_user = your_pushover_user_key

# SendGrid example (email)
sendgrid_api_key = SG.xxxxx
notify_email = you@yourdomain.tld

# Optional: host esender.php on your server and set URL
esender_url = https://yourdomain.tld/esender.php
```

Notes:
- Set `earliest_acceptable_date` to the latest date that still works for your university start date (e.g., at least a few days before classes begin).
- Use `random_jitter_seconds` to add small randomized delays to each poll — this reduces suspicious patterns.

---

## Running (recommended Kenya workflow)

1. Install prerequisites (Python, Chrome, webdriver-manager, etc.) — see Prerequisites in the main README.
2. Clone repo and prepare config:
   ```
   git clone <repo-url>
   cd <repo-directory>
   cp config.ini.example config.ini
   # edit config.ini with your account, Nairobi facility id, earliest date, and notifications
   ```
3. Test a single run (check mode) to ensure login/parsing works:
   ```
   python3 visa.py --once
   ```
   If your script does not support `--once`, run it and watch logs for a single iteration.
4. When the single check succeeds, run in polling mode:
   ```
   python3 visa.py
   ```
5. Watch notifications. If an earlier slot is found, the script will notify you via configured providers.

---

## Recommended safe settings for Kenya

- poll_interval_seconds: 300–600 (5–10 minutes)
- random_jitter_seconds: 10–60
- Use exponential backoff after 3–5 consecutive failures
- Use dry-run mode for initial tests (no notifications/bookings)
- Avoid running many parallel instances against the same account

---

## Practical tips for getting a slot before university starts

- Set `earliest_acceptable_date` several days before your classes to allow for travel and check-in.
- Check availability around midnight local time and early morning — some slots appear after maintenance or daily refreshes.
- If you have flexibility, target multiple visa categories (if applicable) or different facility ids (if Nairobi has multiple).
- Keep mobile notifications enabled — immediate alerts help you act quickly.
- If you plan to book manually: when notified, immediately log in and confirm the slot — some scripts only notify and not book.

---

## Running as a background service (example systemd unit)

Create `/etc/systemd/system/visa_rescheduler.service` (adapt paths and user):

```
[Unit]
Description=Visa Rescheduler (Nairobi)
After=network.target

[Service]
User=youruser
WorkingDirectory=/home/youruser/visa_rescheduler
ExecStart=/usr/bin/python3 /home/youruser/visa_rescheduler/visa.py
Restart=on-failure
RestartSec=60

[Install]
WantedBy=multi-user.target
```

Then:
```
sudo systemctl daemon-reload
sudo systemctl enable --now visa_rescheduler
```

---

## Troubleshooting (Kenya-specific notes)

- Timezone mismatch: server time may be UTC — convert to EAT (UTC+3) when interpreting dates.
- If login fails: verify credentials manually on usvisa-info.com, clear cookies, and ensure the account doesn’t have CAPTCHA or extra verification.
- If the site layout has changed: update the selectors in `visa.py` and any page parsing functions to match Nairobi flows.
- If you get banned/blocked temporarily: stop polling, increase poll interval, increase jitter, and consider a different IP or wait for the block to lift.
