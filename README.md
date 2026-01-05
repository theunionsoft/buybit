# Plasma Lunar Marketplace

Next-gen classified + multi-vendor marketplace platform.

## Tech Stack
- **Frontend:** Next.js 14 (App Router)
- **Backend:** NestJS
- **Database:** PostgreSQL 15
- **Cache:** Redis 7
- **Monorepo:** Turborepo + pnpm

## Getting Started

See `/docs` for full documentation.

```bash
# Install dependencies
pnpm install

# Start database
docker-compose up -d

# Run migrations
pnpm --filter api prisma migrate dev

# Start development
pnpm dev
```

## Project Structure

```
plasma-lunar/
├── apps/
│   ├── web/          # Next.js frontend
│   └── api/          # NestJS backend
├── packages/
│   ├── ui/           # Shared UI components
│   └── types/        # Shared TypeScript types
└── docs/             # Documentation
```

## Development

- All UX/UI specifications are frozen - see `/docs`
- MVP scope only (8 weeks)
- No feature additions beyond specification
- Follow design system strictly

## License

Proprietary
