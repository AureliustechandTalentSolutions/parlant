# Parlant Platform Integrations

This directory contains integration guides for deploying and using Parlant with various platforms and systems.

## Available Integrations

### Federal & Government Platforms

#### [Aurelius Federal Platform](./aurelius-federal-platform.md)
Comprehensive guide for integrating Parlant with the Aurelius Federal Platform's Resource Library. Includes:
- Federal compliance considerations
- Security configurations
- Deployment options (Docker, Kubernetes)
- Integration examples for federal use cases

**Quick Start**: [Aurelius Quick Start Guide](./aurelius-quickstart.md)

## Integration Overview

Parlant is designed to integrate seamlessly with enterprise and government platforms through:

### 1. REST API
Full-featured REST API for all operations:
- Agent management
- Session handling
- Guideline configuration
- Tool integration

### 2. SDK Support
Official client libraries:
- **Python**: `pip install parlant-client`
- **TypeScript/JavaScript**: `npm install parlant-client`

### 3. LLM Provider Support
Compatible with major LLM providers:
- OpenAI
- Azure OpenAI (including Government Cloud)
- AWS Bedrock (including GovCloud, FedRAMP authorized)
- Google Vertex AI
- Anthropic
- Custom providers via adapter pattern

### 4. Deployment Flexibility
Multiple deployment options:
- Standalone server
- Docker containers
- Kubernetes clusters
- Cloud platforms (AWS, Azure, GCP)
- On-premises installations

## Federal & Government Compliance

Parlant is designed with federal requirements in mind:

### Security Features
- ✅ Apache 2.0 License (Federal-friendly)
- ✅ Custom authentication policies
- ✅ Audit logging
- ✅ Data encryption (at rest and in transit)
- ✅ On-premises deployment option
- ✅ No data leaves your infrastructure by default

### Compliance Readiness
- 🔄 FedRAMP Moderate (In Progress)
- ✅ FISMA Compatible
- ✅ Section 508 Compliant
- ✅ NIST AI RMF Aligned

## Common Integration Patterns

### Pattern 1: Secure API Integration

```python
import parlant.sdk as p

@p.tool
async def secure_api_call(context: p.ToolContext, data: str) -> p.ToolResult:
    """Call external API with proper authentication"""
    # Implement your secure API call
    return p.ToolResult({"status": "success"})
```

### Pattern 2: Custom Authentication

```python
from parlant.api.authorization import AuthorizationPolicy

class CustomAuthPolicy(AuthorizationPolicy):
    async def authorize(self, request, operation):
        # Your authentication logic
        return True
```

### Pattern 3: Database Integration

```python
# Configure PostgreSQL for production
export DATABASE_URL="postgresql://user:pass@host:5432/parlant"
```

## Adding New Integrations

To add a new platform integration guide:

1. Create a new markdown file: `platform-name.md`
2. Include the following sections:
   - Overview
   - Prerequisites
   - Installation
   - Configuration
   - Deployment Options
   - Security Considerations
   - Examples
   - Troubleshooting
   - Support

3. Update this README with a link to your guide

## Support

For integration support:

- **Documentation**: https://parlant.io/docs
- **GitHub Issues**: https://github.com/AureliustechandTalentSolutions/parlant/issues
- **Community Discord**: https://discord.gg/duxWqxKk6J
- **Email**: support@parlant.io

For federal/government specific support:
- **Email**: federal-support@parlant.io

## Contributing

We welcome integration guides for additional platforms! Please see our [Contributing Guide](../../CONTRIBUTING.md) for details on how to submit your integration documentation.

## License

All integration documentation is provided under the same Apache 2.0 license as Parlant.
