# Read Aloud Studio - AWS Lambda Functions

AWS Lambda functions serving as a proxy layer between Microsoft Copilot Studio and OpenAI's audio APIs. This enables the "Read Aloud Studio" agent to help communicators, executives, and spokespeople rehearse scripts and speeches through document-to-speech functionality.

## Project Overview

This project provides five Lambda functions integrated with API Gateway:

1. **TTS Proxy** - Converts text to speech using OpenAI TTS API and stores audio in S3
2. **STT Proxy** - Converts speech to text using OpenAI Whisper API
3. **Word Generator** - Generates Word documents from templates stored in S3
4. **Doc Reader** - Extracts text from documents (PDF, Word) stored in S3
5. **Doc TTS** - Reads documents aloud by combining document extraction with TTS

## Architecture

```
Copilot Studio → Power Automate → API Gateway → Lambda Functions → OpenAI API / S3
```

## Features

- ✅ Text-to-Speech (TTS) with multiple voice options
- ✅ Speech-to-Text (STT) with language detection
- ✅ Word document generation from templates
- ✅ S3 storage with presigned URLs
- ✅ Secure API key management via AWS Secrets Manager
- ✅ Comprehensive error handling
- ✅ CORS support for Power Automate
- ✅ CloudWatch logging
- ✅ Production-ready code

## Project Structure

```
lambda_factory/
├── tts_proxy/
│   └── lambda_function.py          # TTS proxy function
├── stt_proxy/
│   └── lambda_function.py          # STT proxy function
├── word_generator/
│   └── lambda_function.py          # Word document generator
├── doc_reader/
│   └── lambda_function.py          # Document text extraction
├── doc_tts/
│   └── lambda_function.py          # Document-to-speech function
├── utils/
│   ├── __init__.py
│   ├── secrets_manager.py          # Secrets Manager helper
│   ├── error_handler.py            # Error handling utilities
│   └── cors.py                     # CORS handling
├── requirements.txt                 # Python dependencies
├── env.template                     # Environment variables template
├── copilot_api_spec.yaml            # OpenAPI spec for Copilot Studio
├── create_lambda_functions.sh       # Create Lambda functions
├── create_api_gateway.sh           # Create API Gateway
├── deploy.sh                        # Deployment script
├── test_word_document.sh            # Test document processing
├── monitor_logs.sh                  # CloudWatch log monitoring
├── MIGRATION_GUIDE.md               # Migration to GitHub guide
└── README.md                        # This file
```

## Quick Start

### Prerequisites

- AWS Account with appropriate permissions
- AWS CLI installed and configured (`aws configure`)
- Python 3.11+
- OpenAI API key
- S3 bucket for storage

### Setup Steps

1. **Clone this repository**
   ```bash
   git clone <repository-url>
   cd lambda_factory
   ```

2. **Configure environment variables**
   ```bash
   # Copy the template
   cp env.template .env
   
   # Edit .env with your values
   # Required: AWS_REGION, S3_BUCKET_NAME, OPENAI_SECRET_NAME, etc.
   ```

3. **Set up AWS Secrets Manager**
   ```bash
   # Store OpenAI API key in Secrets Manager
   aws secretsmanager create-secret \
     --name read-aloud-studio/openai-api-key \
     --secret-string '{"OPENAI_API_KEY":"sk-your-key-here"}' \
     --region us-east-1
   ```

4. **Create Lambda Functions**
   ```bash
   chmod +x create_lambda_functions.sh
   export ENV_SUFFIX=dev  # or staging, prod
   ./create_lambda_functions.sh
   ```

5. **Create API Gateway**
   ```bash
   chmod +x create_api_gateway.sh
   export ENV_SUFFIX=dev
   ./create_api_gateway.sh
   ```

6. **Test Endpoints**
   ```bash
   # Set environment variables
   export API_GATEWAY_ID=your-api-id
   export API_GATEWAY_KEY=your-api-key
   
   # Test document processing
   chmod +x test_word_document.sh
   ./test_word_document.sh path/to/document.docx
   ```

7. **Integrate with Copilot Studio**
   - Use `copilot_api_spec.yaml` to import the API into Copilot Studio
   - See [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) for integration details

### Migration to GitHub

If you're migrating this project to your company's GitHub repository, see [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) for:
- Sensitive information sanitization checklist
- Environment configuration setup
- Secrets management best practices
- CI/CD pipeline setup
- Infrastructure as Code recommendations

## API Endpoints

All endpoints require API key authentication via the `x-api-key` header.

### TTS Endpoint

**POST** `/tts`

Converts text to speech and stores audio in S3.

**Request:**
```json
{
  "text": "Text to convert to speech",
  "voice": "nova",
  "model": "tts-1-hd",
  "speed": 1.0
}
```

**Response:**
```json
{
  "success": true,
  "audioUrl": "https://s3.../audio.mp3?presigned-url",
  "voiceUsed": "nova",
  "modelUsed": "tts-1-hd",
  "speedUsed": 1.0,
  "s3Key": "audio/tts/20240101_120000_abc123.mp3",
  "expiresIn": 3600
}
```

### STT Endpoint

**POST** `/stt`

Converts speech to text using OpenAI Whisper.

**Request:**
```json
{
  "audio": "base64_encoded_audio_string",
  "language": "en",
  "response_format": "text"
}
```

**Response:**
```json
{
  "success": true,
  "transcript": "Transcribed text here",
  "language": "en",
  "format": "text"
}
```

### Document Reader Endpoint

**POST** `/doc`

Extracts text from documents (PDF, Word) stored in S3.

**Request:**
```json
{
  "s3Key": "uploads/document.docx",
  "fileType": "docx"
}
```

**Response:**
```json
{
  "success": true,
  "text": "Extracted text from document...",
  "fileType": "docx",
  "s3Key": "uploads/document.docx"
}
```

### Document-to-Speech Endpoint

**POST** `/doc-tts`

Reads documents aloud by extracting text and converting to speech.

**Request:**
```json
{
  "s3Key": "uploads/document.docx",
  "fileType": "docx",
  "voice": "nova",
  "model": "tts-1-hd",
  "speed": 1.0
}
```

**Response:**
```json
{
  "success": true,
  "audioUrl": "https://s3.../audio.mp3?presigned-url",
  "text": "Extracted text from document...",
  "voiceUsed": "nova",
  "modelUsed": "tts-1-hd",
  "speedUsed": 1.0,
  "s3Key": "audio/tts/20240101_120000_abc123.mp3",
  "expiresIn": 3600
}
```

### Word Generator Endpoint

**POST** `/word`

Generates Word documents from templates stored in S3.

**Request:**
```json
{
  "scriptText": "Script content here",
  "templateName": "template.docx",
  "metadata": {
    "tone": "Formal",
    "voice": "nova",
    "scenario": "Executive briefing"
  }
}
```

**Response:**
```json
{
  "success": true,
  "documentUrl": "https://s3.../document.docx?presigned-url",
  "s3Key": "documents/script_formal_20240101_120000_abc123.docx",
  "filename": "script_formal_20240101_120000_abc123.docx",
  "metadata": {...},
  "expiresIn": 3600
}
```

**OpenAPI Specification:** See `copilot_api_spec.yaml` for complete API documentation.

## Configuration

### Environment Variables

All environment variables should be set via Lambda function configuration. Use `env.template` as a reference.

**Common Variables:**
- `AWS_REGION` - AWS region for Lambda functions (e.g., `us-east-1`)
- `ENV_SUFFIX` - Environment identifier (e.g., `dev`, `staging`, `prod`)
- `S3_BUCKET_NAME` - S3 bucket name for storage
- `S3_REGION` - S3 bucket region (must match actual bucket region)
- `OPENAI_SECRET_NAME` - Secrets Manager secret name (e.g., `read-aloud-studio/openai-api-key`)

**TTS Proxy:**
- `S3_BUCKET_NAME` - S3 bucket for audio storage
- `S3_REGION` - AWS region (must match bucket region)
- `S3_PREFIX` - S3 prefix for audio files (default: `audio/tts/`)
- `OPENAI_SECRET_NAME` - Secrets Manager secret name for OpenAI API key
- `PRESIGNED_URL_EXPIRY` - Presigned URL expiry in seconds (default: 3600)

**STT Proxy:**
- `OPENAI_SECRET_NAME` - Secrets Manager secret name for OpenAI API key

**Doc Reader:**
- `S3_BUCKET_NAME` - S3 bucket for document storage
- `S3_REGION` - AWS region (must match bucket region)
- `S3_UPLOAD_PREFIX` - S3 prefix for uploaded documents (default: `uploads/`)

**Doc TTS:**
- `S3_BUCKET_NAME` - S3 bucket for audio storage
- `S3_REGION` - AWS region (must match bucket region)
- `S3_PREFIX` - S3 prefix for audio files (default: `audio/tts/`)
- `OPENAI_SECRET_NAME` - Secrets Manager secret name for OpenAI API key
- `PRESIGNED_URL_EXPIRY` - Presigned URL expiry in seconds (default: 3600)

**Word Generator:**
- `S3_BUCKET_NAME` - S3 bucket for document storage
- `S3_REGION` - AWS region (must match bucket region)
- `S3_TEMPLATE_PREFIX` - S3 prefix for templates (default: `templates/`)
- `S3_DOCUMENT_PREFIX` - S3 prefix for generated documents (default: `documents/`)
- `PRESIGNED_URL_EXPIRY` - Presigned URL expiry in seconds (default: 3600)

**Important Notes:**
- `S3_REGION` must match the actual S3 bucket region for presigned URLs to work correctly
- Use AWS Secrets Manager for all API keys (recommended) or set `OPENAI_API_KEY` directly (not recommended for production)
- Environment variables are set automatically by `create_lambda_functions.sh`

## Security

### Current Security Measures

- ✅ API keys stored in AWS Secrets Manager (recommended) or environment variables
- ✅ API Gateway API key authentication via `x-api-key` header
- ✅ S3 objects encrypted at rest (AES256)
- ✅ Presigned URLs with expiration (default: 1 hour)
- ✅ IAM roles with least-privilege policies
- ✅ CORS configured for Copilot Studio integration
- ✅ No hardcoded credentials in code (sanitized for GitHub migration)

### Security Best Practices

1. **Secrets Management:**
   - Store all API keys in AWS Secrets Manager
   - Use IAM roles for Lambda functions to access secrets
   - Never commit secrets to version control

2. **API Gateway:**
   - Use API keys for authentication
   - Consider IAM authentication for internal services
   - Enable CloudWatch logging for audit trails

3. **S3 Security:**
   - Use bucket policies to restrict access
   - Enable encryption at rest
   - Use presigned URLs with short expiration times

4. **IAM Roles:**
   - Follow least-privilege principle
   - Grant only necessary permissions
   - Regularly review and audit IAM policies

See [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) for security checklist and company compliance requirements.

## Error Handling

All endpoints return consistent error responses:

```json
{
  "success": false,
  "error": "ErrorType",
  "message": "Human-readable error message"
}
```

**Error Types:**
- `ValidationError` - Invalid input parameters (400)
- `DependencyError` - External service failure (502)
- `InternalServerError` - Unexpected error (500)

## Monitoring

- CloudWatch Logs for all Lambda functions
- API Gateway access logs
- CloudWatch Metrics for invocations, errors, duration
- Set up alarms for error rates and latency

## Cost Optimization

- Lambda: Pay per invocation (first 1M requests free)
- API Gateway: Pay per API call (first 1M requests free)
- S3: Pay for storage and requests
- Secrets Manager: $0.40 per secret per month
- Estimated cost: ~$5-10/month for moderate usage

## Troubleshooting

### Common Issues

1. **401 Unauthorized / 403 Forbidden**
   - Verify API key is set in `x-api-key` header
   - Check API Gateway API key is enabled and valid
   - Verify API key is associated with usage plan
   - Check IAM role permissions for Lambda

2. **500 Internal Server Error**
   - Check CloudWatch logs: `./monitor_logs.sh`
   - Verify Lambda function has permission to invoke (resource-based policy)
   - Check if API Gateway can invoke Lambda (source ARN must include account ID)
   - Verify environment variables are set correctly

3. **S3 Presigned URL Errors**
   - **Critical:** Ensure `S3_REGION` environment variable matches actual bucket region
   - Verify IAM role has `s3:PutObject`, `s3:GetObject`, `s3:HeadObject` permissions
   - Check bucket policy doesn't block presigned URLs
   - Verify bucket name is correct in environment variables

4. **Timeout Errors**
   - Increase Lambda timeout (default: 30-60 seconds)
   - Check OpenAI API status and response times
   - Review CloudWatch logs for slow operations
   - Consider increasing Lambda memory allocation

5. **CORS Errors**
   - Verify CORS headers in Lambda response (handled by `utils/cors.py`)
   - Check API Gateway CORS configuration
   - Test OPTIONS preflight requests
   - Verify allowed origins match Copilot Studio domain

6. **Secrets Manager Errors**
   - Verify secret exists: `aws secretsmanager describe-secret --secret-id <name>`
   - Check IAM role has `secretsmanager:GetSecretValue` permission
   - Verify secret name matches `OPENAI_SECRET_NAME` environment variable
   - Check secret format (should be JSON: `{"OPENAI_API_KEY":"sk-..."}`)

7. **Document Processing Errors**
   - Verify document is uploaded to S3 before calling `/doc` or `/doc-tts`
   - Check file type is supported (PDF, DOCX)
   - Verify S3 key path is correct
   - Check document isn't corrupted or password-protected

### Debugging Commands

```bash
# Check Lambda function configuration
aws lambda get-function --function-name read-aloud-studio-doc_tts-dev

# Check API Gateway configuration
aws apigateway get-rest-api --rest-api-id <api-id>

# Test API endpoint
curl -X POST "https://<api-id>.execute-api.us-east-1.amazonaws.com/prod/tts" \
  -H "Content-Type: application/json" \
  -H "x-api-key: <your-key>" \
  -d '{"text":"Test","voice":"nova","model":"tts-1-hd","speed":1.0}'

# Check IAM role permissions
aws iam get-role-policy --role-name read-aloud-studio-lambda-role-dev --policy-name S3AccessPolicy

# Monitor logs
./monitor_logs.sh
```

### Key Issues Resolved

This project has resolved several critical issues during development:

1. **API Gateway Permissions:** Fixed Lambda resource-based policy to include account ID in source ARN
2. **S3 Region Mismatch:** Ensured `S3_REGION` matches actual bucket region for presigned URLs
3. **S3 Bucket Name:** Standardized bucket name across all functions
4. **IAM Permissions:** Added comprehensive S3 permissions (PutObject, GetObject, HeadObject, ListBucket)
5. **Logging:** Added comprehensive logging for debugging

See [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) for more troubleshooting tips.

## Development

### Local Testing

1. **Set up virtual environment:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set environment variables:**
   ```bash
   # Load from template
   source env.template  # Edit first with your values
   
   # Or set manually
   export OPENAI_API_KEY=your-key
   export S3_BUCKET_NAME=your-bucket
   export S3_REGION=us-east-2
   ```

4. **Test functions locally:**
   - Use AWS SAM for local Lambda testing
   - Or use the AWS Lambda runtime interface emulator
   - Test with `test_word_document.sh` script

### Code Structure

- **Lambda Functions**: Main handler logic in `lambda_function.py` per function
- **Utils**: Shared utilities for secrets, errors, CORS
- **Error Handling**: Consistent error responses across all functions
- **Logging**: Comprehensive CloudWatch logging with structured logs
- **Secrets Management**: Centralized secret retrieval via `utils/secrets_manager.py`

### Monitoring and Debugging

**View CloudWatch Logs:**
```bash
# Monitor logs in real-time
./monitor_logs.sh

# Or specify function name
export LAMBDA_FUNCTION_NAME=read-aloud-studio-doc_tts-dev
./monitor_logs.sh
```

**Common Debugging Steps:**
1. Check CloudWatch logs for errors
2. Verify environment variables are set correctly
3. Test API Gateway endpoints with curl
4. Verify IAM role permissions
5. Check S3 bucket permissions and region

## Contributing

1. Follow Python PEP 8 style guide
2. Add error handling for all external calls
3. Include logging for debugging
4. Update documentation for any changes
5. Test all endpoints before deployment

## License

This project is provided as-is for the Read Aloud Studio use case.

## Support

For issues or questions:
1. Check CloudWatch logs
2. Review SETUP_GUIDE.md troubleshooting section
3. Verify all prerequisites are met
4. Check API Gateway and Lambda function configurations

## Deployment

### Initial Deployment

1. **Create Lambda Functions:**
   ```bash
   export ENV_SUFFIX=dev
   export S3_BUCKET_NAME=your-bucket-name
   export OPENAI_API_KEY=your-key  # Or use Secrets Manager
   ./create_lambda_functions.sh
   ```

2. **Create API Gateway:**
   ```bash
   export ENV_SUFFIX=dev
   ./create_api_gateway.sh
   ```

3. **Set up API Key Authentication:**
   ```bash
   ./setup_api_key_usage_plan.sh
   ```

### Updating Functions

```bash
# Update specific function
./redeploy_tts.sh

# Or update all functions
./deploy.sh
```

### Environment Management

The project supports multiple environments using `ENV_SUFFIX`:
- `dev` - Development environment
- `staging` - Staging environment
- `prod` - Production environment

Set `ENV_SUFFIX` before running deployment scripts.

## Migration to GitHub

This project has been prepared for migration to company GitHub repositories:

- ✅ Sensitive information sanitized (API keys, account IDs removed)
- ✅ Environment variables template created (`env.template`)
- ✅ `.gitignore` updated to exclude sensitive files
- ✅ Migration guide created (`MIGRATION_GUIDE.md`)

**Before committing to GitHub:**
1. Review [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)
2. Verify no sensitive data in git history
3. Set up company-specific secrets management
4. Configure CI/CD pipeline
5. Review company compliance requirements

## Next Steps

- ✅ Complete setup using environment template
- ✅ Deploy functions using `create_lambda_functions.sh`
- ✅ Create API Gateway using `create_api_gateway.sh`
- ✅ Test endpoints using `test_word_document.sh`
- ✅ Integrate with Copilot Studio using `copilot_api_spec.yaml`
- ✅ Set up monitoring and alerts in CloudWatch
- ✅ Configure cost alerts in AWS Billing
- 📋 Review [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) for GitHub migration

---

**Built for Read Aloud Studio - Empowering communicators to rehearse and refine their delivery.**

**Project Status:** Production-ready, prepared for GitHub migration


