# Detailed Setup Guide

## Step-by-Step Configuration

### 1. Twilio WhatsApp Sandbox Setup

#### Create Twilio Account
1. Go to [twilio.com](https://www.twilio.com) and sign up
2. Verify your email and phone number
3. Get your Account SID and Auth Token from the Console Dashboard

#### Enable WhatsApp Sandbox
1. In Twilio Console, go to **Messaging** → **Try it out** → **Send a WhatsApp message**
2. You'll see a WhatsApp number (usually +14155238886)
3. Send the provided code to that number to join your sandbox
4. Note: Sandbox expires after 72 hours, you'll need to rejoin

#### Configure Webhook
1. In Twilio Console, go to **Messaging** → **Settings** → **WhatsApp Sandbox Settings**
2. Set the webhook URL to: `https://your-n8n-domain.com/webhook`
3. Set HTTP method to POST
4. Save the configuration

### 2. Google Cloud Platform Setup

#### Create Project
1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project or select existing one
3. Enable billing (required for API usage)

#### Enable APIs
1. Go to **APIs & Services** → **Library**
2. Search and enable these APIs:
   - Google Drive API
   - Google Sheets API
   - Google Docs API (for document processing)

#### Create OAuth 2.0 Credentials
1. Go to **APIs & Services** → **Credentials**
2. Click **Create Credentials** → **OAuth 2.0 Client IDs**
3. Application type: **Web application**
4. Name: `n8n-whatsapp-drive`
5. Authorized redirect URIs:
   - `http://localhost:5678/callback` (for local development)
   - `https://your-domain.com/callback` (for production)
6. Click **Create**
7. Download the JSON file and save it securely

#### Create Service Account (for Google Sheets)
1. Go to **APIs & Services** → **Credentials**
2. Click **Create Credentials** → **Service Account**
3. Name: `n8n-audit-log`
4. Grant **Editor** role
5. Create and download the JSON key file

### 3. Google Sheets Setup

#### Create Audit Log Sheet
1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new spreadsheet
3. Name it: `WhatsApp Drive Bot - Audit Log`
4. Add these headers in row 1:
   - A1: `Timestamp`
   - B1: `Phone Number`
   - C1: `Command`
   - D1: `Folder`
   - E1: `File`
   - F1: `Status`
   - G1: `Result`
5. Format headers as bold
6. Copy the Spreadsheet ID from the URL (long string between /d/ and /edit)

#### Share with Service Account
1. Click **Share** button
2. Add your service account email (from the JSON file)
3. Grant **Editor** permissions
4. Click **Done**

### 4. OpenAI API Setup

#### Get API Key
1. Go to [OpenAI Platform](https://platform.openai.com)
2. Sign up or log in
3. Go to **API Keys** section
4. Click **Create new secret key**
5. Name it: `n8n-whatsapp-bot`
6. Copy the key (starts with `sk-`)
7. **Important**: Save it securely, you won't see it again

#### Verify Access
1. Ensure you have access to GPT-4o model
2. Check your usage limits and billing
3. Note: GPT-4o requires a paid account

### 5. n8n Installation

#### Option A: Docker Installation (Recommended)

1. **Install Docker and Docker Compose**
   ```bash
   # Ubuntu/Debian
   sudo apt update
   sudo apt install docker.io docker-compose
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker $USER
   ```

2. **Create Project Directory**
   ```bash
   mkdir n8n-whatsapp-drive
   cd n8n-whatsapp-drive
   ```

3. **Copy Files**
   ```bash
   # Copy the provided files
   cp workflow.json ./
   cp docker-compose.yml ./
   cp .env.sample ./
   ```

4. **Configure Environment**
   ```bash
   cp .env.sample .env
   nano .env  # Edit with your credentials
   ```

5. **Start n8n**
   ```bash
   docker-compose up -d
   ```

6. **Access n8n**
   - Open `http://localhost:5678`
   - Login with admin/your_secure_password_here

#### Option B: Local Installation

1. **Install Node.js**
   ```bash
   # Ubuntu/Debian
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```

2. **Install n8n**
   ```bash
   npm install -g n8n
   ```

3. **Start n8n**
   ```bash
   n8n start
   ```

### 6. Workflow Configuration

#### Import Workflow
1. Open n8n interface
2. Click **Import from file**
3. Select `workflow.json`
4. Click **Import**

#### Configure Credentials
1. **Twilio Credentials**
   - Click on Twilio nodes
   - Click **Create New Credential**
   - Name: `Twilio WhatsApp`
   - Account SID: Your Twilio Account SID
   - Auth Token: Your Twilio Auth Token
   - Save

2. **Google Drive Credentials**
   - Click on Google Drive nodes
   - Click **Create New Credential**
   - Name: `Google Drive OAuth2`
   - Client ID: From your OAuth JSON file
   - Client Secret: From your OAuth JSON file
   - Authorize the application

3. **OpenAI Credentials**
   - Click on OpenAI nodes
   - Click **Create New Credential**
   - Name: `OpenAI API`
   - API Key: Your OpenAI API key
   - Save

4. **Google Sheets Credentials**
   - Click on Google Sheets nodes
   - Click **Create New Credential**
   - Name: `Google Sheets Service Account`
   - Service Account: Upload your service account JSON file
   - Save

#### Configure Nodes
1. **Webhook Node**
   - Note the webhook URL
   - Copy it to your Twilio webhook configuration

2. **Google Drive Nodes**
   - Update folder IDs to match your Google Drive structure
   - Test with a simple folder first

3. **Google Sheets Node**
   - Update Spreadsheet ID to your audit log sheet
   - Test by running the workflow manually

#### Activate Workflow
1. Click the **Active** toggle
2. The webhook will be available at your n8n URL
3. Test with a WhatsApp message

### 7. Testing

#### Test Basic Commands
1. Send `HELP` to your WhatsApp number
2. You should receive the help message
3. Test `LIST /your-folder-name`
4. Verify files are listed correctly

#### Test File Operations
1. Create a test file in Google Drive
2. Test `DELETE /folder/test-file.txt`
3. Confirm the deletion prompt
4. Send `CONFIRM DELETE`

#### Test AI Summarization
1. Upload a PDF or text file to Google Drive
2. Test `SUMMARY /folder-name`
3. Verify AI summary is generated

### 8. Production Deployment

#### Domain Setup
1. Get a domain name
2. Set up SSL certificate
3. Configure reverse proxy (nginx recommended)

#### Environment Variables
```bash
# Production .env
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=very_secure_password
N8N_ENCRYPTION_KEY=32_character_random_string
WEBHOOK_URL=https://your-domain.com
N8N_PROTOCOL=https
N8N_HOST=your-domain.com
```

#### Security Considerations
1. Use strong passwords
2. Enable 2FA if available
3. Regular backups
4. Monitor logs
5. Set up alerts for failures

### 9. Troubleshooting

#### Common Issues

**Webhook Not Working**
- Check Twilio webhook URL
- Verify n8n is accessible
- Check firewall settings
- Review n8n logs

**Google Drive Authentication**
- Re-authenticate OAuth2
- Check API quotas
- Verify folder permissions
- Test with simple operations first

**AI Summarization Fails**
- Check OpenAI API key
- Verify billing status
- Check file format support
- Review API usage limits

**Audit Log Issues**
- Verify service account permissions
- Check spreadsheet sharing
- Validate spreadsheet ID
- Test manual sheet access

#### Debug Commands
```bash
# Check n8n logs
docker-compose logs n8n

# Restart n8n
docker-compose restart n8n

# Check webhook status
curl -X POST https://your-domain.com/webhook -d "test"

# Verify environment
docker-compose exec n8n env | grep N8N
```

### 10. Maintenance

#### Regular Tasks
1. **Monitor Usage**
   - Check Twilio usage
   - Monitor OpenAI API costs
   - Review Google Drive storage

2. **Backup Data**
   - Export n8n workflows
   - Backup Google Sheets
   - Save configuration files

3. **Update Dependencies**
   - Update n8n regularly
   - Check for security patches
   - Review API changes

4. **Performance Optimization**
   - Monitor response times
   - Optimize workflow efficiency
   - Scale resources as needed

This setup guide provides comprehensive instructions for deploying the WhatsApp Google Drive integration workflow. Follow each step carefully and test thoroughly before going live.
