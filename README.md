# content-publishing-api
Production-ready Content Management System API with JWT authentication, content versioning, scheduled publishing, media uploads, full-text search, and caching


## Overview

This is a production-ready Content Management System (CMS) API built with Node.js, Express, and PostgreSQL. It provides comprehensive content lifecycle management including drafting, scheduling, and publishing capabilities, with advanced features like content versioning, background job processing, media uploads, full-text search, and caching.

## Features

- **JWT Authentication**: Secure role-based access control (author & public roles)
- **Content Lifecycle**: Draft → Scheduled → Published workflow
- **Content Versioning**: Complete revision history for all content changes
- **Scheduled Publishing**: Automatic publishing via background workers
- **Media Management**: File upload and storage system  
- **Full-Text Search**: Fast content search using PostgreSQL GIN indexes
- **Caching Layer**: Redis-based caching for improved performance
- **Docker Support**: Fully containerized application stack
- **Comprehensive Testing**: Automated test suite for all core functionality

## Tech Stack

- **Runtime**: Node.js
- **Framework**: Express.js
- **Database**: PostgreSQL
- **Cache**: Redis
- **Authentication**: JWT (jsonwebtoken)
- **ORM**: Sequelize
- **Background Jobs**: node-cron
- **Containerization**: Docker & Docker Compose
- **Testing**: Jest & Supertest

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Node.js 18+ (for local development)
- Git

### Setup Instructions

1. **Clone the repository**
```bash
git clone https://github.com/Kishorekalingi/content-publishing-api.git
cd content-publishing-api
```

2. **Start with Docker Compose**
```bash
docker-compose up -d
```

This will start:
- API server on port 3000
- PostgreSQL database on port 5432
- Redis cache on port 6379
- Background worker for scheduled publishing

3. **Run database migrations**
```bash
npm run migrate
```

4. **Access the API**
- API Base URL: http://localhost:3000
- API Documentation: http://localhost:3000/api-docs

## Architecture

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed system architecture and design decisions.

## API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login and get JWT token

### Posts (Author)
- `POST /api/posts` - Create new post (draft)
- `GET /api/posts` - List author's posts
- `GET /api/posts/:id` - Get specific post
- `PUT /api/posts/:id` - Update post (creates revision)
- `DELETE /api/posts/:id` - Delete post
- `POST /api/posts/:id/publish` - Publish post immediately
- `POST /api/posts/:id/schedule` - Schedule post for future
- `GET /api/posts/:id/revisions` - Get post revision history

### Posts (Public)
- `GET /api/posts/published` - List all published posts
- `GET /api/posts/published/:id` - Get specific published post
- `GET /api/search?q=query` - Search published posts

### Media
- `POST /api/media/upload` - Upload media file (Author)

## Database Schema

### Users Table
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(50) DEFAULT 'author',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Posts Table
```sql
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(500) NOT NULL,
  slug VARCHAR(500) UNIQUE NOT NULL,
  content TEXT,
  status VARCHAR(50) DEFAULT 'draft',
  author_id INTEGER REFERENCES users(id),
  scheduled_for TIMESTAMP,
  published_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Post Revisions Table  
```sql
CREATE TABLE post_revisions (
  id SERIAL PRIMARY KEY,
  post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
  title_snapshot VARCHAR(500),
  content_snapshot TEXT,
  revision_author_id INTEGER REFERENCES users(id),
  revision_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Development

### Local Development Setup

```bash
# Install dependencies
npm install

# Copy environment variables
cp .env.example .env

# Start PostgreSQL and Redis (Docker)
docker-compose up -d postgres redis

# Run migrations
npm run migrate

# Start development server
npm run dev

# Start background worker
npm run worker
```

### Running Tests

```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run specific test file
npm test -- posts.test.js
```

## Environment Variables

Create a `.env` file with:

```env
NODE_ENV=development
PORT=3000

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cms_db
DB_USER=postgres
DB_PASSWORD=your_password

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# JWT
JWT_SECRET=your_jwt_secret_key_here
JWT_EXPIRES_IN=7d

# Worker
WORKER_INTERVAL=60000
```

## Background Worker

The scheduled publishing worker runs every minute and:
1. Queries posts with status='scheduled' and scheduled_for <= NOW()
2. Updates status to 'published' and sets published_at
3. Invalidates relevant cache entries
4. Logs all publishing actions

## Caching Strategy

- Published posts are cached for 1 hour
- Cache keys: `post:published:{id}` and `posts:published:list`
- Cache is invalidated on:
  - Post update
  - Post status change
  - New post published

## Testing

Test coverage includes:
- Authentication and authorization
- CRUD operations for posts
- Content lifecycle transitions
- Version history creation and retrieval
- Scheduled publishing worker
- Full-text search functionality
- Cache hit/miss scenarios
- Concurrent request handling

## Security

- Passwords hashed with bcrypt
- JWT tokens for authentication
- SQL injection protection via parameterized queries
- Input validation and sanitization
- Role-based access control (RBAC)
- Rate limiting on API endpoints

## Performance

- Database indexes on frequently queried fields
- Redis caching reduces database load
- Connection pooling for database
- Pagination for list endpoints
- Search response time < 500ms

## Deployment

### Docker Production Deployment

```bash
# Build and start all services
docker-compose -f docker-compose.prod.yml up -d

# View logs
docker-compose logs -f api

# Stop services
docker-compose down
```

## License

MIT

## Author

Kishore Kalingi
