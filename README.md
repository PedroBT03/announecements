# RSSNotify

**RSSNotify** watches your favorite RSS feeds and emails you when new items appear. It runs on GitHub Actions and uses SendGrid for delivery.

---

## Table of Contents

1. [Requirements](#requirements)  
2. [Repository Setup](#repository-setup)  
3. [SendGrid Configuration](#sendgrid-configuration)  
4. [Environment Variables](#environment-variables)  
5. [Customizing Feeds & Subjects](#customizing-feeds--subjects)  
6. [GitHub Actions Workflow](#github-actions-workflow)  
7. [Usage & Testing](#usage--testing)  
8. [Troubleshooting](#troubleshooting)  

---

## Requirements

- **GitHub Account** & repo to host the automation  
- **SendGrid Account** with a verified sender and an API key  
- **Python** (3.6+)  
- Python packages:  
  - `feedparser`  
  - `sendgrid`  

---

## Repository Setup

1. **Clone** or **create** a new repo.  
2. Add these files to the project root:  
   - `rss_notifier.py`  
   - `seen_posts.txt`  

---

## SendGrid Configuration

1. Sign in to SendGrid (or create an account).  
2. In the dashboard, go to **Settings > API Keys** → **Create API Key** → give it a name & Full Access → **Create & View**.  
3. Under **Sender Authentication**, verify the email address you’ll use as the sender.

---

## Environment Variables

Store sensitive values in **GitHub Secrets**:

| Secret Name         | Description                                  |
|---------------------|----------------------------------------------|
| `SENDGRID_API_KEY`  | Your SendGrid API Key                        |
| `SENDER_EMAIL`      | Verified “from” email address                |
| `RECEIVER_EMAIL`    | Destination email for notifications          |

> **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

---

## Customizing Feeds & Subjects

### Feed List

Open `rss_notifier.py` and locate:

```python
RSS_FEED_URLS = [
    "https://example.com/course1/rss",
    "https://example.com/course2/rss"
]
```

Add, remove, or replace URLs as needed.

---

### Email Subject Logic

In the same script, adjust `get_course_name()` to map feed URLs to meaningful subjects:

```python
def get_course_name(feed_url):
    if "CCU" in feed_url:
        return "IST – CCU Updates"
    elif "SIRS" in feed_url:
        return "IST – SIRS News"
    else:
        return "RSSNotify – New Item"
```

## GitHub Actions Workflow

Create `.github/workflows/rssnotify.yml`:

```yaml
name: RSSNotify

on:
  schedule:
    - cron: "*/30 * * * *"   # every 30 minutes (modify as needed)
  workflow_dispatch:         # manual trigger

permissions:
  contents: write            # allow pushing updates

jobs:
  run-notifier:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.x"

      - name: Install dependencies
        run: pip install feedparser sendgrid

      - name: Execute RSS notifier
        env:
          SENDGRID_API_KEY: ${{ secrets.SENDGRID_API_KEY }}
          SENDER_EMAIL: ${{ secrets.SENDER_EMAIL }}
          RECEIVER_EMAIL: ${{ secrets.RECEIVER_EMAIL }}
        run: python rss_notifier.py

      - name: Upload state file
        uses: actions/upload-artifact@v3
        with:
          name: seen-posts
          path: seen_posts.txt

      - name: Commit updated seen_posts.txt
        run: |
          git diff --quiet seen_posts.txt || (
            git config user.name "github-actions[bot]" &&
            git config user.email "github-actions[bot]@users.noreply.github.com" &&
            git add seen_posts.txt &&
            git commit -m "Update seen_posts.txt" &&
            git push
          )
```

## Usage & Testing

1. Push your changes to GitHub.
2. Visit the **Actions** tab → select **RSSNotify** → click **Run workflow**.
3. Check the recipient inbox for a test email.
4. Monitor the logs under **Actions** to verify feed parsing and email dispatch.

## Troubleshooting

**No email received?**
- Confirm `SENDER_EMAIL` is verified in SendGrid.
- Check GitHub Actions logs for errors.

**Malformed feed URL?**
- Paste the URL in a browser or RSS reader to verify its validity.

**Secrets not found?**
- Ensure spelling matches exactly under **Settings > Secrets**.
