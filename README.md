# visa_rescheduler

The `visa_rescheduler` is a bot that helps reschedule US visa appointments on usvisa-info.com. It continuously checks for earlier appointment slots and can notify you (and optionally attempt a booking) so you can move your appointment into a desired time window.

IMPORTANT: Use responsibly. Automating interactions with government services can violate Terms of Service and may lead to account suspension or legal issues. This tool is provided "as-is" — read the Disclaimer and Safety section below.

---

## Table of contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Attention / Supported embassies](#attention--supported-embassies)
- [Finding a facility id (how to add an embassy)](#finding-a-facility-id-how-to-add-an-embassy)
- [Initial setup / Installation](#initial-setup--installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Notifications](#notifications)
- [esender.php (optional website email pusher)](#esenderphp-optional-website-email-pusher)
- [Troubleshooting & tips](#troubleshooting--tips)
- [TODO](#todo)
- [Contributing](#contributing)
- [Security, privacy & legal](#security-privacy--legal)
- [Acknowledgements](#acknowledgements)
- [License](#license)

---

## Features

- Periodically checks usvisa-info.com for earlier appointment slots.
- Filters and alerts when earlier dates within your desired range become available.
- Multiple notification options: Pushover, SendGrid (email), or the included `esender.php`.
- Extensible — embassy list and facility IDs are centralized in `embassy.py`.
- Simple command-line usage; easy to run on local machine or VPS.

---

## Prerequisites

- You must already have an existing US visa appointment scheduled.
- Python 3.x (tested with Python 3.8+)
- Google Chrome (for Selenium browser automation)
- Optional: API token for Pushover and/or SendGrid (for notifications)
- Optional: A website to host `esender.php` if you prefer pushing emails from your site

Python packages (versions known to work — recommended):
```
pip install requests==2.27.1
pip install selenium==4.2.0
pip install webdriver-manager==3.7.0
pip install sendgrid==6.9.7
```

On Windows, a bundled `.bat` may install the dependencies automatically.

---

## Attention / Supported embassies

- Many embassies/consulates are supported, but some are not. See the `embassy.py` file for the current supported list.
- If your embassy is unsupported you can add it manually by finding the facility id (instructions below).

---

## Finding a facility id (how to add an embassy)

To add a new embassy (English-language flow):

1. Open your booking page in Google Chrome and find the location (embassy/consulate) selector.
2. Right‑click the selector and choose "Inspect".
3. In the Elements pane you should see a `<select>` element; look for `value="..."` attributes and `data-facility-id` or similar — these are the facility ids you need to add to `embassy.py`.
4. There may be several facility ids for different locations; add all relevant ids.

Illustration:
![Finding Facility id](https://github.com/Soroosh-N/us_visa_scheduler/blob/main/_img.png?raw=true)

---

## Initial setup / Installation

1. Install Google Chrome: https://www.google.com/chrome/
2. Install Python 3.x: https://www.python.org/downloads/
3. Install Python dependencies (see Prerequisites above).
4. Clone this repository and change into it:
   ```
   git clone <repo-url>
   cd <repo-directory>
   ```

---

## Configuration

1. Copy the example config and edit it:
   ```
   cp config.ini.example config.ini
   ```
2. Open `config.ini` and set:
   - Your login credentials for the appointment site (if required by the script)
   - Target embassy / facility id (from `embassy.py` or the inspector method above)
   - Desired earliest acceptable date (ISO format: `YYYY-MM-DD`)
   - Poll interval (how often the script checks for new slots)
   - Notification settings (Pushover, SendGrid, etc.)

Notes:
- Keep `config.ini` out of version control (add to `.gitignore`) because it contains sensitive data.
- If you prefer environment variables, adapt the script to read them instead of `config.ini`.

---

## Usage

Run the main script:
```
python3 visa.py
```

Modes:
- Single-check mode: runs one availability check and exits (useful for cron jobs).
- Polling mode: runs continuously and checks every `POLL_INTERVAL` seconds as set in `config.ini`.

Typical console output:
```
[2025-12-28T12:00:00Z] Checking availability for New Delhi (B2)...
[2025-12-28T12:00:02Z] Found earlier slot: 2026-02-15 — notifying via email and Pushover.
```

Recommended:
- Test first with dry-run or with notifications disabled to ensure the script can log in and parse pages correctly.
- Use a low-frequency polling interval at first to avoid being rate-limited or blocked (e.g., 300 seconds).

---

## Notifications

Supported options:
- Pushover: excellent mobile push notifications (requires API token/user key).
- SendGrid: for email notifications (requires SendGrid API key).
- esender.php: included simple PHP email pusher if you host a small site and prefer server-side email sending.

Configure notification providers in `config.ini`:
- PUSHHOVER_TOKEN, PUSHHOVER_USER
- SENDGRID_API_KEY, NOTIFY_EMAIL
- ESENDER_URL (URL to your hosted `esender.php`)

---

## esender.php (optional website email pusher)

- If you prefer to send emails from a hosted website, the repository includes `esender.php`.
- Deploy `esender.php` to a secure server and set `ESENDER_URL` in `config.ini`.
- Make sure the endpoint is protected and does not accept arbitrary requests from the public.

---

## Troubleshooting & tips

- "Branch/main not found" or other Git errors — check your git remote and default branch name.
- Selenium errors: ensure your Chrome version is compatible with the WebDriver. `webdriver-manager` usually helps manage driver versions.
- If the appointment site changes its layout, XPath/CSS selectors in the script may no longer work — update the parsing logic.
- Rate limits and bans: add jitter/randomized delays and use reasonable polling intervals. Avoid hammering the site.
- Use a dedicated low-risk account for testing.

---

## TODO

- Optimize timing & backoff strategy to reduce chance of blocking.
- Add GUI (PyQt-based) for easier configuration and status monitoring.
- Multi-account support (round-robin / scheduled switching between accounts).
- Add an audible alert for events.
- Expand the embassy list with more facility ids.

---

## Contributing

Contributions are welcome. Recommended workflow:

1. Fork the repo
2. Create a feature branch:
   ```
   git checkout -b feat/your-feature
   ```
3. Add tests and update documentation
4. Open a Pull Request with a clear description of changes

When submitting changes for embassies, include:
- The facility id(s)
- The country/city and any notes about language-specific flows

---

## Security, privacy & legal

- Never commit credentials or personal data to the repository.
- Use environment variables or secrets management in production.
- Be mindful of Terms of Service on usvisa-info.com — automated booking or form submissions may be prohibited.
- The author(s) are not responsible for consequences arising from misuse.

---

## Acknowledgements

Thanks to everyone who contributed to this project and to the maintainers of libraries used (Requests, Selenium, webdriver-manager, SendGrid library).

---

## License

Choose an appropriate license for your project (e.g., MIT). If you want, I can add a LICENSE file with the text of the license you prefer.

---

If you'd like, I can:
- Tailor the README further to match your repo structure and actual config fields (I can open and read `config.ini.example` and `visa.py` to make the README exact).
- Add examples of `config.ini` sections and sample command-line flags.
- Create a small checklist to help run this safely on a VPS or scheduled job (systemd / cron example).
