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

## 🏗️ Architecture

```
┌─────────────┐
│   Browser   │
│  (WebSocket)│
└──────┬──────┘
       │
┌──────▼──────────────────┐
│   FastAPI Server        │
│   - WebSocket Handler   │
│   - STT/TTS Services    │
│   - Email API           │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   LangGraph Agent       │
│   - Conversation Flow   │
│   - State Management    │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   MCP Tools (Parallel)  │
│   - POI Search          │
│   - Weather API         │
│   - Travel Guides (RAG) │
└─────────────────────────┘
```

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
│   ├── agent/           # LangGraph agent & MCP tools
│   │   ├── factory.py   # Agent factory (parallelized)
│   │   ├── graph.py     # LangGraph workflow
│   │   └── prompts.py   # Phase-specific prompts
│   ├── mcp_servers/     # MCP tool servers
│   │   ├── poi_search.py
│   │   ├── weather.py
│   │   └── itinerary.py
│   ├── rag/             # Retrieval-Augmented Generation
│   │   ├── retrieve.py  # Lazy-loaded embeddings
│   │   └── client.py    # Pinecone client
│   ├── voice/           # Voice services
│   │   ├── stt_service.py
│   │   ├── tts_service.py
│   │   └── websocket_handler.py
│   ├── static/          # Frontend
│   │   ├── index.html
│   │   ├── app.js
│   │   └── styles.css
│   └── server.py        # FastAPI server
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
