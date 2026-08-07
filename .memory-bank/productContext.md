# Product Context

## Why This Project Exists
Open Notebook was created to provide a privacy-focused, self-hostable alternative to cloud-based AI research assistants like Google Notebook LM. Users want AI-powered note-taking and research capabilities without sacrificing data privacy or vendor lock-in.

## Problems It Solves
1. **Privacy Concerns**: Keep your research and notes under your control
2. **Vendor Lock-in**: Support multiple AI providers, not just one
3. **Cost**: Self-hosting reduces ongoing subscription costs
4. **Customization**: Open source allows customization for specific workflows
5. **Data Ownership**: All data stored locally in SurrealDB

## How It Should Work

### User Experience Goals
- **Simple Setup**: Docker Compose deployment should "just work"
- **Flexible AI**: Easy to switch between AI providers (Groq, OpenAI, Anthropic, local Ollama)
- **Fast Performance**: Quick note-taking, instant search, responsive UI
- **Rich Features**: Podcasts, insights, embeddings, graph views
- **Privacy by Default**: No data leaves the user's environment unless explicitly configured

### Key Workflows
1. **Note Creation**: User creates notes, AI provides insights
2. **Source Processing**: Upload documents, auto-extract and embed content
3. **Podcast Generation**: Convert notes to audio podcasts
4. **AI Chat**: Chat with notes and sources using preferred AI provider
5. **Graph Navigation**: Visualize connections between notes and insights

## Product Principles
1. Privacy-first architecture
2. Multi-provider AI support (no single vendor dependency)
3. Self-hostable and deployable
4. Open source and community-driven
5. Modular and extensible design

## Current Challenges
- Docker build failures on frontend (npm issues)
- SSL verification when behind corporate proxies (CloudFlare, etc.)
- Balancing feature richness with performance

## Target Users
- Researchers and students
- Knowledge workers
- Privacy-conscious individuals
- Developers wanting self-hosted AI tools
- Organizations needing data sovereignty
