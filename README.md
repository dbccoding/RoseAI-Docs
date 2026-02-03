# RoseAI System

An advanced AI companion system featuring multiple distinct personalities, sophisticated memory management, and collaborative storytelling capabilities.

RoseAI is currently a work in progress and is not available to clone. 

## Overview

RoseAI is a Flask-based conversational AI system featuring Rose, an advanced emotional companion with sophisticated personality variants. The system combines advanced memory management, emotional intelligence, and soft roleplay features to create engaging, context-aware conversations.

## Architecture

### Core Components

**Application Framework**
- Flask web server with WebSocket-like real-time communication
- SQLite-based data persistence for users and conversation memories
- Modular personality system with YAML-based trait configuration
- Enhanced model interface with retry logic, circuit breaker, and graceful error handling

**Memory Management System**
- SQLite-based impression storage with rich metadata
- Vector-based semantic memory (ChromaDB integration)
- Hybrid memory retrieval combining SQL and vector search
- Advanced memory optimization with clustering and summarization
- Session tracking with conversation turn management

**Personality Engine**
- Advanced Rose personality system with multiple variants (default, playful, professional)
- YAML-based trait system with personality-specific configurations
- Dynamic trait loading with token-budget management
- Location and context blocks for immersive romantic roleplay experiences

## AI Personalities

### Rose - The Advanced Companion
- **Access**: Available to all users (with admin features for advanced configurations)
- **Purpose**: Advanced emotional companion with sophisticated personality variants
- **Personality Variants**:
  - **Rose Default**: Core emotional companion with balanced interactions
  - **Rose Playful**: Cheeky variant with light teasing and wit
  - **Rose Professional**: More formal, structured interaction style
- **Features**:
  - Sophisticated emotional classification system with 20+ emotion types
  - Science fiction roleplay scenario with meaningful dilemmas to reason through with Rose
  - Advanced memory integration with contextual recall
  - Dynamic personality switching based on user preferences
  - Complex emotional taxonomy with genre-aware responses
- **Specialties**: Emotional support, roleplay, sophisticated conversation, personality adaptation and endlessly customisable identity

## Technical Features

### Memory Management
- **Impression System**: Every interaction classified and stored with emotional metadata
- **Contextual Recall**: Relevant memories automatically injected into conversations
- **Hybrid Storage**: SQL + vector embeddings for semantic and structured search
- **Clustering**: Automatic grouping of related memories by themes
- **Optimization**: Background memory cleanup and summarization

### Advanced RAG Implementation
- **Semantic Clustering**: TF-IDF vectorization with K-means clustering
- **Conversation Summarization**: Multi-strategy with emotional arc analysis
- **Adaptive Retrieval**: Context-aware memory selection with relevance scoring
- **Performance Monitoring**: Real-time metrics and optimization recommendations

### Error Handling & Resilience
- **Circuit Breaker**: Automatic API failure handling
- **Retry Logic**: Exponential backoff for failed requests
- **Graceful Degradation**: Fallback responses when LLM unavailable
- **Comprehensive Logging**: Structured logging with rotation and filtering

## Database Schema

### Users Table
- User authentication with role-based access (user/admin)
- Password hashing and session management
- Account creation and deletion capabilities

### Impressions Table
- Rich conversation memory storage
- Emotional metadata (weight, tone, tags)
- Session and conversation turn tracking
- Recall readiness flags for contextual retrieval

### Sessions Table
- Conversation session tracking
- Activity monitoring and session cleanup
- Metadata storage for session-specific data

## Installation & Setup

### Prerequisites
- Python 3.11+
- LMStudio running on localhost:1234
- Modern web browser

### Installation
```bash
# Set up virtual environment
python -m venv compEnv
compEnv\Scripts\activate  # Windows
# source compEnv/bin/activate  # Linux/Mac

# Install dependencies
pip install -r requirements.txt

# Initialize databases
python src/database.py

# Run application
python app.py
```

### Configuration
Environment variables in `.env` file:
```bash
FLASK_SECRET_KEY=your-secure-key-here
LMSTUDIO_API_URL=http://localhost:1234/v1/chat/completions
SESSION_LIFETIME_HOURS=1
ENABLE_VECTOR_MEMORY=True
```

## Usage

### Web Interface
1. Navigate to `http://localhost:5000`
2. Register an account or log in
3. Begin conversation with Rose
4. Optionally switch between Rose's personality variants

### User Roles
- **Standard Users**: Full access to Rose companion system
- **Admin Users**: Additional access to personality configuration and advanced features

### Personality Variants
- Dynamic switching between Rose's different personality modes
- Each variant has distinct interaction styles and emotional responses
- Seamless transitions based on conversation context or user preference

## Memory System

The system maintains detailed conversation memories with:
- **Automatic Classification**: Every interaction categorized by type and significance
- **Emotional Tracking**: Tone and emotional weight preserved
- **Contextual Retrieval**: Relevant memories automatically recalled
- **Performance Optimization**: Background cleanup and summarization

## API Endpoints

### Core Chat
- `POST /chat/rose` - Rose companion chat (all users)
- Rose personality variants handled automatically based on configuration

### User Management
- `POST /register` - User registration
- `POST /login` - User authentication
- `POST /logout` - Session cleanup

### Admin Functions
- `POST /admin/add_user` - Create new user accounts
- `GET /admin/list_users` - List all users
- `POST /admin/delete_user` - Remove user accounts
- `POST /admin/exit` - Graceful application shutdown

### Personality Management
- Automatic personality variant switching
- Configuration-based personality selection
- Real-time personality adaptation

## Performance & Optimization

### Memory Management
- Automatic memory usage monitoring
- Background cleanup processes
- Conversation summarization for long-term storage
- Vector database optimization

### Caching
- LRU cache for frequent queries
- Session-based conversation caching
- Memory cluster caching for performance

### Monitoring
- Real-time performance metrics
- Memory pressure detection
- API response time tracking
- Usage analytics and reporting

## Development Features

### Testing Suite
- Memory flow validation scripts
- Performance benchmarking tools
- Database migration utilities
- System diagnostic scripts

### Utilities
- Memory inspection tools (`scripts/inspect_memory.py`)
- Database cleanup utilities (`scripts/wipe_databases.py`)
- Environment diagnostics (`scripts/diagnose_env.py`)

## Security Considerations

- Password hashing with Werkzeug security
- Role-based access control for sensitive features
- Session management with configurable timeouts
- Input validation and sanitization
- Local processing (no external data sharing)

---

This system represents a sophisticated approach to conversational AI, combining multiple personality types with advanced memory management and storytelling capabilities for rich, contextual interactions. 
