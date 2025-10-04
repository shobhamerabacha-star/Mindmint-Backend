# Game Database Schema Documentation

This document describes the comprehensive database schema designed for the quest-service game application, including user management, puzzle systems, progress tracking, achievements, and analytics.

## 📋 Table of Contents

- [Overview](#overview)
- [Database Architecture](#database-architecture)
- [Entity Relationships](#entity-relationships)
- [Core Entities](#core-entities)
- [Migration Scripts](#migration-scripts)
- [Performance Optimizations](#performance-optimizations)
- [Data Integrity](#data-integrity)
- [Setup Instructions](#setup-instructions)

## 🏗️ Overview

The database schema is designed to support a comprehensive puzzle game platform with the following key features:

- **User Management**: Comprehensive user profiles with preferences and statistics
- **Puzzle System**: Flexible puzzle content with categories, difficulty ratings, and metadata
- **Progress Tracking**: Detailed tracking of user progress and performance
- **Achievement System**: Complex achievement unlock conditions and progression tracking
- **Analytics**: Session tracking and performance metrics
- **Scalability**: Optimized for performance with proper indexing and constraints

## 🏛️ Database Architecture

### Technology Stack

- **Database**: PostgreSQL 14+
- **ORM**: TypeORM
- **Migration System**: TypeORM Migrations
- **Extensions**: uuid-ossp, pg_stat_statements

### Key Design Principles

1. **Normalization**: Proper normalization to reduce data redundancy
2. **Performance**: Strategic indexing for common query patterns
3. **Integrity**: Comprehensive constraints and validation rules
4. **Flexibility**: JSONB fields for extensible metadata
5. **Scalability**: Optimized for high-volume operations

## 🔗 Entity Relationships

```
Users (1) ←→ (∞) UserAchievements (∞) ←→ (1) Achievements
  ↓
  (1) ←→ (∞) PuzzleProgress (∞) ←→ (1) Puzzles
  ↓                                     ↓
  (1) ←→ (∞) GameSessions              (1) ←→ (∞) PuzzleRatings
  ↓                                     ↓
  (1) ←→ (1) UserStats                 (∞) ←→ (1) PuzzleCategories
```

## 📊 Core Entities

### 👤 Users

**Primary user entity with comprehensive profile and preference management.**

```sql
Table: users
- id (UUID, PK)
- username (VARCHAR, UNIQUE)
- email (VARCHAR, UNIQUE)
- firstName, lastName (VARCHAR)
- avatar, dateOfBirth, country (VARCHAR/DATE, NULLABLE)
- status (ENUM: active, inactive, suspended, deleted)
- role (ENUM: player, admin, moderator)
- totalScore, level, experience (INTEGER)
- puzzlesSolved, achievementsCount (INTEGER)
- lastLoginAt, lastActiveAt (TIMESTAMP)
- preferences (JSONB) - theme, difficulty, notifications, privacy, gameSettings
- profile (JSONB) - bio, website, socialLinks, stats
- metadata (JSONB) - system metadata
- createdAt, updatedAt, deletedAt (TIMESTAMP)
```

**Key Features:**

- Comprehensive user preferences stored as JSONB
- Profile information with social links and bio
- Performance tracking (score, level, experience)
- Soft delete support with `deletedAt`

### 🧩 Puzzles

**Flexible puzzle content system with rich metadata.**

```sql
Table: puzzles
- id (UUID, PK)
- title, description (VARCHAR/TEXT)
- category (VARCHAR) - links to puzzle_categories
- difficulty (ENUM: easy, medium, hard, expert)
- difficultyRating (INTEGER 1-10)
- basePoints, timeLimit, maxHints (INTEGER)
- attempts, completions (INTEGER) - analytics counters
- averageRating, ratingCount (DECIMAL/INTEGER)
- averageCompletionTime (INTEGER)
- isActive, isFeatured (BOOLEAN)
- publishedAt (TIMESTAMP)
- createdBy (UUID, FK to users)
- parentPuzzleId (UUID, FK to puzzles) - for puzzle series
- content (JSONB) - puzzle question, options, answers, media
- hints (JSONB) - structured hint system
- tags (TEXT ARRAY) - searchable tags
- prerequisites (JSONB) - required puzzle IDs
- scoring (JSONB) - scoring configuration
- analytics (JSONB) - performance analytics
- metadata (JSONB) - admin metadata
```

**Key Features:**

- Flexible content system supporting multiple puzzle types
- Hierarchical puzzle relationships (parent/child)
- Rich analytics and performance tracking
- Advanced scoring and hint systems

### 📈 PuzzleProgress

**Detailed tracking of user progress on individual puzzles.**

```sql
Table: puzzle_progress
- id (UUID, PK)
- userId (UUID, FK to users)
- puzzleId (UUID, FK to puzzles)
- status (ENUM: not_started, in_progress, completed, failed, skipped)
- attempts, score, bestScore (INTEGER)
- hintsUsed, timeSpent, bestTime (INTEGER)
- startedAt, completedAt, lastAttemptAt (TIMESTAMP)
- rating (DECIMAL 1-5) - user's rating of the puzzle
- progress (JSONB) - current state, checkpoints, answers
- metrics (JSONB) - performance metrics
- sessionData (JSONB) - interaction data
```

**Key Features:**

- Comprehensive progress tracking with multiple attempts
- Performance metrics and analytics
- Save state for resuming puzzles
- User ratings integrated with progress

### 🏆 Achievements

**Complex achievement system with flexible unlock conditions.**

```sql
Table: achievements
- id (UUID, PK)
- name, description (VARCHAR/TEXT)
- category (VARCHAR) - puzzle_mastery, speed, consistency, exploration, social
- rarity (ENUM: common, rare, epic, legendary)
- points (INTEGER) - points awarded
- iconUrl, badgeUrl (VARCHAR) - visual assets
- isActive, isSecret (BOOLEAN)
- unlockedCount, unlockRate (INTEGER/DECIMAL) - statistics
- unlockConditions (JSONB) - complex condition system
- prerequisites (TEXT ARRAY) - required achievement IDs
- progression (JSONB) - multi-step achievements
- timeConstraints (JSONB) - seasonal/event achievements
- metadata (JSONB) - tips, celebration messages
```

**Key Features:**

- Flexible unlock condition system
- Multi-step achievement progression
- Seasonal and event-based achievements
- Secret achievements support

### 🎯 UserAchievements

**Junction table tracking user achievement progress.**

```sql
Table: user_achievements
- id (UUID, PK)
- userId (UUID, FK to users)
- achievementId (UUID, FK to achievements)
- progress, progressTotal (INTEGER)
- isUnlocked, isNotified, isViewed (BOOLEAN)
- unlockedAt, notifiedAt, viewedAt (TIMESTAMP)
- unlockContext (JSONB) - context of unlock
- progressDetails (JSONB) - step-by-step progress
```

### 🎮 GameSessions

**Comprehensive session tracking for analytics.**

```sql
Table: game_sessions
- id (UUID, PK)
- sessionId (VARCHAR, UNIQUE)
- userId (UUID, FK to users)
- platform (ENUM: web, mobile, tablet, desktop)
- deviceInfo, browserInfo (VARCHAR)
- startTime, endTime (TIMESTAMP)
- duration (INTEGER) - in seconds
- puzzlesAttempted, puzzlesCompleted, puzzlesFailed, puzzlesSkipped (INTEGER)
- totalScore, totalHintsUsed, achievementsUnlocked (INTEGER)
- averageAccuracy, longestStreak (DECIMAL/INTEGER)
- puzzleIds, categoriesPlayed (TEXT ARRAY)
- analytics (JSONB) - detailed performance metrics
- sessionConfig (JSONB) - game mode, filters, settings
- sessionState (JSONB) - current state for resuming
- contextData (JSONB) - location, referrer, experiments
- isActive, status (BOOLEAN/ENUM)
```

### ⭐ PuzzleRatings

**User ratings and reviews for puzzles.**

```sql
Table: puzzle_ratings
- id (UUID, PK)
- userId (UUID, FK to users)
- puzzleId (UUID, FK to puzzles)
- rating (DECIMAL 1-5)
- difficultyVote (ENUM: easy, medium, hard, expert)
- review (TEXT) - optional text review
- tags (TEXT ARRAY) - review tags
- isReported, isPublic (BOOLEAN)
- metadata (JSONB) - completion context
```

### 📊 UserStats

**Aggregated user statistics and performance metrics.**

```sql
Table: user_stats
- id (UUID, PK)
- userId (UUID, FK to users, UNIQUE)
- totalPuzzlesAttempted, totalPuzzlesCompleted, totalPuzzlesFailed (INTEGER)
- totalScore, totalTimeSpent, totalHintsUsed (INTEGER)
- currentStreak, longestStreak (INTEGER)
- overallAccuracy, averageCompletionTime (DECIMAL)
- totalAchievements, totalGameSessions (INTEGER)
- difficultyStats (JSONB) - stats by difficulty level
- categoryStats (JSONB) - stats by category
- timeStats (JSONB) - daily/weekly/monthly patterns
- trends (JSONB) - skill progression and improvements
- milestones (JSONB) - significant achievements
- rankings (JSONB) - leaderboard positions
- lastActivityAt, lastCalculatedAt (TIMESTAMP)
```

### 📁 PuzzleCategories

**Puzzle categorization system.**

```sql
Table: puzzle_categories
- id (UUID, PK)
- name, slug (VARCHAR, UNIQUE)
- description (TEXT)
- iconUrl, color (VARCHAR)
- isActive (BOOLEAN)
- sortOrder, puzzleCount (INTEGER)
- metadata (JSONB) - skills, age range, time estimates
```

## 🗂️ Migration Scripts

The database schema is implemented through a series of TypeORM migrations:

1. **CreateGameDatabaseSchema.ts** - Core tables (users, puzzles, categories)
2. **CreateProgressAndAchievementTables.ts** - Progress and achievement tracking
3. **CreateSupportingTables.ts** - Ratings and statistics
4. **AddPerformanceIndexes.ts** - Performance optimization indexes
5. **AddDatabaseConstraints.ts** - Data integrity constraints
6. **SeedInitialData.ts** - Initial puzzle and achievement data

### Running Migrations

```bash
# Setup database (automated script)
./scripts/setup-database.sh

# Manual migration commands
npm run build
npx typeorm migration:run -d dist/config/orm-config.js
```

## ⚡ Performance Optimizations

### Strategic Indexing

**Primary Indexes:**

- Unique indexes on email, username
- Composite indexes for common query patterns
- Foreign key indexes for join performance

**Specialized Indexes:**

- Full-text search on puzzle content
- JSONB indexes for frequent JSON queries
- Partial indexes for filtered queries
- GIN indexes for array operations

**Example Indexes:**

```sql
-- Leaderboard queries
CREATE INDEX CONCURRENTLY "IDX_user_stats_leaderboard_total"
ON "user_stats" ("totalScore" DESC, "totalPuzzlesCompleted" DESC);

-- Puzzle discovery
CREATE INDEX CONCURRENTLY "IDX_puzzles_category_difficulty_rating"
ON "puzzles" ("category", "difficulty", "averageRating" DESC)
WHERE "isActive" = true AND "publishedAt" IS NOT NULL;

-- Full-text search
CREATE INDEX CONCURRENTLY "IDX_puzzles_fulltext_search"
ON "puzzles" USING gin(to_tsvector('english', "title" || ' ' || "description"));
```

### Query Optimization Features

- **Concurrent index creation** to avoid blocking
- **Partial indexes** for filtered queries
- **Composite indexes** for multi-column queries
- **JSONB indexing** for metadata queries

## 🛡️ Data Integrity

### Constraints and Validation

**Check Constraints:**

- Email format validation
- Enum value validation
- Positive value constraints
- Range validation (ratings 1-5, difficulty 1-10)
- Logical consistency (best score ≥ current score)

**Foreign Key Constraints:**

- Cascade deletes for dependent data
- Set null for optional references
- Referential integrity maintenance

**Business Logic Constraints:**

```sql
-- User constraints
ALTER TABLE "users" ADD CONSTRAINT "CHK_users_email_format"
CHECK ("email" ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Puzzle progress logic
ALTER TABLE "puzzle_progress" ADD CONSTRAINT "CHK_puzzle_progress_completion_logic"
CHECK (
  ("status" != 'completed' AND "completedAt" IS NULL) OR
  ("status" = 'completed' AND "completedAt" IS NOT NULL)
);

-- Achievement unlock logic
ALTER TABLE "user_achievements" ADD CONSTRAINT "CHK_user_achievements_unlock_logic"
CHECK (
  ("isUnlocked" = false AND "unlockedAt" IS NULL) OR
  ("isUnlocked" = true AND "unlockedAt" IS NOT NULL)
);
```

## 🚀 Setup Instructions

### Prerequisites

- PostgreSQL 14+
- Node.js 18+
- npm or yarn

### Database Setup

1. **Clone and install dependencies:**

```bash
git clone <repository>
cd quest-service
npm install
```

2. **Configure environment:**

```bash
cp .env.example .env
# Edit .env with your database credentials
```

3. **Run database setup:**

```bash
./scripts/setup-database.sh
```

4. **Verify setup:**

```bash
npm run start:dev
```

### Environment Variables

```env
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=your_password
DB_NAME=quest_service
NODE_ENV=development
```

### Development Workflow

```bash
# Generate new migration
npx typeorm migration:generate src/migrations/YourMigrationName -d src/config/orm-config.ts

# Run migrations
npm run build
npx typeorm migration:run -d dist/config/orm-config.js

# Revert migration
npx typeorm migration:revert -d dist/config/orm-config.js
```

## 📝 Sample Data

The database includes comprehensive seed data:

- **5 Puzzle Categories**: Logic, Math, Word Games, Pattern Recognition, Spatial Reasoning
- **5 Sample Puzzles**: Covering different difficulty levels and types
- **7 Achievements**: Including common, rare, epic achievements and one secret achievement
- **Rich Metadata**: Realistic examples of JSONB content structures

## 🔧 Maintenance

### Regular Tasks

- Monitor query performance using pg_stat_statements
- Update statistics: `ANALYZE;`
- Vacuum regularly: `VACUUM ANALYZE;`
- Monitor index usage and add/remove as needed

### Monitoring Queries

```sql
-- Check table sizes
SELECT schemaname,tablename,pg_size_pretty(size) as size_pretty
FROM (SELECT schemaname,tablename,pg_total_relation_size(schemaname||'.'||tablename) as size
      FROM pg_tables WHERE schemaname='public') t
ORDER BY size DESC;

-- Monitor query performance
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

---

## 🎉 Conclusion

This comprehensive database schema provides a solid foundation for a scalable puzzle game platform with:

✅ **Complete entity relationships with proper foreign keys**  
✅ **Performance-optimized indexes for all common queries**  
✅ **Comprehensive constraints ensuring data integrity**  
✅ **Flexible JSONB fields for extensible metadata**  
✅ **Rich seed data for immediate development**  
✅ **Migration scripts for all environments**  
✅ **Detailed documentation and setup instructions**

The schema is designed to scale from development through production, with monitoring and maintenance procedures documented for long-term success.
