
# Chronos - Development Guide

## Project Structure

```
chronos-history-agent/
├── backend/                    # Python FastAPI application
│   ├── app/
│   │   ├── agents/           # AI agents (LangGraph-based)
│   │   ├── routers/          # API endpoints
│   │   ├── models/           # Database models
│   │   ├── core/             # Core utilities (DB, LLM, RAG)
│   │   ├── config.py         # Configuration management
│   │   └── main.py           # FastAPI app entry point
│   ├── requirements.txt       # Python dependencies
│   ├── Dockerfile
│   └── .env.example
├── frontend/                   # React + TypeScript application
│   ├── src/
│   │   ├── components/       # React components
│   │   ├── pages/            # Page components
│   │   ├── App.tsx           # Main app component
│   │   ├── main.tsx          # Entry point
│   │   └── index.css         # Global styles
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── Dockerfile
├── docker-compose.yml         # Full stack Docker Compose
└── README.md
```

## Quick Start

### Option 1: Docker Compose (Recommended)

```bash
# Build and start all services
docker-compose up

# Services available at:
# - Frontend: http://localhost:5173
# - Backend API: http://localhost:8000
# - API Docs: http://localhost:8000/docs
# - PostgreSQL: localhost:5432
# - Redis: localhost:6379
```

### Option 2: Local Development

#### Backend Setup

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create .env file
cp .env.example .env

# Start PostgreSQL and Redis (using Docker)
docker run -d --name postgres -e POSTGRES_PASSWORD=chronos_pwd -p 5432:5432 ankane/pgvector
docker run -d --name redis -p 6379:6379 redis:7-alpine

# Run migrations and start server
uvicorn app.main:app --reload
```

Backend will be available at `http://localhost:8000`

#### Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

Frontend will be available at `http://localhost:5173`

## API Endpoints

### Health Check
- `GET /health` - Service health status
- `GET /ready` - Readiness probe

### Query (Ask Questions)
- `POST /api/v1/query` - Process a history question
- `GET /api/v1/query/examples` - Get example queries

### Timelines
- `POST /api/v1/timelines/generate` - Generate timeline for a topic
- `POST /api/v1/timelines/compare` - Compare multiple timelines

### Analysis
- `POST /api/v1/analysis/compare` - Comparative analysis
- `POST /api/v1/analysis/counterfactual` - What-if analysis
- `POST /api/v1/analysis/causality` - Cause and consequence analysis

### Citations
- `GET /api/v1/citations/search` - Search citations
- `POST /api/v1/citations/verify` - Verify citation credibility
- `GET /api/v1/citations/topic/{topic}` - Get citations by topic

## Development Workflow

### Adding a New API Endpoint

1. **Create a router file** in `backend/app/routers/`
2. **Define Pydantic models** for request/response validation
3. **Implement route handlers**
4. **Include router** in `backend/app/main.py`

Example:
```python
# backend/app/routers/my_feature.py
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter(prefix="/api/v1", tags=["my_feature"])

class MyRequest(BaseModel):
    field: str

@router.post("/my-endpoint")
async def my_endpoint(request: MyRequest):
    return {"status": "success"}
```

### Adding Agent Functionality

1. **Create agent class** in `backend/app/agents/`
2. **Implement async methods** for different tasks
3. **Use LangGraph** for complex workflows
4. **Integrate with routers** via API endpoints

### Frontend Components

All React components are in `frontend/src/`:
- Pages go in `pages/` (routed via React Router)
- Reusable components go in `components/`
- Use TypeScript for type safety
- Tailwind CSS for styling

## Configuration

### Environment Variables

Backend configuration (`.env` file):

```env
DATABASE_URL=postgresql://user:password@localhost:5432/chronos_db
REDIS_URL=redis://localhost:6379
OPENAI_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here
ENVIRONMENT=development
DEBUG=true
```

## Database Management

### Create Migrations (using SQLAlchemy)

```bash
# In backend directory
alembic init migrations  # First time only
alembic revision --autogenerate -m "Add new table"
alembic upgrade head
```

### Database Models

All models are in `backend/app/models/history.py`:
- `HistoricalEvent` - Historical events with dates and locations
- `HistoricalFigure` - Historical figures with biographies
- `Citation` - Sources and citations with credibility scores

## Testing

```bash
cd backend
pytest
pytest -v  # Verbose
pytest --cov  # With coverage
```

## Deployment

### Docker

Build and push images:
```bash
docker build -t chronos-backend backend/
docker build -t chronos-frontend frontend/
```

### Environment-specific Configuration

Create separate `.env` files:
- `.env.development`
- `.env.staging`
- `.env.production`

## Common Issues

### Port Already in Use
```bash
# Find process using port 8000
lsof -i :8000

# Find process using port 5173
lsof -i :5173

# Kill process
kill -9 <PID>
```

### Database Connection Issues
```bash
# Check PostgreSQL is running
docker ps | grep postgres

# Verify connection string in .env
# Format: postgresql://user:password@host:port/database
```

### Frontend Build Issues
```bash
# Clear node_modules and reinstall
rm -rf node_modules
npm install

# Clear Vite cache
rm -rf dist
npm run build
```

## Useful Commands

```bash
# Backend
uvicorn app.main:app --reload          # Development server with auto-reload
uvicorn app.main:app --host 0.0.0.0    # Expose to network
python -m app.main                      # Direct run

# Frontend
npm run dev                             # Development server
npm run build                           # Production build
npm run preview                         # Preview production build
npm run lint                            # Linting

# Docker
docker-compose up -d                    # Start in background
docker-compose logs -f backend          # Follow backend logs
docker-compose stop                     # Stop services
docker-compose down                     # Stop and remove containers
```

## Architecture Notes

### Backend
- **Framework**: FastAPI for async HTTP API
- **Agent Framework**: LangGraph for orchestrating complex AI workflows
- **Database**: PostgreSQL with pgvector for vector embeddings
- **Caching**: Redis for session management and caching
- **LLM Integration**: Support for multiple providers (OpenAI, Anthropic, Grok)

### Frontend
- **Framework**: React 18 with TypeScript
- **Routing**: React Router v6
- **Styling**: Tailwind CSS
- **Build**: Vite for fast development and building
- **Visualization**: Recharts, Leaflet, D3.js (to be integrated)

## Next Steps for Development

1. Implement LangGraph agent workflows
2. Set up RAG pipeline with PostgreSQL pgvector
3. Integrate with LLM APIs
4. Build timeline and chart visualizations
5. Implement citation verification system
6. Add authentication and user sessions
7. Create comprehensive test suite
8. Set up CI/CD pipeline

## Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [React Documentation](https://react.dev/)
- [PostgreSQL pgvector](https://github.com/pgvector/pgvector)
- [Tailwind CSS](https://tailwindcss.com/)
