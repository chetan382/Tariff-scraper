# Flexport Tariff Scraper - GitHub Actions Setup

## 📁 File Structure

Your repository should have this structure:

```
your-repo/
├── .github/
│   └── workflows/
│       └── scrape-tariffs.yml
├── scraper.py
├── requirements.txt
└── README.md
```

## 🚀 Setup Instructions

### 1. Create the Repository Structure

1. Create a new GitHub repository (or use an existing one)
2. Create the `.github/workflows/` directory
3. Add all three files:
   - `scraper.py` (the main scraper script)
   - `.github/workflows/scrape-tariffs.yml` (the workflow file)
   - `requirements.txt` (Python dependencies)

### 2. Get a GitHub Personal Access Token

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Give it a name like "Tariff Scraper API"
4. Select scopes:
   - ✅ `repo` (Full control of private repositories)
   - ✅ `workflow` (Update GitHub Action workflows)
5. Click "Generate token"
6. **Save this token securely** - you'll need it for API calls

## 🔌 How to Trigger via API

### Using cURL

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer YOUR_GITHUB_TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/YOUR_USERNAME/YOUR_REPO/actions/workflows/scrape-tariffs.yml/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "hts_codes": "[\"2940.00.6000\",\"2936.27.0000\",\"2922.50.5000\"]"
    }
  }'
```

### Using Python

```python
import requests
import json

GITHUB_TOKEN = "your_github_token_here"
REPO_OWNER = "your_username"
REPO_NAME = "your_repo_name"

def trigger_scraper(hts_codes: list):
    url = f"https://api.github.com/repos/{REPO_OWNER}/{REPO_NAME}/actions/workflows/scrape-tariffs.yml/dispatches"
    
    headers = {
        "Accept": "application/vnd.github+json",
        "Authorization": f"Bearer {GITHUB_TOKEN}",
        "X-GitHub-Api-Version": "2022-11-28"
    }
    
    data = {
        "ref": "main",  # or your default branch
        "inputs": {
            "hts_codes": json.dumps(hts_codes)
        }
    }
    
    response = requests.post(url, headers=headers, json=data)
    
    if response.status_code == 204:
        print("✓ Workflow triggered successfully!")
    else:
        print(f"✗ Error: {response.status_code}")
        print(response.text)

# Example usage
hts_codes = ["2940.00.6000", "2936.27.0000", "2922.50.5000"]
trigger_scraper(hts_codes)
```

### Using JavaScript/Node.js

```javascript
const axios = require('axios');

const GITHUB_TOKEN = 'your_github_token_here';
const REPO_OWNER = 'your_username';
const REPO_NAME = 'your_repo_name';

async function triggerScraper(htsCodes) {
  const url = `https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/actions/workflows/scrape-tariffs.yml/dispatches`;
  
  try {
    await axios.post(url, {
      ref: 'main',
      inputs: {
        hts_codes: JSON.stringify(htsCodes)
      }
    }, {
      headers: {
        'Accept': 'application/vnd.github+json',
        'Authorization': `Bearer ${GITHUB_TOKEN}`,
        'X-GitHub-Api-Version': '2022-11-28'
      }
    });
    
    console.log('✓ Workflow triggered successfully!');
  } catch (error) {
    console.error('✗ Error:', error.response?.data || error.message);
  }
}

// Example usage
const htsCodes = ['2940.00.6000', '2936.27.0000', '2922.50.5000'];
triggerScraper(htsCodes);
```

## 📊 Webhook Response Format

The scraper will send results to your n8n webhook in this format:

```json
{
  "status": "completed",
  "total_codes": 3,
  "successful": 3,
  "failed": 0,
  "results": [
    {
      "hts_code": "2940.00.6000",
      "success": true,
      "total_duty_rate": 6.5,
      "base_cost": 10000.00,
      "total_duties": 650.00,
      "landed_cost": 10650.00,
      "tariff_breakdown": [
        {
          "hts_code": "2940.00.6000",
          "description": "Base rate",
          "rate": "6.5%",
          "rate_value": 6.5,
          "amount": 650.00
        }
      ],
      "verification": {
        "sum_matches": true,
        "calculated_sum": 6.5
      }
    }
  ]
}
```

## 🔍 Monitoring the Workflow

1. Go to your GitHub repository
2. Click on the "Actions" tab
3. You'll see the workflow runs listed there
4. Click on a run to see detailed logs

## 🎯 Customizing the Webhook URL

You can override the default webhook URL when triggering:

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer YOUR_GITHUB_TOKEN" \
  https://api.github.com/repos/YOUR_USERNAME/YOUR_REPO/actions/workflows/scrape-tariffs.yml/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "hts_codes": "[\"2940.00.6000\"]",
      "webhook_url": "https://your-custom-webhook.com/endpoint"
    }
  }'
```

## 📥 Accessing Results

Results are also uploaded as GitHub Actions artifacts and stored for 7 days:

1. Go to the workflow run in GitHub Actions
2. Scroll down to the "Artifacts" section
3. Download `tariff-results` to get the `results.json` file

## ⚠️ Important Notes

- The workflow runs on GitHub's servers (free for public repos, limited minutes for private repos)
- Each workflow run is completely isolated
- Results are automatically sent to your n8n webhook
- The scraper runs in headless mode for compatibility with GitHub Actions
- Rate limits apply: Be respectful to the Flexport website

## 🐛 Troubleshooting

**Workflow not triggering?**
- Check your GitHub token has the right permissions
- Verify the repository name and owner are correct
- Ensure the workflow file is on the `main` branch (or specify your branch in the API call)

**Scraper failing?**
- Check the Actions logs for detailed error messages
- Verify the HTS codes are valid
- Ensure the webhook URL is accessible

**Webhook not receiving data?**
- Verify the webhook URL is correct and accessible
- Check the workflow logs to see if the webhook request succeeded
- Test the webhook URL with a tool like Postman

## 💡 Tips

- You can trigger multiple workflow runs in parallel with different HTS codes
- Store your GitHub token as an environment variable for security
- The workflow automatically handles duplicate HTS codes
- Results are de-duplicated and processed efficiently