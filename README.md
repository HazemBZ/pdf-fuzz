# pdf-fuzz

PoC bulk search your PDF files using fuzzy text lookup.

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![React](https://img.shields.io/badge/React-61DAFB?style=flat&logo=react&logoColor=black)](https://reactjs.org/)
[![Elasticsearch](https://img.shields.io/badge/Elasticsearch-005571?style=flat&logo=elasticsearch&logoColor=white)](https://www.elastic.co/elasticsearch/)

## Quick Start

Get up and running in 5 minutes:

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/HazemBZ/pdf-fuzz

# Navigate to project
cd pdf-fuzz

# Start all containers
docker-compose up

# Access the application
open http://localhost:7000
```

## Features

- **Fuzzy Text Search** - Search across PDF files with fuzzy matching
- **File Management** - Upload, view, and delete PDF files
- **Processing States** - Track file processing (queued/processing/success/failure)
- **File Deduplication** - Automatically detect and handle duplicate files
- **Dark/Light Mode** - Toggle between themes
- **Docker Setup** - One-command deployment with Docker Compose

## Architecture Overview

The application follows a microservices architecture with the following components:

![Data Flow](docs/diagrams/pdf_fuzz_data_flow.png)

### Components

- **Frontend** - React SPA with nginx reverse proxy
- **Backend** - Django REST API for file management and search
- **Elasticsearch** - Powers fuzzy text search capabilities
- **Celery** - Async task queue for PDF processing
- **RabbitMQ** - Message broker for Celery tasks
- **Redis** - Caching layer for task results

## Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Frontend | React 19, TypeScript, Vite, shadcn/ui | User interface |
| Backend | Django 5.2, Python 3.12 | REST API |
| Search | Elasticsearch 8.14 | Fuzzy text indexing |
| Task Queue | Celery + RabbitMQ | Async PDF processing |
| Cache | Redis | Task results |
| Infrastructure | Docker Compose | Container orchestration |

## Detailed Setup

### Docker Setup (Recommended)

**Prerequisites:**
- Docker
- docker-compose

**Steps:**

```bash
# 1. Clone the repository with submodules
git clone --recurse-submodules https://github.com/HazemBZ/pdf-fuzz

# 2. Navigate to project directory
cd pdf-fuzz

# 3. Start all services
docker-compose up

# 4. Access the application
# Frontend: http://localhost:7000
# Backend API: http://localhost:8000
# Elasticsearch: http://localhost:9200
```

### Manual Setup

**Prerequisites:**
- Python 3.12+
- Node.js 18+
- pnpm
- Elasticsearch 8.x
- RabbitMQ
- Redis

**Backend Setup:**

```bash
cd pdf_fuzz_back

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# or
.venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Start Elasticsearch, RabbitMQ, Redis (or use Docker)
# Start Django server
python manage.py runserver 8000
```

**Frontend Setup:**

```bash
cd pdf-fuzz-front

# Install dependencies
pnpm install

# Start development server
pnpm dev
```

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `ELASTIC_ADDRESS` | Elasticsearch host | `elastic` |
| `CELERY_BROKER` | RabbitMQ URL | `amqp://guest@rabbitmq//` |
| `CELERY_RESULT_BACKEND` | Redis URL | `redis://redis:6379/0` |
| `PROD` | Production mode | `false` |
| `BACKEND_DOMAIN` | Backend hostname | `backend` |
| `BACKEND_PORT` | Backend port | `8000` |

### Ports

| Service | Port | Description |
|---------|------|-------------|
| Frontend | 7000 | Application UI |
| Backend | 8000 | REST API |
| Elasticsearch | 9200 | Search engine |
| RabbitMQ | 5672 | Message broker |
| RabbitMQ Management | 15672 | Management UI |
| Redis | 6379 | Cache |

## Usage

### 1. Upload PDF Files

Drag and drop PDF files into the upload area or use the file picker. Files are automatically processed and indexed.

### 2. Search Documents

Enter your search query in the search bar. The fuzzy search will find relevant documents even with partial matches or typos.

### 3. View Results

Browse through search results with preview thumbnails. Click on any result to view the full PDF page.

### 4. Manage Files

- View file processing status
- Delete files you no longer need
- The system automatically handles duplicate files

## Development

### Starting Development Environment

```bash
# Start all services with hot-reloading
docker-compose up

# Start in background
docker-compose up -d

# Start with Flower dashboard (optional)
docker-compose --profile flower up
```

### Development Workflow

#### Frontend Development

The frontend container supports hot-reloading via volume mounts:

```bash
# Frontend auto-reloads on file changes
# Edit files in pdf-fuzz-front/src/

# View frontend logs
docker-compose logs -f frontend
```

#### Backend Development

The backend container also supports hot-reloading:

```bash
# Backend auto-reloads on Django file changes
# Edit files in pdf_fuzz_back/

# View backend logs
docker-compose logs -f backend

# Access Django shell
docker-compose exec backend python manage.py shell

# Run migrations
docker-compose exec backend python manage.py migrate
```

### Running Tests

```bash
# Backend tests
docker-compose exec backend pytest

# Frontend linting and type checking
docker-compose exec frontend pnpm lint
docker-compose exec frontend pnpm typecheck
```

### Useful Commands

```bash
# Rebuild containers after dependency changes
docker-compose up --build

# Stop all services
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v

# View running containers
docker-compose ps

# Execute command in specific container
docker-compose exec <service> <command>
```

### Accessing Services

| Service | URL | Credentials |
|---------|-----|-------------|
| Frontend | http://localhost:7000 | - |
| Backend API | http://localhost:8000 | - |
| Elasticsearch | http://localhost:9200 | - |
| RabbitMQ Management | http://localhost:15672 | guest/guest |
| Flower Dashboard | http://localhost:5555 | (with `--profile flower`) |

### Debugging

```bash
# View all logs
docker-compose logs -f

# View logs for specific service
docker-compose logs -f backend

# Check container health
docker-compose ps

# Inspect container
docker-compose inspect backend

# Access container shell
docker-compose exec backend /bin/bash
```

## Troubleshooting

### Common Issues

#### 1. Container Won't Start

**Symptom:** Docker containers fail to start or immediately exit.

**Solution:**
- Check if ports are already in use: `lsof -i :7000`
- Ensure Docker is running: `docker info`
- Check Docker logs: `docker-compose logs`

#### 2. Elasticsearch Health Check Failed

**Symptom:** Backend can't connect to Elasticsearch.

**Solution:**
- Wait 30-60 seconds for Elasticsearch to initialize
- Check Elasticsearch status: `curl http://localhost:9200`
- View Elasticsearch logs: `docker-compose logs elastic`

#### 3. Frontend Can't Connect to Backend

**Symptom:** API requests fail from the frontend.

**Solution:**
- Verify backend is healthy: `docker-compose ps`
- Check backend logs: `docker-compose logs backend`
- Ensure nginx proxy is configured correctly

#### 4. Celery Worker Not Processing Tasks

**Symptom:** PDF files stuck in "processing" state.

**Solution:**
- Check RabbitMQ status: `http://localhost:15672`
- View Celery logs: `docker-compose logs celery`
- Restart Celery worker: `docker-compose restart celery`

#### 5. Frontend Build Fails

**Symptom:** Docker build fails for frontend container.

**Solution:**
- Clear Docker cache: `docker-compose build --no-cache`
- Check Node.js version compatibility
- Verify pnpm lockfile is present

### Viewing Logs

```bash
# View all logs
docker-compose logs -f

# View logs for specific service
docker-compose logs -f backend

# View last 100 lines
docker-compose logs --tail 100 backend
```

### Resetting the Environment

```bash
# Stop and remove all containers
docker-compose down

# Remove volumes (database, search index)
docker-compose down -v

# Rebuild and start fresh
docker-compose up --build
```

## Project Structure

```
pdf-fuzz/
├── pdf_fuzz_back/              # Django backend
│   ├── PDF_Fuzz/              # Main Django project
│   │   ├── settings.py        # Django settings
│   │   ├── urls.py            # URL routing
│   │   └── celery.py          # Celery configuration
│   ├── fuzz/                  # Search application
│   │   ├── models.py          # Data models
│   │   ├── views.py           # API views
│   │   ├── tasks.py           # Celery tasks
│   │   └── search.py          # Elasticsearch integration
│   ├── requirements.txt       # Python dependencies
│   └── Dockerfile
├── pdf-fuzz-front/            # React frontend
│   ├── src/                   # Source code
│   │   ├── components/        # React components
│   │   ├── stores/            # State management
│   │   └── App.tsx            # Main app component
│   ├── package.json           # Node.js dependencies
│   └── Dockerfile
├── docs/                      # Documentation
│   └── diagrams/             # Architecture diagrams
├── docker-compose.yaml        # Container orchestration
└── README.md                  # This file
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines

- Follow existing code style and conventions
- Write tests for new features
- Update documentation as needed
- Keep commits focused and descriptive

## Acknowledgments

- Built with Django, React, and Elasticsearch
- Containerized with Docker Compose
- Inspired by the need for efficient PDF search capabilities
