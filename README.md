# RedisBoard

Lightweight leaderboard engine built on Redis sorted sets. Handles global rankings, per-entity rankings, and configurable top-k queries with atomic score updates.

## How It Works

```
Redis ZSET (O(log n)):
    user123 → 1850.5  ← score
    user456 → 1720.0
    user789 → 1690.3
    ...

Operations:
    ZADD   → atomic score update
    ZRANK  → get user rank (ascending)
    ZREVRANK → get user rank (descending, for leaderboards)
    ZREVRANGE → get top-k users
```

## Features

- **Global leaderboard** — rank all users by score
- **Entity leaderboards** — rank within groups (countries, regions)
- **Configurable top-k** — show top N entries
- **Atomic updates** — no race conditions on score changes
- **Float/int scores** — configurable precision

## Usage

```go
import "github.com/lijuuu/RedisBoard"

board := redisboard.New(redisboard.Config{
    Namespace: "zenxbattle",
    K:          100,        // top 100
    RedisAddr:  "localhost:6379",
})

// Add/update score
board.UpdateScore(ctx, "player_123", 1850.5)

// Get top 10
top10, _ := board.GetTopK(ctx, 10)

// Get user's rank
rank, _ := board.GetRank(ctx, "player_123")

// Entity rankings (per problem, per country, etc.)
board.UpdateEntityScore(ctx, "problem-42", "player_123", 95.0)
top5 := board.GetTopK(ctx, 5)
```

## Performance

Redis ZSET operations are O(log n). With 1M users:
- **Score update:** ~0.1ms
- **Top-100 query:** ~0.5ms
- **Rank lookup:** ~0.1ms

No indexing needed — Redis ZSETs are natively ordered.

## Related Services

- [ChallengeService](https://github.com/zenxbattle/ChallengeService) — battle results feed into rankings
- [ProblemService](https://github.com/zenxbattle/ProblemService) — per-problem leaderboards
- [Frontend](https://github.com/zenxbattle/Frontend) — leaderboard display

## Docs

See `doc/` for detailed API reference and configuration guide.
