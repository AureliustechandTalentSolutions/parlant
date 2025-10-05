# Aurelius Federal Platform - Parlant Quick Start

This quick start guide helps you get Parlant running in the Aurelius Federal Platform environment in under 10 minutes.

## Prerequisites

Before starting, ensure you have:

- Access to the Aurelius Federal Platform
- Python 3.10 or higher installed
- API credentials for an approved LLM provider (Azure Government Cloud recommended)
- Network access to required services

## Step 1: Install Parlant

For Azure Government Cloud integration (recommended):

```bash
pip install parlant[azure]
```

For AWS GovCloud integration:

```bash
pip install parlant[aws]
```

## Step 2: Configure Environment

Create a `.env` file with your credentials:

```bash
# For Azure Government Cloud
export AZURE_OPENAI_API_KEY="your-key-here"
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.us/"
export AZURE_OPENAI_API_VERSION="2024-02-15-preview"
export AZURE_OPENAI_CHAT_MODEL="gpt-4"

# Optional: Configure data directory
export PARLANT_HOME="/var/lib/parlant"
```

For AWS GovCloud:

```bash
# AWS GovCloud Configuration
export AWS_REGION="us-gov-west-1"
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_BEDROCK_MODEL="anthropic.claude-3-sonnet-20240229-v1:0"
```

## Step 3: Create Your First Federal Agent

Create a file called `federal_agent.py`:

```python
import parlant.sdk as p
import asyncio

@p.tool
async def lookup_regulation(
    context: p.ToolContext,
    regulation_number: str
) -> p.ToolResult:
    """Look up federal regulation details"""
    # Integration with your regulation database
    return p.ToolResult({
        "regulation": regulation_number,
        "status": "active",
        "summary": "Regulation details..."
    })

async def main():
    # Use Azure Government Cloud
    async with p.Server(nlp_service=p.NLPServices.azure) as server:
        
        # Create a compliance-focused agent
        agent = await server.create_agent(
            name="Compliance Assistant",
            description="Assists federal employees with regulatory compliance questions"
        )
        
        # Add compliance guidelines
        await agent.create_guideline(
            condition="User asks about a regulation",
            action="Look up the regulation and provide accurate, official information",
            tools=[lookup_regulation]
        )
        
        await agent.create_guideline(
            condition="Request involves personal information",
            action="Verify user authorization before proceeding and log the request"
        )
        
        await agent.create_guideline(
            condition="User asks about sensitive information",
            action="Remind user about classification levels and ensure proper clearance"
        )
        
        print("✓ Agent created successfully!")
        print("✓ Compliance guidelines configured")
        print("✓ Server running at http://localhost:8800")
        print("\nPress Ctrl+C to stop the server")

if __name__ == "__main__":
    asyncio.run(main())
```

## Step 4: Run Your Agent

```bash
python federal_agent.py
```

## Step 5: Test Your Agent

Open your browser and navigate to:

```
http://localhost:8800
```

Try these test prompts:
- "What is regulation CFR 1234?"
- "I need information about employee benefits"
- "Tell me about data classification requirements"

## Step 6: Deploy to Aurelius Platform

### Option A: Docker Deployment

1. Create a `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install Parlant with Azure support
RUN pip install --no-cache-dir parlant[azure]

# Copy your agent code
COPY federal_agent.py .

# Expose the port
EXPOSE 8800

# Run the agent
CMD ["python", "federal_agent.py"]
```

2. Build and deploy:

```bash
docker build -t parlant-federal-agent .
docker run -d -p 8800:8800 --env-file .env parlant-federal-agent
```

### Option B: Kubernetes Deployment

1. Create a `ConfigMap` for configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: parlant-config
  namespace: aurelius-ai
data:
  federal_agent.py: |
    # Your agent code here
```

2. Create a `Secret` for credentials:

```bash
kubectl create secret generic parlant-secrets \
  --from-literal=azure-api-key='your-key' \
  --from-literal=azure-endpoint='https://your-resource.openai.azure.us/' \
  -n aurelius-ai
```

3. Deploy:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: parlant-federal-agent
  namespace: aurelius-ai
spec:
  replicas: 2
  selector:
    matchLabels:
      app: parlant-agent
  template:
    metadata:
      labels:
        app: parlant-agent
    spec:
      containers:
      - name: parlant
        image: parlant-federal-agent:latest
        ports:
        - containerPort: 8800
        env:
        - name: AZURE_OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: parlant-secrets
              key: azure-api-key
        - name: AZURE_OPENAI_ENDPOINT
          valueFrom:
            secretKeyRef:
              name: parlant-secrets
              key: azure-endpoint
```

## Step 7: Configure Production Settings

For production deployments, add these configurations:

### Database Configuration

Use PostgreSQL for production:

```bash
export DATABASE_URL="postgresql://user:password@postgres.aurelius.internal:5432/parlant"
```

### Logging Configuration

Configure audit logging:

```bash
export PARLANT_LOG_LEVEL="info"
export PARLANT_LOG_FILE="/var/log/parlant/agent.log"
```

### Security Configuration

Enable authentication and rate limiting:

```python
async def main():
    async with p.Server(
        nlp_service=p.NLPServices.azure,
        configure_container=configure_security
    ) as server:
        # ... your agent setup
        pass

async def configure_security(container):
    # Add your custom authorization policy
    # Configure rate limiting
    # Set up audit logging
    return container
```

## Step 8: Monitor Your Agent

### Health Check

```bash
curl http://localhost:8800/health
```

### View Logs

```bash
tail -f /var/log/parlant/agent.log
```

### Check Metrics

Parlant provides metrics endpoints for monitoring:

```bash
curl http://localhost:8800/api/metrics
```

## Common Configuration Patterns

### Pattern 1: Multi-Agent Setup

```python
async def main():
    async with p.Server(nlp_service=p.NLPServices.azure) as server:
        
        # Customer service agent
        customer_agent = await server.create_agent(
            name="Customer Service",
            description="Helps citizens with general inquiries"
        )
        
        # Benefits agent
        benefits_agent = await server.create_agent(
            name="Benefits Specialist",
            description="Assists with federal benefits programs"
        )
        
        # Compliance agent
        compliance_agent = await server.create_agent(
            name="Compliance Officer",
            description="Ensures regulatory compliance"
        )
```

### Pattern 2: Federal API Integration

```python
@p.tool
async def query_federal_database(
    context: p.ToolContext,
    query: str
) -> p.ToolResult:
    """Query secure federal database"""
    # Use proper authentication and encryption
    headers = {
        "Authorization": f"Bearer {get_secure_token()}",
        "X-Request-ID": context.correlation_id
    }
    
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "https://api.aurelius.gov/query",
            json={"query": query},
            headers=headers,
            verify=True  # Ensure SSL verification
        )
    
    return p.ToolResult(response.json())
```

### Pattern 3: Compliance Logging

```python
@p.tool
async def process_sensitive_request(
    context: p.ToolContext,
    request_type: str,
    details: str
) -> p.ToolResult:
    """Process requests involving sensitive data"""
    
    # Log to audit system
    audit_log.info(
        f"Sensitive request processed",
        extra={
            "user_id": context.user_id,
            "request_type": request_type,
            "session_id": context.session_id,
            "timestamp": datetime.utcnow().isoformat()
        }
    )
    
    # Process the request
    result = await process_request(request_type, details)
    
    return p.ToolResult(result)
```

## Troubleshooting

### Issue: Connection Refused

**Solution**: Check that the server is running and the port is accessible:

```bash
netstat -an | grep 8800
```

### Issue: Authentication Failed

**Solution**: Verify your credentials are correct:

```bash
# Test Azure credentials
az account show

# Test AWS credentials
aws sts get-caller-identity --region us-gov-west-1
```

### Issue: Slow Response Times

**Solution**: Increase resources or use caching:

```python
# Enable response caching
await agent.create_variable(
    name="cache_enabled",
    value="true"
)
```

## Next Steps

1. **Read the full integration guide**: [Aurelius Federal Platform Integration](./aurelius-federal-platform.md)
2. **Review security best practices**: [API Hardening](../production/api-hardening.md)
3. **Explore advanced features**: [Engine Extensions](../advanced/engine-extensions.md)
4. **Join the community**: [Discord Channel](https://discord.gg/duxWqxKk6J)

## Support

For Aurelius Platform-specific questions:
- **Platform Support**: platform-support@aurelius.gov
- **Technical Issues**: federal-support@parlant.io
- **Security Concerns**: security@parlant.io

## Compliance Checklist

Before deploying to production, ensure:

- [ ] Using approved LLM provider (Azure Gov Cloud or AWS GovCloud)
- [ ] Audit logging configured
- [ ] Authentication enabled
- [ ] HTTPS/TLS 1.3 enforced
- [ ] Data encryption at rest enabled
- [ ] Rate limiting configured
- [ ] Backup and recovery tested
- [ ] Security scan completed
- [ ] Documentation reviewed
- [ ] Compliance guidelines implemented

---

**Need Help?** Contact the Aurelius Platform team or visit the [Parlant Documentation](https://parlant.io/docs).
