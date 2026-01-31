# 🎙️ Voice Travel Agent

An AI-powered voice travel planning assistant that creates personalized itineraries through natural voice conversations. Built with Claude AI, ElevenLabs, and LangGraph.

![Voice Travel Agent](https://img.shields.io/badge/Python-3.11-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.128-green)
![Claude](https://img.shields.io/badge/AI-Claude%20Sonnet-purple)

---

## ✨ Features

### 🗣️ **Voice Interaction**
- Real-time voice-to-voice conversation
- Natural language understanding
- ElevenLabs TTS for human-like responses
- Live transcript display

### 🗺️ **Intelligent Itinerary Planning**
- Multi-day trip planning
- POI search integration
- Weather information
- RAG-powered travel tips from Wikivoyage
- Customizable based on preferences

### 📧 **Email Delivery**
- Beautiful HTML email templates
- Send itineraries to any email
- Responsive email design

### 🎨 **Modern UI**
- Clean, responsive interface
- Real-time conversation display
- Side-by-side itinerary view
- Mobile-friendly design

---

## 🔍 AI Evaluations

The Voice Travel Agent includes three comprehensive AI evaluation systems that automatically validate itinerary quality:

### 1. **Feasibility Evaluation** ✅
Ensures itineraries are practical and achievable:
- **Daily Duration ≤ Available Time**: Activities fit within time windows (Morning: 3h, Afternoon: 3h, Evening: 4h)
- **Reasonable Travel Times**: Validates travel between activities (<60 minutes recommended)
- **Pace Consistency**: Balanced activity distribution (ideal: ≤6 activities/day)

### 2. **Edit Correctness Evaluation** ✅
Ensures voice edits are accurate:
- **Intended Changes Only**: Edits modify only requested sections
- **No Unintended Changes**: Detects modifications outside edit scope
- **Smart Detection**: Infers intended sections from natural language instructions

### 3. **Grounding & Hallucination Evaluation** ✅
Verifies information authenticity:
- **POI Grounding**: ≥70% of POIs match search results (Foursquare/OpenStreetMap)
- **Source Citations**: ≥50% of tips cite RAG sources `[Source: Wikivoyage...]`
- **Explicit Uncertainty**: Validates uncertainty markers for missing data

### How It Works

Evaluations run automatically in the background whenever itineraries are created or edited:

```
Agent Creates Itinerary
    ↓
Evaluations Run in Background
    ├─ Feasibility (duration, travel, pace)
    ├─ Grounding (POIs, citations, uncertainty)
    └─ Edit Correctness (if editing)
    ↓
Results Saved to evaluation_results.json
```

### Usage

```python
from app.evals import EvaluationRunner

runner = EvaluationRunner()

# Run all evaluations
results = runner.run_all_evals(
    itinerary_text=itinerary,
    context={
        "search_results": search_results,
        "travel_times": travel_times
    }
)

# Check results
if results["overall"]["all_passed"]:
    print("✓ All evaluations passed!")
```

### View Results

```bash
# Check latest evaluation results
cat evaluation_results.json | python -m json.tool

# Run standalone test
python test_evals.py
```

**See [app/evals/README.md](app/evals/README.md) for complete documentation.**

---

## 🏗️ Architecture

### System Overview

The Voice Travel Agent is built on a modular architecture with the **LangGraph Agent** at its core, orchestrating multiple specialized services through the **MCP (Model Context Protocol)** framework.

```
┌────────────────────────────────────────────────────────────────────────────┐
│                           USER INTERACTION LAYER                           │
└────────────────────────────────────────────────────────────────────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    │                                   │
         ┌──────────▼──────────┐           ┌───────────▼──────────┐
         │   Browser (WebUI)   │           │   CLI Interface      │
         │   - WebSocket       │           │   - Direct REPL      │
         │   - Real-time UI    │           │   - Text-based       │
         └──────────┬──────────┘           └───────────┬──────────┘
                    │                                   │
                    └─────────────────┬─────────────────┘
                                      │
┌────────────────────────────────────────────────────────────────────────────┐
│                          VOICE PROCESSING LAYER                            │
│                                                                            │
│   ┌─────────────────────┐                    ┌─────────────────────┐     │
│   │   STT Service       │                    │   TTS Service       │     │
│   │   (ElevenLabs)      │                    │   (ElevenLabs)      │     │
│   │                     │                    │                     │     │
│   │  Audio → Text       │                    │  Text → Audio       │     │
│   └──────────┬──────────┘                    └──────────▲──────────┘     │
│              │                                           │                │
└──────────────┼───────────────────────────────────────────┼────────────────┘
               │                                           │
               │  Transcribed Text                         │  Response Text
               │                                           │
┌──────────────▼───────────────────────────────────────────┴────────────────┐
│                         FASTAPI SERVER LAYER                              │
│                                                                            │
│   ┌────────────────────────────────────────────────────────────────┐     │
│   │              WebSocket Handler / Session Manager              │     │
│   │  - Manages voice sessions                                      │     │
│   │  - Routes audio/text between voice services and agent          │     │
│   │  - Handles email delivery                                      │     │
│   └────────────────────────────┬───────────────────────────────────┘     │
│                                │                                          │
└────────────────────────────────┼──────────────────────────────────────────┘
                                 │
                                 │  User Messages
                                 │
┌────────────────────────────────▼──────────────────────────────────────────┐
│                          AGENT ORCHESTRATION LAYER                        │
│                                                                            │
│   ┌────────────────────────────────────────────────────────────────┐     │
│   │                    Agent Factory                               │     │
│   │  - Singleton pattern for agent lifecycle                       │     │
│   │  - Initializes MCP clients and tools                           │     │
│   │  - Manages checkpointer for conversation persistence           │     │
│   └────────────────────────────┬───────────────────────────────────┘     │
│                                │                                          │
│   ┌────────────────────────────▼───────────────────────────────────┐     │
│   │              Evaluated Agent Wrapper                           │     │
│   │  - Wraps base agent with evaluation capabilities               │     │
│   │  - Tracks tool calls and search results                        │     │
│   │  - Triggers evaluations on itinerary generation                │     │
│   └────────────────────────────┬───────────────────────────────────┘     │
│                                │                                          │
│   ┌────────────────────────────▼───────────────────────────────────┐     │
│   │              LangGraph Agent (Core)                            │     │
│   │                                                                 │     │
│   │  ┌─────────────────────────────────────────────────────┐       │     │
│   │  │  State Graph with Memory (MemorySaver)             │       │     │
│   │  │  - Multi-turn conversation management              │       │     │
│   │  │  - Phase-aware prompting (Clarifying/Planning)     │       │     │
│   │  │  - Tool orchestration with parallel execution      │       │     │
│   │  └─────────────────────────────────────────────────────┘       │     │
│   │                                                                 │     │
│   │  Uses: ChatGroq (qwen/qwen3-32b) with tool binding             │     │
│   └────────────────────────────┬───────────────────────────────────┘     │
│                                │                                          │
└────────────────────────────────┼──────────────────────────────────────────┘
                                 │
                    Tool Calls & Results
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        │                        │                        │
┌───────▼────────┐    ┌──────────▼──────────┐    ┌──────▼───────────┐
│  MCP Server    │    │   MCP Server        │    │   MCP Server     │
│  (POI Search)  │    │   (Itinerary)       │    │   (Weather)      │
│                │    │                     │    │                  │
│  Tools:        │    │  Tools:             │    │  Tools:          │
│  - search_     │    │  - estimate_        │    │  - get_forecast  │
│    places      │    │    travel_time      │    │                  │
│                │    │                     │    │  External API:   │
│  External:     │    │  External API:      │    │  - Open-Meteo    │
│  - Foursquare  │    │  - OSRM Routing     │    │                  │
│  - OpenStreet  │    │                     │    │                  │
│    Map         │    │                     │    │                  │
└────────────────┘    └─────────────────────┘    └──────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
           ┌────────▼────────┐       ┌────────▼─────────┐
           │  RAG Tool       │       │  Evaluation      │
           │  (Wikivoyage)   │       │  System          │
           │                 │       │                  │
           │  Components:    │       │  Evaluators:     │
           │  - Sentence     │       │  - Feasibility   │
           │    Transformer  │       │  - Grounding     │
           │  - Pinecone     │       │  - Edit          │
           │    Vector DB    │       │    Correctness   │
           │  - Lazy Loading │       │                  │
           │                 │       │  Output:         │
           │  Returns:       │       │  - evaluation_   │
           │  - Travel tips  │       │    results.json  │
           │  - Cultural     │       │                  │
           │    context      │       │                  │
           │  - Safety info  │       │                  │
           └─────────────────┘       └──────────────────┘
```

### Component Details

#### 1. **Central Agent (LangGraph)**
- **Location**: `app/agent/graph.py`
- **Purpose**: Orchestrates the entire conversation flow and tool execution
- **Key Features**:
  - State-based workflow management
  - Multi-turn conversation with memory persistence
  - Parallel tool execution (disabled for reduced iterations)
  - Phase-aware prompting (Clarifying → Planning → Reviewing)
  - Tool binding with ChatGroq LLM

#### 2. **MCP Servers (Model Context Protocol)**
MCP servers run as separate processes, communicating via stdio:

**POI Search Server** (`app/mcp_servers/poi_search.py`)
- Searches for points of interest using Foursquare and OpenStreetMap
- Geocodes cities using Nominatim
- Returns structured POI data (name, rating, location)

**Itinerary Server** (`app/mcp_servers/itinerary.py`)
- Estimates travel time between coordinates
- Uses OSRM (Open Source Routing Machine)
- Supports driving and walking modes

**Weather Server** (`app/mcp_servers/weather.py`)
- Fetches weather forecasts
- Uses Open-Meteo API
- Provides temperature, conditions, precipitation

#### 3. **RAG System (Retrieval-Augmented Generation)**
- **Location**: `app/rag/`
- **Purpose**: Provides contextual travel information
- **Components**:
  - **Embeddings**: Sentence Transformers (all-MiniLM-L6-v2)
  - **Vector Store**: Pinecone for similarity search
  - **Data Source**: Wikivoyage travel guides
  - **Lazy Loading**: Models load on first use to speed up server startup
- **Returns**: Relevant travel tips, cultural context, safety information

#### 4. **Voice Services**
**STT (Speech-to-Text)** (`app/voice/stt_service.py`)
- ElevenLabs Conversational AI API
- Converts audio chunks to text in real-time
- Handles audio buffering and streaming

**TTS (Text-to-Speech)** (`app/voice/tts_service.py`)
- ElevenLabs TTS API
- Converts agent responses to natural speech
- Streams audio back to client
- Configurable voice (default: Bella - warm, friendly)

#### 5. **Evaluation System**
- **Location**: `app/evals/`
- **Purpose**: Quality assurance for generated itineraries
- **Triggers**: Automatically runs after itinerary creation/editing
- **Components**:
  - **Feasibility Evaluator**: Validates time constraints and pacing
  - **Grounding Evaluator**: Checks POI authenticity and source citations
  - **Edit Correctness Evaluator**: Ensures edits only modify intended sections
- **Output**: `evaluation_results.json` with detailed pass/fail results

### Data Flow

#### Itinerary Generation Flow
```
1. User speaks → STT converts to text
2. Text sent to Agent via WebSocket
3. Agent (LangGraph) processes request:
   a. Determines conversation phase
   b. Calls tools in parallel:
      - search_places (POI Server)
      - retrieve_travel_guides (RAG)
      - get_forecast (Weather Server)
   c. Synthesizes results into itinerary
4. Evaluation system validates itinerary
5. Response text → TTS converts to audio
6. Audio streamed back to user
7. Results saved to evaluation_results.json
```

#### Tool Execution Flow
```
Agent → MCP Client Manager → MCP Server (stdio) → External API
                                         ↓
                          Results ← Tool Response
```

### Key Design Patterns

1. **Singleton Pattern**: AgentFactory ensures single agent instance
2. **Wrapper Pattern**: EvaluatedAgentWrapper adds evaluations without modifying core
3. **Factory Pattern**: MCP servers created and managed by MCPClientManager
4. **Lazy Loading**: RAG models and embeddings load on first use
5. **Parallel Execution**: MCP servers connect in parallel for faster startup
6. **State Management**: LangGraph with MemorySaver for conversation persistence

---

## 🚀 Quick Start

### Prerequisites

- Python 3.11+
- API Keys (see [Setup](#setup))

### Local Development

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd voice_agent
   ```

2. **Create virtual environment**
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your API keys
   ```

5. **Run the server**
   ```bash
   python -m uvicorn app.server:app --host 0.0.0.0 --port 8001 --reload
   ```

6. **Open in browser**
   ```
   http://localhost:8001
   ```

---

## 🔑 Setup

### Required API Keys

Create a `.env` file with the following:

```env
# Required
GROQ_API_KEY=gsk_...              # Get from: https://console.groq.com
ELEVENLABS_API_KEY=sk_...         # Get from: https://elevenlabs.io
RESEND_API_KEY=re_...             # Get from: https://resend.com

# Optional (for enhanced features)
PINECONE_API_KEY=...              # For RAG travel guides
FOURSQUARE_SERVICE_API_KEY=...   # For POI search
```

### API Key Setup Guide

#### 1. **Groq (LLM - Required)**
   - Visit [console.groq.com](https://console.groq.com)
   - Sign up for free account
   - Create API key
   - Free tier: 30 requests/minute

#### 2. **ElevenLabs (Voice - Required)**
   - Visit [elevenlabs.io](https://elevenlabs.io)
   - Sign up for free account
   - Get API key from settings
   - Free tier: 10,000 characters/month

#### 3. **Resend (Email - Required for email feature)**
   - Visit [resend.com](https://resend.com)
   - Sign up (no credit card needed)
   - Create API key
   - Free tier: 3,000 emails/month

#### 4. **Pinecone (Optional - for RAG)**
   - Visit [pinecone.io](https://pinecone.io)
   - Create free account
   - Create index named `travel-agent-rag`
   - Get API key

---

## 🐳 Docker Deployment

### Build and Run Locally

```bash
# Build Docker image
docker build -t voice-agent .

# Run container
docker run -p 8001:8001 \
  -e GROQ_API_KEY="your_key" \
  -e ELEVENLABS_API_KEY="your_key" \
  -e RESEND_API_KEY="your_key" \
  voice-agent
```

### Docker Compose

```yaml
version: '3.8'
services:
  voice-agent:
    build: .
    ports:
      - "8001:8001"
    environment:
      - GROQ_API_KEY=${GROQ_API_KEY}
      - ELEVENLABS_API_KEY=${ELEVENLABS_API_KEY}
      - RESEND_API_KEY=${RESEND_API_KEY}
    volumes:
      - model-cache:/app/models

volumes:
  model-cache:
```

---

## 🌐 Production Deployment

### Render.com (Recommended)

1. **Push to GitHub**
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

2. **Deploy to Render**
   - Go to [render.com](https://render.com)
   - Click "New +" → "Web Service"
   - Connect GitHub repository
   - Render auto-detects `render.yaml`
   - Add environment variables in dashboard
   - Deploy!

3. **Set Environment Variables in Render Dashboard**
   - `GROQ_API_KEY`
   - `ELEVENLABS_API_KEY`
   - `RESEND_API_KEY`
   - `PINECONE_API_KEY` (optional)

**Estimated Cost:** $7-25/month (or free tier for testing)

### Alternative Platforms

- **Railway.app** - Similar to Render, $10-15/month
- **Fly.io** - Global edge deployment, ~$15/month
- **DigitalOcean App Platform** - $12/month

---

## 📊 Performance

### Startup Time

| Environment | Startup Time |
|-------------|--------------|
| Local (cold) | ~19 seconds |
| Local (warm) | ~19 seconds |
| Docker (first build) | ~5-10 minutes |
| Docker (cached) | ~4 seconds ⚡ |
| Production (Render) | ~4 seconds ⚡ |

### Optimizations

- ✅ Parallel MCP server connections
- ✅ Lazy-loaded RAG model
- ✅ Pre-cached models in Docker
- ✅ Efficient WebSocket handling

---

## 📖 Usage

### Voice Workflow

1. **Click microphone** to start recording
2. **Speak your request**: "I want to plan a trip to Paris"
3. **Answer questions**: Agent asks about duration, dates, interests
4. **Get itinerary**: Agent generates personalized plan
5. **Email it**: Click email button to send to your inbox

### Example Conversation

```
You: "I want to plan a trip to Japan"

Agent: "Great! How many days are you planning to spend in Japan?"

You: "Five days"

Agent: "When are you planning to travel?"

You: "Next month"

Agent: "What are your main interests - culture, food, adventure, or relaxation?"

You: "Food and culture"

Agent: "Perfect! I have everything I need. Let me create a personalized
       itinerary for you."

[Agent generates 5-day itinerary with POI, weather, and travel tips]
```

---

## 🛠️ API Endpoints

### Health & Readiness

```bash
# Basic health check
GET /api/health

# Readiness check (for load balancers)
GET /api/ready
```

### Email Itinerary

```bash
POST /api/send-itinerary
Content-Type: application/json

{
  "email": "user@example.com",
  "destination": "Paris, France",
  "itinerary_content": "# Day 1..."
}
```

### WebSocket

```bash
WS /ws/voice

# Messages:
# - audio_chunk: Send audio data
# - stop_recording: End recording
# - interrupt: Cancel current processing
```

---

## 🧪 Development

### Project Structure

```
voice_agent/
├── app/
│   ├── agent/              # LangGraph agent & MCP tools
│   │   ├── factory.py      # Agent factory (parallelized)
│   │   ├── graph.py        # LangGraph workflow
│   │   ├── evaluated_agent.py  # Evaluation wrapper
│   │   └── prompts.py      # Phase-specific prompts
│   ├── evals/              # AI Evaluation System
│   │   ├── feasibility.py  # Feasibility checks
│   │   ├── edit_correctness.py  # Edit validation
│   │   ├── grounding.py    # Grounding & hallucination checks
│   │   ├── runner.py       # Evaluation orchestrator
│   │   └── README.md       # Full documentation
│   ├── mcp_servers/        # MCP tool servers
│   │   ├── poi_search.py
│   │   ├── weather.py
│   │   └── itinerary.py
│   ├── rag/                # Retrieval-Augmented Generation
│   │   ├── retrieve.py     # Lazy-loaded embeddings
│   │   └── client.py       # Pinecone client
│   ├── voice/              # Voice services
│   │   ├── stt_service.py
│   │   ├── tts_service.py
│   │   └── websocket_handler.py
│   ├── static/             # Frontend
│   │   ├── index.html
│   │   ├── app.js
│   │   └── styles.css
│   └── server.py           # FastAPI server
├── test_evals.py           # Evaluation tests
├── test_integration.py     # Integration tests
├── evaluation_results.json # Auto-generated evaluation results
├── Dockerfile
├── requirements.txt
├── render.yaml
└── .env
```

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=app

# Run specific test
pytest tests/test_agent.py
```

---

## 🔧 Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GROQ_API_KEY` | Yes | - | Groq API key for LLM |
| `ELEVENLABS_API_KEY` | Yes | - | ElevenLabs for voice |
| `RESEND_API_KEY` | Yes | - | Resend for email |
| `PINECONE_API_KEY` | No | - | Pinecone for RAG |
| `SERVER_PORT` | No | 8000 | Server port |
| `LOG_LEVEL` | No | INFO | Logging level |

### Model Settings

```python
# Voice settings
ELEVENLABS_VOICE_ID = "EXAVITQu4vr4xnSDxMaL"  # Bella voice
ELEVENLABS_MODEL_ID = "eleven_turbo_v2_5"     # Fast model

# LLM settings
MODEL = "llama-3.3-70b-versatile"  # via Groq

# Embedding model (cached)
EMBEDDING_MODEL = "all-MiniLM-L6-v2"  # 384 dimensions
```

---

## 🐛 Troubleshooting

### Common Issues

**1. Server won't start**
```bash
# Check if port is in use
netstat -ano | findstr :8001  # Windows
lsof -i :8001                 # Mac/Linux

# Kill process and restart
```

**2. Voice not working**
- Check microphone permissions in browser
- Ensure HTTPS or localhost (required for WebRTC)
- Check ElevenLabs API quota

**3. Slow startup**
- First run downloads model (~90MB)
- Subsequent runs use cache (4s startup)
- Use Docker for production (pre-cached)

**4. Email not sending**
- Verify Resend API key
- Check email address format
- Review Resend dashboard for errors

---

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- **Claude AI** by Anthropic - LLM and agent framework
- **ElevenLabs** - Natural voice synthesis
- **Groq** - Fast LLM inference
- **LangGraph** - Agent orchestration
- **Pinecone** - Vector database for RAG
- **Resend** - Email delivery

---

## 📞 Support

For issues, questions, or suggestions:
- Open an issue on GitHub
- Check [documentation](docs/)
- Review [FAQ](docs/FAQ.md)

---

## 🗺️ Roadmap

- [ ] Multi-language support
- [ ] Voice clone customization
- [ ] Mobile app (React Native)
- [ ] Booking integrations
- [ ] Multi-user sessions
- [ ] Advanced analytics

---

**Built with ❤️ using Claude, ElevenLabs, and LangGraph**

⭐ Star this repo if you find it helpful!
