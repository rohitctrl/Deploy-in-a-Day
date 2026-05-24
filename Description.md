# Deploy in a Day - AI Agent Stack

## Overview

Deploy in a Day is a pre-configured Docker-based AI agent stack consisting of three interconnected services: OpenClaw (gateway), Hermes (task engine), and Paperclip (orchestrator/dashboard). This stack enables rapid deployment of a personal AI assistant system with capabilities for messaging integration, task automation, and persistent memory.

## Architecture

The system follows a microservices architecture where each component has a specific responsibility:

```
External Services (WhatsApp, Telegram, etc.)
            ↓
    [OpenClaw Gateway] ←→ [Hermes Agent] ←→ [Paperclip Dashboard]
            ↓                           ↑
    External Webhooks               Internal API
            ↓                           ↑
    User Interactions          Orchestration & UI
```

### Service Details

#### 1. OpenClaw (Gateway & Personal Assistant)
- **Image**: `ghcr.io/openclaw/openclaw:latest`
- **Port**: 8080 (exposed externally)
- **Purpose**: Acts as the entry point for external communications (webhooks from messaging platforms like WhatsApp, Telegram)
- **Key Configuration**:
  - `GATEWAY_BIND=0.0.0.0` - Binds to all interfaces for proper firewall/VPC routing
  - Connects to LLM provider for natural language processing
  - Persists data to `./data/openclaw`

#### 2. Hermes Agent (Self-Improving Task Engine)
- **Image**: `nousresearch/hermes-agent:latest`
- **Port**: 8000 (internal only)
- **Purpose**: Core task execution engine with self-improving capabilities and layered memory architecture
- **Key Features**:
  - **3-Layer Memory Architecture**:
    - Working Memory (`./data/hermes/working`) - Short-term context
    - Semantic Memory (`./data/hermes/semantic`) - Long-term facts & knowledge
    - Episodic Memory (`./data/hermes/episodic`) - Experience & interaction history
  - **Optimizations**:
    - `PRELOAD_MEMORY_VECTORS=true` - Prevents ARM cold-start crashes
    - Connects to same LLM provider as OpenClaw
  - Communicates internally with Paperclip and OpenClaw

#### 3. Paperclip AI (Orchestrator & Dashboard)
- **Image**: `reeoss/paperclipai-paperclip:latest`
- **Port**: 3000 (exposed for dashboard access)
- **Purpose**: Visual orchestrator and React-based dashboard for monitoring and managing the agent system
- **Key Features**:
  - Production mode (`NODE_ENV=production`)
  - Internal service wiring:
    - `OPENCLAW_ENDPOINT=http://openclaw:8080`
    - `HERMES_ENDPOINT=http://hermes:8000`
  - Admin protection via `ADMIN_PASSWORD` from environment
  - Persists dashboard data to `./data/paperclip`

## Configuration

### Environment Variables (.env)
All services share common LLM configuration via environment variables:

```
# AI Model Configuration
LLM_PROVIDER=openai
LLM_API_KEY=sk-[your-key-here]
OPENAI_BASE_URL=https://opencode.ai/zen/v1
LLM_MODEL=qwen3.6-plus-free  # Alternative: mimo-v2-omni-free

# Security
DASHBOARD_PASSWORD=[secure-password]
WEBHOOK_SECRET=[random-string]
```

### Networking
All services communicate via a Docker bridge network named `agent_swarm`:
- OpenClaw: `http://openclaw:8080` (internal) / `localhost:8080` (external)
- Hermes: `http://hermes:8000` (internal only)
- Paperclip: `http://paperclip:3000` (internal) / `localhost:3000` (external)

## Data Persistence
Each service maintains persistent storage in the `./data` directory:
- `./data/openclaw/` - Gateway state and webhook configurations
- `./data/hermes/` - Three-layer memory system (working/semantic/episodic)
- `./data/paperclip/` - Dashboard database and user preferences

## Deployment Instructions

### Prerequisites
- Docker Engine
- Docker Compose v2
- GitHub CLI (for repository operations, optional)
- Valid LLM API key (OpenAI-compatible endpoint)

### Setup
1. Clone or download this repository
2. Copy `.env.example` to `.env` and configure:
   - LLM API key and model preferences
   - Secure dashboard password
   - Webhook secret (for external service validation)
3. Start the stack:
   ```bash
   docker-compose up -d
   ```
4. Access services:
   - Dashboard: http://localhost:3000
   - OpenClaw Webhooks: http://localhost:8080/webhooks/[service]
   - Hermes API: http://localhost:8000 (internal tools only)

### Management
- View logs: `docker-compose logs -f [service-name]`
- Stop stack: `docker-compose down`
- Reset data: Remove contents of `./data/` directories (services will recreate structure)
- Update images: `docker-compose pull && docker-compose up -d`

## Use Cases

1. **Personal AI Assistant**: Connect WhatsApp/Telegram to interact with AI via messaging
2. **Task Automation**: Hermes can execute and learn from recurring tasks
3. **Knowledge Base**: Semantic memory accumulates facts from interactions
4. **Contextual Conversations**: Working memory maintains conversation context
5. **Experience Learning**: Episodic memory improves future responses based on past interactions

## Security Notes
- Change default passwords/secrets in `.env` before production use
- The webhook secret should be shared with external services validating webhook signatures
- Consider adding reverse proxy (NGINX/TRAEFIK) for SSL termination in production
- Regularly backup `./data/` directories to prevent memory loss

## Extensibility
- Add additional services to the `agent_swarm` network
- Extend OpenClaw with custom webhook handlers
- Enhance Hermes with custom skills/plugins
- Customize Paperclip dashboard via extensions

---
*This stack is designed for rapid deployment and experimentation with autonomous agent systems. For production use, consider additional security hardening, monitoring, and scaling considerations.*