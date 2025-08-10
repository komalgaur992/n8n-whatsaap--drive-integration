# Troubleshooting Guide

This guide helps you resolve common issues when setting up and running the WhatsApp Google Drive integration.

## 🔍 Common Issues and Solutions

### 1. WhatsApp Integration Issues

#### Problem: Messages not received in n8n
**Symptoms:**
- WhatsApp messages sent but no response
- Webhook not triggering in n8n
- Error logs in Twilio console

**Solutions:**
1. **Check Webhook URL**
   ```bash
   # Verify webhook is accessible
   curl -X POST https://your-n8n-domain.com/webhook -d "test"
   ```

2. **Verify Twilio Configuration**
   - Log into Twilio Console
   - Go to Messaging → Settings → WhatsApp Sandbox Settings
   - Ensure webhook URL is correct: `https://your-domain.com/webhook`
   - Verify HTTP method is POST

3. **Check n8n Webhook Status**
   - Open n8n interface
   - Go to Settings → Webhooks
   - Verify webhook is active and accessible
   - Check webhook URL matches Twilio configuration

4. **Test Webhook Manually**
   ```bash
   # Test with sample Twilio payload
   curl -X POST https://your-domain.com/webhook \
     -H "Content-Type: application/json" \
     -d '{
       "Body": "HELP",
       "From": "whatsapp:+1234567890",
       "To": "whatsapp:+14155238886"
     }'
   ```

#### Problem: WhatsApp Sandbox expired
**Symptoms:**
- Messages not delivered
- "Sandbox expired" error

**Solutions:**
1. **Rejoin Sandbox**
   - Go to Twilio Console → Messaging → Try it out
   - Send the new code to the WhatsApp number
   - Wait for confirmation message

2. **Upgrade to Production**
   - Apply for WhatsApp Business API
   - Complete business verification
   - Use production WhatsApp number

### 2. Google Drive Integration Issues

#### Problem: Authentication errors
**Symptoms:**
- "Invalid credentials" errors
- OAuth2 authorization failures
- Permission denied errors

**Solutions:**
1. **Re-authenticate OAuth2**
   - Go to n8n → Credentials
   - Find Google Drive credentials
   - Click "Test" or "Update"
   - Complete OAuth2 flow

2. **Check API Quotas**
   - Go to Google Cloud Console
   - Navigate to APIs & Services → Quotas
   - Check Google Drive API usage
   - Request quota increase if needed

3. **Verify API Enablement**
   - Ensure Google Drive API is enabled
   - Enable Google Sheets API if using audit logging
   - Check API status in Google Cloud Console

4. **Update OAuth2 Credentials**
   ```bash
   # Check redirect URIs
   # Should include: http://localhost:5678/callback
   # And: https://your-domain.com/callback
   ```

#### Problem: Folder not found errors
**Symptoms:**
- "Folder not found" responses
- Empty file lists
- Permission errors

**Solutions:**
1. **Verify Folder Structure**
   - Check Google Drive folder names
   - Ensure folders exist and are accessible
   - Use correct folder IDs or names

2. **Check Permissions**
   - Verify OAuth2 account has access to folders
   - Check folder sharing settings
   - Ensure service account has proper permissions

3. **Test with Simple Operations**
   ```bash
   # Test with root folder first
   LIST /
   
   # Then test with specific folders
   LIST /Documents
   ```

### 3. AI Summarization Issues

#### Problem: OpenAI API errors
**Symptoms:**
- "API key invalid" errors
- Rate limit exceeded
- Model not available

**Solutions:**
1. **Verify API Key**
   - Check OpenAI API key is correct
   - Ensure key starts with `sk-`
   - Verify key has proper permissions

2. **Check Billing Status**
   - Log into OpenAI Platform
   - Check billing and usage
   - Add payment method if needed
   - Verify GPT-4o access

3. **Handle Rate Limits**
   ```javascript
   // Add retry logic in workflow
   // Wait between requests
   // Use exponential backoff
   ```

4. **Alternative Models**
   - Use GPT-3.5-turbo if GPT-4o unavailable
   - Consider other AI providers
   - Implement fallback summarization

#### Problem: Text extraction failures
**Symptoms:**
- "No text found" errors
- Empty summaries
- File format not supported

**Solutions:**
1. **Check File Formats**
   - Supported: PDF, DOCX, TXT
   - Ensure files are not corrupted
   - Check file permissions

2. **Improve Text Extraction**
   ```javascript
   // Add better text extraction logic
   // Handle different PDF formats
   // Extract from images if needed
   ```

3. **File Size Limits**
   - Check file size (max 10MB recommended)
   - Split large documents
   - Compress files if needed

### 4. Audit Logging Issues

#### Problem: Google Sheets not updating
**Symptoms:**
- No audit entries in sheet
- Permission denied errors
- Spreadsheet not found

**Solutions:**
1. **Check Service Account**
   - Verify service account JSON is correct
   - Ensure service account has Editor permissions
   - Check API quotas for Google Sheets

2. **Verify Spreadsheet**
   - Check spreadsheet ID is correct
   - Ensure spreadsheet exists and is accessible
   - Verify sheet name is "Audit Log"

3. **Test Manual Access**
   ```bash
   # Test service account access
   curl -H "Authorization: Bearer YOUR_TOKEN" \
     https://sheets.googleapis.com/v4/spreadsheets/SPREADSHEET_ID
   ```

### 5. n8n Workflow Issues

#### Problem: Workflow not activating
**Symptoms:**
- Workflow shows as inactive
- Webhook not accessible
- Nodes showing errors

**Solutions:**
1. **Check Workflow Status**
   - Ensure workflow is active (toggle on)
   - Check for node errors
   - Verify all credentials are set

2. **Test Individual Nodes**
   - Test each node manually
   - Check node configurations
   - Verify data flow between nodes

3. **Check n8n Logs**
   ```bash
   # Docker logs
   docker-compose logs n8n
   
   # Local installation
   n8n start --debug
   ```

#### Problem: Performance issues
**Symptoms:**
- Slow response times
- Timeout errors
- High resource usage

**Solutions:**
1. **Optimize Workflow**
   - Reduce unnecessary API calls
   - Implement caching where possible
   - Use batch operations

2. **Scale Resources**
   - Increase Docker memory limits
   - Add more CPU resources
   - Use load balancing

3. **Monitor Performance**
   ```bash
   # Check resource usage
   docker stats
   
   # Monitor API response times
   # Implement performance logging
   ```

## 🛠️ Debug Commands

### Check System Status
```bash
# Check n8n status
docker-compose ps

# Check logs
docker-compose logs -f n8n

# Check environment
docker-compose exec n8n env | grep N8N

# Check webhook accessibility
curl -I https://your-domain.com/webhook
```

### Test API Connections
```bash
# Test Twilio API
curl -u "YOUR_ACCOUNT_SID:YOUR_AUTH_TOKEN" \
  https://api.twilio.com/2010-04-01/Accounts/YOUR_ACCOUNT_SID/Messages.json

# Test OpenAI API
curl -H "Authorization: Bearer YOUR_OPENAI_KEY" \
  https://api.openai.com/v1/models

# Test Google Drive API
curl -H "Authorization: Bearer YOUR_GOOGLE_TOKEN" \
  https://www.googleapis.com/drive/v3/files
```

### Verify Credentials
```bash
# Check credential files
ls -la ~/.n8n/

# Verify environment variables
echo $TWILIO_ACCOUNT_SID
echo $OPENAI_API_KEY
echo $GOOGLE_DRIVE_CLIENT_ID
```

## 📊 Monitoring and Alerts

### Set Up Monitoring
1. **Health Checks**
   ```bash
   # Create health check script
   #!/bin/bash
   curl -f https://your-domain.com/healthz || echo "n8n down"
   ```

2. **Log Monitoring**
   ```bash
   # Monitor error logs
   tail -f /var/log/n8n/error.log | grep -i error
   
   # Monitor webhook access
   tail -f /var/log/nginx/access.log | grep webhook
   ```

3. **API Usage Monitoring**
   - Monitor Twilio usage and costs
   - Track OpenAI API usage
   - Check Google API quotas

### Alert Setup
1. **Email Alerts**
   - Set up email notifications for failures
   - Configure alert thresholds
   - Include relevant error details

2. **Slack/Discord Integration**
   - Send alerts to team channels
   - Include workflow status
   - Add quick action buttons

## 🔧 Advanced Troubleshooting

### Debug Mode
```bash
# Enable debug logging
N8N_LOG_LEVEL=debug docker-compose up

# Or for local installation
N8N_LOG_LEVEL=debug n8n start
```

### Workflow Debugging
1. **Add Debug Nodes**
   - Insert Code nodes to log data
   - Use Set nodes to inspect variables
   - Add error handling nodes

2. **Test Data Flow**
   ```javascript
   // Add this to Code nodes for debugging
   console.log('Input data:', JSON.stringify($input.all(), null, 2));
   return $input.all();
   ```

### Performance Profiling
```bash
# Monitor Docker resources
docker stats n8n-whatsapp-drive

# Check network connections
netstat -tulpn | grep 5678

# Monitor disk usage
df -h /var/lib/docker/volumes/
```

## 📞 Getting Help

### Before Asking for Help
1. **Collect Information**
   - Error messages and logs
   - System configuration
   - Steps to reproduce
   - Expected vs actual behavior

2. **Check Documentation**
   - Review this troubleshooting guide
   - Check n8n documentation
   - Search existing issues

3. **Test Isolated Components**
   - Test each service independently
   - Verify credentials separately
   - Check network connectivity

### Where to Get Help
1. **n8n Community**
   - [n8n Community Forum](https://community.n8n.io/)
   - [GitHub Issues](https://github.com/n8n-io/n8n/issues)

2. **Service Documentation**
   - [Twilio Documentation](https://www.twilio.com/docs)
   - [OpenAI API Documentation](https://platform.openai.com/docs)
   - [Google Drive API Documentation](https://developers.google.com/drive)

3. **Stack Overflow**
   - Search for similar issues
   - Ask specific questions
   - Include relevant code and error messages

This troubleshooting guide should help you resolve most common issues. If you continue to experience problems, please collect the relevant information and seek help from the community or support channels.
