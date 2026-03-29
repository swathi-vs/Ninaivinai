# Ninaivinai — Semantic Video Memory Search

> *"Search videos by meaning, not by timestamps."*

Ninaivinai (நினைவினை — Tamil for "memory") is a modern web application that transforms video content into a searchable semantic index. Upload a video, and the system automatically transcribes, chunks, and indexes it into a vector database. Then, simply describe a moment in natural language, and Ninaivinai will find and jump to the exact timestamp.

---

## Features

- **Video Upload** — Drag-and-drop or browse to upload video files (MP4, WebM)
- **Client-Side Audio Extraction** — Converts video to lightweight MP3 locally before uploading, drastically reducing transfer time
- **Automatic Transcription** — Audio is transcribed using OpenAI's Whisper model via the backend
- **Semantic Chunking** — Transcripts are split into overlapping segments with embeddings for better retrieval accuracy
- **Vector Database Indexing** — Chunks are stored in Qdrant with cosine similarity search
- **Natural Language Search** — Ask questions like *"When did we discuss the Q3 budget?"* and get precise results
- **Timeline Jump** — Click any search result to instantly seek the video player to the exact moment
- **Dark / Light Mode** — Toggle between warm dark and light themes with persistent preference
- **Real-Time Processing UI** — Beautiful progress bar with step-by-step status during indexing

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Framework** | React 19 + Vite 8 |
| **Routing** | React Router DOM v7 |
| **Icons** | Lucide React |
| **Audio Encoding** | lamejs (CDN) |
| **Styling** | Vanilla CSS with CSS Variables, Glassmorphism |
| **Backend** | FastAPI (Python) |
| **Vector DB** | Qdrant |
| **Transcription** | OpenAI Whisper |
| **Embeddings** | Sentence Transformers (1024-dim) |

---

## Project Structure

```
Ninaivinai/
├── index.html                      # Entry HTML with lamejs CDN
├── vite.config.js                  # Vite configuration
├── package.json                    # Dependencies and scripts
├── backend_temp_understand.py      # Backend API reference (FastAPI)
├── public/                         # Static assets
└── src/
    ├── main.jsx                    # React entry point
    ├── App.jsx                     # Root component with routing
    ├── App.css                     # Layout styles + animations
    ├── index.css                   # Design system (CSS variables, themes)
    ├── assets/                     # Images (hero, logos)
    ├── components/
    │   ├── NavigationBar.jsx       # Top navbar with theme toggle
    │   ├── NavigationBar.css
    │   ├── VideoPlayer.jsx         # HTML5 video with seekTo API
    │   ├── VideoPlayer.css
    │   ├── ChatInterface.jsx       # Semantic search chat UI
    │   └── ChatInterface.css
    ├── contexts/
    │   ├── ThemeContext.jsx         # Dark/Light mode provider
    │   └── AuthContext.jsx         # (Legacy — not active)
    ├── pages/
    │   ├── LandingPage.jsx         # Hero page with feature cards
    │   ├── LandingPage.css
    │   ├── Dashboard.jsx           # Video upload with drag-and-drop
    │   ├── Dashboard.css
    │   ├── Workspace.jsx           # Processing pipeline + player + chat
    │   ├── LoginPage.jsx           # (Legacy — not active)
    │   └── LoginPage.css
    ├── services/
    │   ├── api.js                  # All backend API calls
    │   └── mockApi.js              # (Legacy — mock data)
    └── utils/
        └── audioConverter.js       # Client-side video to MP3 converter
```

---

## Getting Started

### Prerequisites

- **Node.js** >= 18
- **npm** >= 9
- **Backend API** running (FastAPI + Qdrant) — see [Backend API](#backend-api)

### Installation

```bash
# Clone the repository
git clone https://github.com/swathi-vs/Ninaivinai.git
cd Ninaivinai

# Install dependencies
npm install

# Start the development server
npm run dev
```

The app will be available at `http://localhost:5173`

### Configuration

Update the `BASE_URL` in `src/services/api.js` to point to your backend:

```javascript
// For local backend
const BASE_URL = 'http://localhost:8000';

// For Ngrok tunnel
const BASE_URL = 'https://your-ngrok-url.ngrok-free.dev';
```

---

## Application Workflow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│   Landing    │────>│  Dashboard   │────>│    Workspace      │
│    Page      │     │  (Upload)    │     │  (Process + Chat) │
└──────────────┘     └──────────────┘     └──────────────────┘
```

### Processing Pipeline (Workspace)

When a video is uploaded, the Workspace automatically runs a 4-step pipeline:

| Step | Progress | Action | API Endpoint |
|------|----------|--------|-------------|
| 1 | 15% | **Extract MP3 locally** — Rips audio from video in the browser | *Client-side* |
| 2 | 35% | **Transcribe Audio** — Sends MP3 to Whisper model | `POST /transcribe` |
| 3 | 65% | **Semantic Chunking** — Splits transcript into overlapping segments with embeddings | `POST /add_overlap` |
| 4 | 90% | **Build Vector Index** — Creates Qdrant collection and inserts chunks | `POST /create_collection` + `POST /add` |

Once complete (100%), the video player and semantic chat interface are revealed.

### Search Flow

```
User Query -> POST /embedding -> POST /search -> Best Sentence Match -> Jump to Timestamp
```

---

## Backend API

The frontend communicates with a **FastAPI** backend. All endpoints are documented in `backend_temp_understand.py`.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Health check |
| `/transcribe` | POST | Upload audio file, returns transcript JSON |
| `/embedding` | POST | Text to 1024-dim sentence embedding |
| `/add_overlap` | POST | Transcript to semantically chunked segments with embeddings |
| `/create_collection` | POST | Create a Qdrant vector collection |
| `/add` | POST | Insert embedding chunks into a collection |
| `/search` | POST | Semantic vector search, returns best sentence match |
| `/search_with_filter` | POST | Filtered semantic search |

### Backend Requirements

- Python 3.10+
- FastAPI + Uvicorn
- Qdrant (running on `localhost:6333`)
- Whisper (for transcription)
- Sentence Transformers (for embeddings)

```bash
# Start Qdrant
docker run -p 6333:6333 qdrant/qdrant

# Start the backend
python backend_temp_understand.py
# Runs on http://0.0.0.0:8000
```

---

## Design System

The UI uses a **warm terracotta** color palette with glassmorphism effects:

| Token | Dark Mode | Light Mode |
|-------|-----------|------------|
| `--bg-color` | `#1a1716` (warm charcoal) | `#faf6f0` (warm cream) |
| `--accent-color` | `#eb5e28` (terracotta) | `#eb5e28` (terracotta) |
| `--accent-gradient` | `#eb5e28 to #f4a261` | `#eb5e28 to #f4a261` |
| `--panel-bg` | `rgba(38,33,30,0.7)` | `rgba(255,255,255,0.65)` |

Key design features:
- **Glassmorphism panels** with `backdrop-filter: blur(12px)`
- **Smooth theme transitions** with `0.4s ease`
- **Custom scrollbars** for a polished look
- **Radial background gradients** for depth
- **Font**: Inter (system fallback stack)

---

## Available Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server with HMR |
| `npm run build` | Build for production |
| `npm run preview` | Preview production build locally |
| `npm run lint` | Run ESLint |

---

## Contributing

1. Fork the repository
2. Create your feature branch: `git checkout -b feat/your-feature`
3. Commit your changes: `git commit -m 'feat: add your feature'`
4. Push to the branch: `git push origin feat/your-feature`
5. Open a Pull Request

---

## License

This project is part of the **KaatchiMei** RAG pipeline.
