# Replay Service Quick Start Guide

## 🚀 Get Started in 5 Minutes

### Option 1: Docker (Recommended)

```bash
# Start the complete stack (service + database)
docker-compose up -d

# Check if it's running
docker-compose ps

# View logs
docker-compose logs -f replay-service

# Access the service:
# - API: http://localhost:3007
# - Swagger Docs: http://localhost:3007/api/docs
# - Database: postgres://replay_user:replay_password@localhost:5433/replay_db
```

### Option 2: Local Development

```bash
# Install dependencies
npm install

# Start development server
npm run start:dev

# Server will run on http://localhost:3007
```

## 📚 Example API Usage

### 1. Create a Replay Session

```bash
curl -X POST http://localhost:3007/replay/create \
  -H "Content-Type: application/json" \
  -d '{
    "puzzleId": 1,
    "playerId": 1,
    "title": "Puzzle 1 - First Attempt",
    "initialState": {"grid": [1,2,3]},
    "privacyLevel": "private"
  }'
```

**Response:**
```json
{
  "id": "uuid-1",
  "puzzleId": 1,
  "playerId": 1,
  "title": "Puzzle 1 - First Attempt",
  "privacyLevel": "private",
  "createdAt": "2026-01-24T..."
}
```

### 2. Record Actions During Gameplay

```bash
# Single action
curl -X POST http://localhost:3007/replay/uuid-1/action \
  -H "Content-Type: application/json" \
  -d '{
    "type": "move",
    "payload": {"x": 10, "y": 20},
    "timestamp": 1705979000000
  }'

# Multiple actions (batch)
curl -X POST http://localhost:3007/replay/uuid-1/actions \
  -H "Content-Type: application/json" \
  -d '[
    {
      "type": "rotate",
      "payload": {"angle": 90},
      "timestamp": 1705979001000
    },
    {
      "type": "place_piece",
      "payload": {"x": 5, "y": 10},
      "timestamp": 1705979002000
    }
  ]'
```

### 3. Generate Playback

```bash
# Normal speed
curl -X POST http://localhost:3007/replay/uuid-1/playback \
  -H "Content-Type: application/json" \
  -d '{"speed": 1}'

# Fast replay (2x speed)
curl -X POST http://localhost:3007/replay/uuid-1/playback \
  -H "Content-Type: application/json" \
  -d '{"speed": 2}'

# Play from 5-20 seconds
curl -X POST http://localhost:3007/replay/uuid-1/playback \
  -H "Content-Type: application/json" \
  -d '{"startPosition": 5000, "endPosition": 20000}'
```

### 4. Save and Compress Replay

```bash
curl -X POST http://localhost:3007/replay/uuid-1/save \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Response:**
```json
{
  "id": "recording-1",
  "replayId": "uuid-1",
  "status": "completed",
  "compressionType": "gzip",
  "originalSize": 125000,
  "compressedSize": 28000,
  "compressionRatio": 77.6,
  "storageUrl": "/storage/replays/uuid-1-1705979000.replay"
}
```

### 5. Share Replay with Others

```bash
# Make it public
curl -X PUT http://localhost:3007/replay/uuid-1/privacy \
  -H "Content-Type: application/json" \
  -d '{"privacyLevel": "public"}' \
  -G --data-urlencode "playerId=1"

# Share with specific users
curl -X POST http://localhost:3007/replay/uuid-1/share \
  -H "Content-Type: application/json" \
  -d '{"userIds": [2, 3, 4]}' \
  -G --data-urlencode "playerId=1"
```

### 6. Generate Analytics

```bash
curl http://localhost:3007/replay/uuid-1/analytics
```

**Response:**
```json
{
  "replayId": "uuid-1",
  "totalActions": 42,
  "totalDuration": 125000,
  "actionBreakdown": {
    "move": 20,
    "rotate": 10,
    "place_piece": 8,
    "hint_used": 2,
    "undo": 2
  },
  "hints": {
    "count": 2,
    "percentage": 5
  },
  "playerSkillLevel": "intermediate",
  "keyInsights": [
    "✓ Successfully completed the puzzle",
    "Good efficiency but some trial-and-error involved",
    "Used 2 hints (5% of actions)",
    "Deliberate and careful approach with few corrections"
  ],
  "performanceMetrics": {
    "avgTimePerAction": 2976,
    "peakActivityPeriod": "mid-early",
    "consistencyScore": 78
  }
}
```

### 7. Access via Share Token

```bash
# Get unlisted replay
curl http://localhost:3007/replay/share/share-token-123
```

## 🗄️ Database Access

Connect with your favorite database client:

```
Host: localhost
Port: 5433
Username: replay_user
Password: replay_password
Database: replay_db
```

## 🧪 Running Tests

```bash
# Unit tests
npm run test

# Test coverage
npm run test:cov

# E2E tests
npm run test:e2e

# Watch mode
npm run test:watch
```

## 📖 API Documentation

Visit the interactive Swagger UI:
```
http://localhost:3007/api/docs
```

## 🛑 Stopping the Service

```bash
# Docker
docker-compose down

# All data persists in volumes

# To clean up everything
docker-compose down -v
```

## 🔧 Common Tasks

### Check Service Health
```bash
curl http://localhost:3007/health
```

### View Database Logs
```bash
docker-compose logs postgres
```

### Rebuild Docker Image
```bash
docker-compose down
docker-compose up --build
```

### Run Linter
```bash
npm run lint
```

### Format Code
```bash
npm run format
```

## 📝 Environment Variables

Key variables to customize:

```dotenv
# Server
PORT=3007
NODE_ENV=development

# Database
DB_HOST=localhost
DB_PORT=5433
DB_USER=replay_user
DB_PASSWORD=replay_password
DB_NAME=replay_db

# Storage
STORAGE_TYPE=local
STORAGE_PATH=./storage/replays

# Logging
LOG_LEVEL=debug
```

## 🐛 Troubleshooting

### Port Already in Use
```bash
# Change port in .env
PORT=3008
```

### Database Connection Failed
```bash
# Check database is running
docker-compose ps

# Verify credentials in .env
# Default: replay_user / replay_password
```

### Clear Database
```bash
docker-compose down -v
docker-compose up
```

### Storage Directory Issues
```bash
# Create storage directory
mkdir -p ./storage/replays
chmod 777 ./storage/replays
```

## 📊 Next Steps

1. ✅ Service is running
2. 📖 Read full [README.md](./README.md)
3. 🔍 Check [IMPLEMENTATION_SUMMARY.md](./IMPLEMENTATION_SUMMARY.md)
4. 🧪 Run tests: `npm run test:e2e`
5. 🚀 Deploy to production

## 🆘 Need Help?

- Check API documentation: `/api/docs`
- Review error messages in service logs
- Check database directly for data verification
- Review test files for usage examples

## 📦 Project Structure

```
replay-service/
├── src/
│   ├── replay/              - Core replay logic
│   ├── compression/         - Compression service
│   ├── storage/            - Storage abstraction
│   ├── analytics/          - Analytics engine
│   ├── entities/           - Database models
│   └── dto/                - Request validation
├── test/                   - Test files
├── Dockerfile              - Container image
├── docker-compose.yml      - Local stack
└── package.json            - Dependencies
```

---

**Happy Replaying! 🎬**
