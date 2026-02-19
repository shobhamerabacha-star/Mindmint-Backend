# Social Service - Quick Start Guide

## 5-Minute Setup

### 1. Install Dependencies
```bash
cd microservices/social-service
npm install
```

### 2. Configure Environment
```bash
cp .env.example .env
```

Update `.env` with your database credentials:
```env
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=quest_service
```

### 3. Start with Docker (Recommended)
```bash
cd docker
docker-compose up -d
```

This starts:
- PostgreSQL database on port 5432
- Social Service on port 3001

### 4. Run Migrations
```bash
npm run migration:run
```

### 5. Start Development Server
```bash
npm run start:dev
```

Server will be available at `http://localhost:3001`

### 6. Verify Installation
```bash
curl http://localhost:3001/health
# Expected: { "status": "ok" }
```

## Core Features

### Friends System
```bash
# Send friend request
curl -X POST http://localhost:3001/friends/request \
  -H "Content-Type: application/json" \
  -d '{"recipientId": "user-id-2"}'

# Get pending requests
curl http://localhost:3001/friends/requests/pending

# Accept request
curl -X POST http://localhost:3001/friends/requests/{requestId}/accept

# Get all friends
curl http://localhost:3001/friends
```

### Leaderboard
```bash
# Create entry
curl -X POST http://localhost:3001/leaderboards \
  -H "Content-Type: application/json" \
  -d '{"userId": "user-id-1", "score": 100}'

# Get top 10 players
curl http://localhost:3001/leaderboards/top?limit=10

# Get user's rank
curl http://localhost:3001/leaderboards/user/{userId}/rank

# Record a win
curl -X POST http://localhost:3001/leaderboards/user/{userId}/win?seasonId=current
```

### Multiplayer Rooms
```bash
# Create room
curl -X POST http://localhost:3001/rooms \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Puzzle Duel",
    "maxPlayers": 2,
    "isPrivate": false
  }'

# Get available rooms
curl http://localhost:3001/rooms

# Join room
curl -X POST http://localhost:3001/rooms/{roomId}/join

# Start game
curl -X POST http://localhost:3001/rooms/{roomId}/start
```

### WebSocket (Real-Time)
```javascript
// Connect to WebSocket
const ws = new WebSocket('ws://localhost:3001');

// Join room
ws.send(JSON.stringify({
  event: 'room-join',
  data: { roomId: 'room-123', userId: 'user-1' }
}));

// Listen for updates
ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log(message.event, message.data);
};

// Send room message
ws.send(JSON.stringify({
  event: 'room-message',
  data: { roomId: 'room-123', userId: 'user-1', message: 'Hello!' }
}));

// Record game complete
ws.send(JSON.stringify({
  event: 'game-complete',
  data: { 
    roomId: 'room-123', 
    userId: 'user-1',
    results: { winner: 'user-1', score: 150 }
  }
}));
```

## Project Structure

```
microservices/social-service/
├── src/
│   ├── app.module.ts         # Main app module
│   ├── app.controller.ts     # Root controller
│   ├── app.service.ts        # Root service
│   ├── main.ts               # Application entry point
│   ├── config/
│   │   └── orm-config.ts     # Database configuration
│   ├── common/
│   │   └── gateways/
│   │       └── social.gateway.ts  # WebSocket gateway
│   ├── friends/
│   │   ├── friend.entity.ts
│   │   ├── friend-request.entity.ts
│   │   ├── friends.service.ts
│   │   ├── friends.controller.ts
│   │   ├── friends.module.ts
│   │   └── dto/
│   ├── leaderboards/
│   │   ├── leaderboard-entry.entity.ts
│   │   ├── leaderboards.service.ts
│   │   ├── leaderboards.controller.ts
│   │   ├── leaderboards.module.ts
│   │   └── dto/
│   ├── rooms/
│   │   ├── multiplayer-room.entity.ts
│   │   ├── rooms.service.ts
│   │   ├── rooms.controller.ts
│   │   ├── rooms.module.ts
│   │   └── dto/
│   └── database/
│       ├── migrations/
│       └── SCHEMA.md
├── test/
├── docker/
├── package.json
├── tsconfig.json
├── ormconfig.ts
└── README.md
```

## Common Commands

```bash
# Development
npm run start:dev       # Start with hot reload
npm run build          # Build for production
npm run start:prod     # Run production build

# Testing
npm run test           # Run unit tests
npm run test:cov       # With coverage report
npm run test:watch     # Watch mode

# Database
npm run migration:generate  # Generate migration
npm run migration:run       # Run migrations
npm run migration:revert    # Revert last migration

# Code Quality
npm run lint           # Fix linting issues
npm run format         # Format code with Prettier
npm run type-check     # TypeScript check
```

## Docker Commands

```bash
# Start services
docker-compose -f docker/docker-compose.yml up -d

# View logs
docker-compose -f docker/docker-compose.yml logs -f social-service

# Stop services
docker-compose -f docker/docker-compose.yml down

# Remove volumes (clear data)
docker-compose -f docker/docker-compose.yml down -v

# Build custom image
docker build -f docker/Dockerfile -t social-service:v1 .

# Run custom image
docker run -p 3001:3001 --env-file .env social-service:v1
```

## Troubleshooting

### Port Already in Use
```bash
# Find process using port 3001
lsof -i :3001

# Kill process
kill -9 <PID>
```

### Database Connection Failed
1. Check PostgreSQL is running: `psql -U postgres`
2. Verify credentials in `.env`
3. Check database exists: `psql -l`
4. Run migrations: `npm run migration:run`

### WebSocket Connection Issues
1. Ensure service is running: `curl http://localhost:3001/health`
2. Check network connectivity
3. Verify WebSocket is enabled in browser console
4. Check for CORS issues

### TypeScript Errors
```bash
npm run type-check  # Check for TS errors
npm install        # Reinstall dependencies
rm -rf dist        # Clear build
npm run build      # Rebuild
```

## Next Steps

1. **Authentication**: Implement JWT authentication in controllers
2. **Authorization**: Add role-based access control
3. **Caching**: Integrate Redis for leaderboard caching
4. **Notifications**: Add email/push notifications
5. **Analytics**: Integrate tracking for user activities
6. **Tests**: Write integration and E2E tests

## API Documentation

- **Friends**: POST/GET /friends/*, POST /friends/requests/*, DELETE /friends/*
- **Leaderboards**: POST/GET /leaderboards/*, PATCH /leaderboards/user/*
- **Rooms**: POST/GET /rooms/*, POST /rooms/{id}/*
- **Health**: GET /health

See [README.md](README.md) for complete API documentation.

## Support

- Check logs: `npm run start:dev` or `docker-compose logs`
- Database issues: `npm run migration:run`
- WebSocket: Check browser console for connection errors
- Review service tests: `npm test`

## Key Features Implemented

✅ Bidirectional friend requests and relationships
✅ Leaderboard with automatic ranking
✅ Multiplayer room management
✅ WebSocket for real-time updates
✅ Database migrations and schema isolation
✅ Docker support with compose
✅ Unit tests with Jest
✅ TypeScript strict mode
✅ Environment configuration

## Architecture Notes

- **Schema Isolation**: All social service tables in `social` PostgreSQL schema
- **Entity Relationships**: Proper TypeORM decorators with indices
- **WebSocket**: Native NestJS WebSocket support with custom gateway
- **Migrations**: Versioned migrations for schema changes
- **Docker**: Multi-stage builds for optimized images
- **Testing**: Mock repositories for service tests

## Performance Optimization Tips

1. Add Redis caching for leaderboards
2. Index frequently queried fields (✅ already done)
3. Batch database operations where possible
4. Use pagination for large datasets
5. Implement rate limiting on API endpoints
6. Monitor database connection pool

Ready to start! 🚀
