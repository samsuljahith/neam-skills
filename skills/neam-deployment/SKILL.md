---
name: neam-deployment
description: Neam cloud deployment patterns — Docker, Kubernetes, serverless (AWS Lambda, GCP Cloud Run, Azure Functions), Helm, Terraform, and multi-cloud orchestration.
origin: neam-skills
---

# Neam Deployment Patterns

Patterns for compiling, packaging, and deploying Neam agents to cloud infrastructure — from local Docker containers to production Kubernetes clusters and serverless functions.

## When to Activate

- Deploying a Neam agent or application to any cloud target
- Configuring Kubernetes, Lambda, or Cloud Run deployments
- Setting up Docker containers for Neam agents
- Using multi-cloud routing and cost-aware deployment
- Generating Terraform or Helm artifacts from Neam

---

## Compilation Pipeline

Neam agents must be compiled before deployment.

```bash
# Step 1: Compile source to bytecode
neamc src/main.neam -o build/main.neamb

# Step 2: Test the binary locally
neam-cli build/main.neamb

# Step 3: Deploy to target
neam deploy --target <target> [options]
```

---

## Deployment Targets

| Target | Command | Use Case |
|--------|---------|----------|
| Docker | `--target docker` | Local dev, single-host containers |
| Kubernetes | `--target kubernetes` | Production clusters, auto-scaling |
| Helm | `--target helm` | K8s package management |
| AWS Lambda | `--target aws-lambda` | Serverless (AWS) |
| GCP Cloud Run | `--target gcp-cloudrun` | Serverless containers (GCP) |
| Azure Functions | `--target azure-functions` | Serverless (Azure) |
| Terraform | `--target terraform` | Infrastructure as Code |

---

## Pattern 1: Docker (Local / Dev)

Build and run a Neam agent in a container.

```bash
# Generate Docker artifacts
neam deploy --target docker

# Build the image
docker build -t my-agent -f build/deploy/docker/Dockerfile .

# Run with secrets injected via environment
docker run \
  -e OPENAI_API_KEY=$OPENAI_API_KEY \
  my-agent

# Or use docker-compose for multi-container setups
docker-compose -f build/deploy/docker/docker-compose.yml up
```

Generated artifacts in `build/deploy/docker/`:
- `Dockerfile` — multi-stage build producing a minimal runtime image
- `docker-compose.yml` — for local orchestration

---

## Pattern 2: Kubernetes (Production)

Deploy with auto-scaling, resource limits, and health probes.

```bash
# Generate K8s manifests
neam deploy --target kubernetes \
  --replicas 3 \
  --min-replicas 1 \
  --max-replicas 20 \
  --cpu 500m \
  --memory 1Gi \
  --namespace production

# Apply to your cluster
kubectl apply -f build/deploy/kubernetes/
```

Generated artifacts in `build/deploy/kubernetes/`:
- `deployment.yaml` — Deployment with replicas, resource limits, health probes
- `service.yaml` — ClusterIP service
- `configmap.yaml` — Environment configuration
- `ingress.yaml` — Nginx ingress routing

For GPU workloads:

```bash
neam deploy --target kubernetes \
  --gpu nvidia-t4 \
  --gpu-memory 16Gi \
  --replicas 2 \
  --namespace ml-workloads
```

---

## Pattern 3: AWS Lambda (Serverless)

Deploy as a lightweight serverless function. Great for bursty, event-driven workloads.

```neam
// src/handler.neam — keep the agent lean for Lambda cold starts
budget ServerlessBudget {
  time: 10000,      // 10 second timeout
  cost: 0.01,
  tokens: 2000
}

agent EventHandler {
  provider: "bedrock",
  model: "anthropic.claude-3-5-sonnet-20241022-v2:0",
  system: "Process events concisely. Return structured JSON.",
  budget: ServerlessBudget
}

let event = input();
let result = EventHandler.ask("Process this event: " + event);
emit result;
```

```bash
# Compile
neamc src/handler.neam -o build/handler.neamb

# Deploy to Lambda
neam deploy --target aws-lambda \
  --memory 512 \
  --timeout 15 \
  --arch arm64

# Generated: build/deploy/aws/template.yaml (AWS SAM)
sam deploy --template build/deploy/aws/template.yaml
```

---

## Pattern 4: GCP Cloud Run (Serverless Containers)

Best for stateless HTTP workloads with automatic scaling to zero.

```bash
neam deploy --target gcp-cloudrun \
  --region us-central1 \
  --concurrency 80 \
  --cpu-boost

# Generated: build/deploy/cloudrun/service.yaml
gcloud run services replace build/deploy/cloudrun/service.yaml \
  --region us-central1
```

---

## Pattern 5: Helm Chart (K8s Package Management)

Package your Neam agents as a Helm chart for versioned, repeatable deployments.

```bash
neam deploy --target helm

# Generated: build/deploy/helm/myapp/ (full Helm chart)
helm install my-agent build/deploy/helm/myapp/ \
  --namespace production \
  --set replicaCount=3 \
  --set image.tag=v1.2.0
```

---

## Pattern 6: Terraform (Infrastructure as Code)

Generate Terraform configs for repeatable, reviewable infrastructure.

```bash
neam deploy --target terraform

# Generated: build/deploy/terraform/main.tf
cd build/deploy/terraform/
terraform init
terraform plan
terraform apply
```

---

## Pattern 7: Environment Configuration per Target

Use Neam `env` blocks to configure agents per deployment environment.

```neam
env Development {
  API_URL: "http://localhost:8000",
  DEBUG: "true",
  LOG_LEVEL: "debug"
}

env Staging {
  API_URL: "https://api.staging.example.com",
  DEBUG: "false",
  LOG_LEVEL: "info"
}

env Production {
  API_URL: "https://api.example.com",
  DEBUG: "false",
  LOG_LEVEL: "warn",
  API_KEY: env("PROD_API_KEY")    // always read secrets from env vars
}

agent MyAgent {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Production agent",
  env: Production    // swap this per environment
}
```

---

## Pattern 8: Multi-Cloud Routing

Neam's `CostAwareRouter` automatically routes workloads to the cheapest viable cloud provider.

**How it works:**
1. Polls spot prices across AWS, GCP, Azure, Alibaba
2. Evaluates current latency from your region
3. Enforces data residency constraints
4. Routes to the cheapest viable option
5. Fails over automatically after 3 consecutive errors (60s cooldown)

```neam
// Multi-cloud pipeline — each agent can run on the cheapest provider
budget ResearchBudget {
  cost: 25.00,
  tokens: 500000
}

agent Planner {
  provider: "bedrock",     // AWS
  model: "anthropic.claude-3-5-sonnet-20241022-v2:0",
  budget: ResearchBudget
}

agent Executor {
  provider: "openai",      // OpenAI
  model: "gpt-4o"
}

agent Synthesizer {
  provider: "ollama",      // Free local
  model: "qwen2.5:14b"
}
```

Cloud provider capabilities:

| Feature | AWS | GCP | Azure | Alibaba |
|---------|-----|-----|-------|---------|
| Serverless | Lambda | Cloud Run | Functions | Function Compute |
| Kubernetes | EKS | GKE | AKS | ACK |
| Managed LLM | Bedrock | Vertex AI | Azure OpenAI | PAI |
| GPU | g4dn/g5/p4d | T4/L4/A100 | NC4/NC24 | GN6i/GN7i |
| Spot Instances | Yes | Yes | Yes | Yes |

---

## Pattern 9: Auto-Scaling Configuration

Neam uses ML-based predictive scaling with time-series forecasting.

```bash
# Kubernetes with auto-scaling
neam deploy --target kubernetes \
  --min-replicas 1 \
  --max-replicas 50 \
  --cpu 250m \
  --memory 512Mi

# Scaling triggers (automatic):
# - Scale up: load forecast exceeds capacity
# - Scale down: sustained low utilization
# - Switch to spot: spot price < 70% of on-demand
# - Switch to on-demand: spot interruption detected
# - Switch to FaaS: bursty, unpredictable traffic
```

---

## Generated Artifact Structure

```
build/deploy/
  kubernetes/
    deployment.yaml        # Deployment spec
    service.yaml           # ClusterIP service
    configmap.yaml         # Environment vars
    ingress.yaml           # Nginx ingress
  cloudrun/
    service.yaml           # GCP Knative service
  aws/
    template.yaml          # AWS SAM template
  helm/
    myapp/                 # Full Helm chart
      Chart.yaml
      values.yaml
      templates/
  docker/
    Dockerfile
    docker-compose.yml
  terraform/
    main.tf
```

---

## Deployment Checklist

- Compile source to bytecode before deploying: `neamc src/main.neam -o build/main.neamb`
- Test locally with `neam-cli build/main.neamb`
- Set all secrets as environment variables — never hardcode
- Attach `budget` to every production agent
- Use `env` blocks per environment (dev/staging/prod)
- Set resource limits (`--cpu`, `--memory`) for K8s deployments
- Use ARM64 (`--arch arm64`) for Lambda — cheaper and faster
- Run `kubectl apply --dry-run=client` before live K8s apply
- Tag images with version: `docker build -t my-agent:v1.2.0`

---

## Anti-Patterns

```bash
# Bad: Deploying with debug mode on in production
neam deploy --target kubernetes --debug true

# Bad: Unlimited replicas with no resource limits
neam deploy --target kubernetes --max-replicas 1000

# Good: Bounded, resource-constrained deployment
neam deploy --target kubernetes \
  --min-replicas 1 \
  --max-replicas 20 \
  --cpu 500m \
  --memory 1Gi \
  --namespace production
```

```neam
// Bad: Production agent with no budget — unlimited cost exposure
agent ProdAgent {
  provider: "openai",
  model: "gpt-4o"
}

// Good: Always cap cost and tokens
budget ProdBudget { cost: 100.00, tokens: 2000000 }
agent ProdAgent {
  provider: "openai",
  model: "gpt-4o",
  budget: ProdBudget,
  env: Production
}
```

__Remember__: `neam deploy` generates the artifacts — you still apply them. Always test with `--dry-run` before live deploys. Use the cheapest provider that meets your latency and data residency requirements. Arm64 Lambda is typically cheapest for serverless workloads.
