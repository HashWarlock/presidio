# Presidio PII Detection Demo on Phala Cloud

This repository contains a Streamlit-based demo application for [Microsoft Presidio](https://microsoft.github.io/presidio/), a powerful open-source framework for PII (Personally Identifiable Information) detection and de-identification. This demo is optimized for deployment on Phala Cloud's Confidential Virtual Machines (CVMs).

## What is Presidio?

Microsoft Presidio is a data protection and de-identification SDK that helps organizations:
- **Detect PII** in text using advanced NLP models and pattern recognition
- **Anonymize sensitive data** through various techniques (redaction, masking, encryption, replacement)
- **Customize detection** with custom recognizers and deny/allow lists
- **Support multiple languages** and NLP frameworks (spaCy, Transformers, Flair, Stanza)

## Demo Application Features

This Streamlit demo provides an interactive web interface to explore Presidio's capabilities:

### 🔍 **PII Detection**
- **Multiple NLP Models**: Choose from various pre-trained models including:
  - spaCy models (en_core_web_lg)
  - Flair NER models (ner-english-large)
  - HuggingFace Transformers (stanford-deidentifier-base, deid_roberta_i2b2)
  - Stanza models
- **Entity Types**: Detects 20+ entity types including:
  - Personal identifiers (names, SSN, phone numbers)
  - Financial data (credit cards, bank accounts)
  - Location data (addresses, IP addresses)
  - Healthcare identifiers
  - Custom patterns via regex

### 🛡️ **De-identification Methods**
- **Redact**: Completely remove PII text
- **Replace**: Substitute with generic placeholders (e.g., `<PERSON>`)
- **Mask**: Replace characters with asterisks or custom characters
- **Hash**: Replace with SHA-256 hashes
- **Encrypt**: AES encryption (reversible with key)
- **Synthesize**: Generate fake but realistic data using OpenAI (requires API key)
- **Highlight**: Visual annotation of detected PII

### ⚙️ **Customization Options**
- **Confidence Threshold**: Adjust sensitivity of PII detection
- **Allow/Deny Lists**: Fine-tune detection with custom word lists
- **Custom Regex**: Add domain-specific patterns
- **Analysis Explanations**: View detailed decision processes

### 🎯 **Use Cases**
- **Data Privacy Compliance**: GDPR, CCPA, HIPAA compliance
- **Data Anonymization**: Safely share datasets for analytics
- **Document Redaction**: Prepare documents for public release
- **Testing & Development**: Generate synthetic test data
- **Security Audits**: Identify PII exposure in systems

## Quick Start on Phala Cloud

### Prerequisites
1. Install [Phala Cloud CLI](https://docs.phala.network/)
2. Have a Phala Cloud account and API credentials configured
3. Create a `.env` file with your configuration (optional)

### Step 1: Clone and Navigate
```bash
git clone https://github.com/HashWarlock/presidio.git
cd presidio/docs/samples/python/streamlit
```

### Step 2: Configure Environment (Optional)
Create a `.env` file to customize the application:

```bash
# OpenAI Configuration (for text synthesis feature)
OPENAI_TYPE=openai
OPENAI_KEY=your_openai_api_key_here
OPENAI_API_VERSION=2024-07-18
OPENAI_MODEL=gpt-3.5-turbo

# Model Configuration
ALLOW_OTHER_MODELS=true
```

> **Note**: The OpenAI configuration is optional. The app works fully without it, but you won't have access to the "synthesize" feature that generates fake data.

### Step 3: Deploy to Phala Cloud
Deploy the application using the Phala Cloud CLI:

```bash
phala cvms create -n presidio-tee -c docker-compose.yml --vcpu 2 --memory 8394 --disk-size 80 -e .env
```

**Command Parameters:**
- `-n presidio-tee`: Names your deployment "presidio-tee"
- `-c docker-compose.yml`: Uses the Docker Compose configuration
- `--vcpu 2`: Allocates 2 virtual CPUs
- `--memory 8394`: Allocates ~8GB RAM (recommended for NLP models)
- `--disk-size 80`: Allocates 80GB disk space
- `-e .env`: Loads environment variables from .env file

### Step 4: Access Your Application
After deployment, Phala Cloud will provide you with a URL to access your Streamlit application. The app will be running on port 7860.

## Resource Requirements

### Recommended Specifications
- **CPU**: 2+ vCPUs (for model inference)
- **Memory**: 8GB+ RAM (NLP models are memory-intensive)
- **Storage**: 80GB+ (for model downloads and caching)
- **Network**: Stable internet for model downloads

### Performance Notes
- **First Launch**: May take 5-10 minutes to download NLP models
- **Model Loading**: Larger models (like transformers) take longer to initialize
- **Concurrent Users**: Scale CPU/memory for multiple simultaneous users

## Application Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Streamlit     │    │     Presidio     │    │   NLP Models    │
│   Frontend      │◄──►│     Engine       │◄──►│  (spaCy, etc.)  │
│                 │    │                  │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   User Input    │    │  PII Detection   │    │   OpenAI API    │
│   & Results     │    │  & Anonymization │    │  (Optional)     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## Security & Privacy

### Phala Cloud Benefits
- **Confidential Computing**: Your data is processed in secure enclaves
- **TEE Protection**: Trusted Execution Environment ensures data privacy
- **No Data Persistence**: Processed text is not stored permanently
- **Encrypted Communication**: All network traffic is encrypted

### Local Processing
- PII detection runs entirely within your CVM
- No data is sent to external services (except optional OpenAI synthesis)
- Models are downloaded once and cached locally

## Sample Data

The demo includes example text containing various PII types:
```
Hello, my name is David Johnson and I live in Maine.
My credit card number is 4095-2609-9393-4932 and my crypto wallet id is 16Yeky6GMjeNkAiNcBY7ZhrLoMSgg1BoyZ.
Kate's social security number is 078-05-1126. Her driver license? it is 1234567A.
```

## Troubleshooting

### Common Issues

**Out of Memory Errors**
- Increase memory allocation: `--memory 16000` (16GB)
- Use smaller models like `en_core_web_sm` instead of `en_core_web_lg`

**Slow Model Loading**
- First-time model downloads can take 10+ minutes
- Check CVM logs at the [Phala Cloud Dashboard](dashboard.cloud.phala.network)

**OpenAI Synthesis Not Working**
- Verify your OpenAI API key in the `.env` file
- Check API quota and billing status
- Ensure `OPENAI_TYPE` is set correctly

### Monitoring
```bash
# Check deployment status
phala cvms list

# Check Remote Attestation
phala cvms attestation
```

## Advanced Configuration

### Custom Models
To use custom NLP models, modify the docker-compose.yml to mount your model directory:

```yaml
volumes:
  - ./custom_models:/home/user/app/models
```

### Scaling
For production use, consider:
- Load balancer for multiple CVMs
- Persistent storage for model caching
- Redis for session management

## Support & Documentation

- **Presidio Documentation**: https://microsoft.github.io/presidio/
- **Phala Cloud Docs**: https://docs.phala.network/
- **Issues & Bugs**: https://github.com/microsoft/presidio/issues
- **Community**: Join the Presidio Discord/Slack

## License

This demo is provided under the MIT License. See the main Presidio repository for full license details.

---

**Ready to deploy?** Run the command above and start exploring PII detection with Presidio on Phala Cloud! 🚀 