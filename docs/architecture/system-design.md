# Plasma Lunar Marketplace - Project Structure

> **Type:** Monorepo (Turborepo)  
> **Package Manager:** pnpm  
> **Stack:** Next.js + NestJS + PostgreSQL + Redis

---

## Complete Folder Structure

```
plasma-lunar-marketplace/
├── .git/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
│
├── apps/
│   ├── web/                          # Next.js Frontend
│   │   ├── .next/
│   │   ├── app/
│   │   │   ├── (public)/
│   │   │   │   ├── page.tsx          # Homepage
│   │   │   │   ├── category/
│   │   │   │   │   └── [slug]/
│   │   │   │   │       └── page.tsx  # Category page
│   │   │   │   ├── ad/
│   │   │   │   │   └── [slug]/
│   │   │   │   │       └── page.tsx  # Ad detail
│   │   │   │   └── search/
│   │   │   │       └── page.tsx      # Search results
│   │   │   │
│   │   │   ├── (auth)/
│   │   │   │   ├── login/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── verify/
│   │   │   │       └── page.tsx
│   │   │   │
│   │   │   ├── (dashboard)/
│   │   │   │   ├── layout.tsx        # Dashboard layout
│   │   │   │   ├── ads/
│   │   │   │   │   └── page.tsx      # My ads
│   │   │   │   ├── favorites/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── messages/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── profile/
│   │   │   │       └── page.tsx
│   │   │   │
│   │   │   ├── admin/
│   │   │   │   ├── layout.tsx
│   │   │   │   ├── dashboard/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── moderation/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── users/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── categories/
│   │   │   │       └── page.tsx
│   │   │   │
│   │   │   ├── api/                  # API routes (if needed)
│   │   │   │   └── upload/
│   │   │   │       └── route.ts
│   │   │   │
│   │   │   ├── layout.tsx            # Root layout
│   │   │   └── globals.css
│   │   │
│   │   ├── components/
│   │   │   ├── ads/
│   │   │   │   ├── ad-card.tsx
│   │   │   │   ├── ad-grid.tsx
│   │   │   │   └── ad-detail.tsx
│   │   │   ├── category/
│   │   │   │   ├── category-card.tsx
│   │   │   │   └── category-grid.tsx
│   │   │   ├── chat/
│   │   │   │   ├── conversation-list.tsx
│   │   │   │   └── chat-bubble.tsx
│   │   │   ├── forms/
│   │   │   │   ├── post-ad-form.tsx
│   │   │   │   └── login-form.tsx
│   │   │   ├── layout/
│   │   │   │   ├── header.tsx
│   │   │   │   ├── footer.tsx
│   │   │   │   └── sidebar.tsx
│   │   │   └── admin/
│   │   │       ├── moderation-queue.tsx
│   │   │       └── stats-card.tsx
│   │   │
│   │   ├── lib/
│   │   │   ├── api/
│   │   │   │   ├── client.ts         # API client setup
│   │   │   │   ├── ads.ts
│   │   │   │   ├── auth.ts
│   │   │   │   ├── users.ts
│   │   │   │   └── categories.ts
│   │   │   ├── hooks/
│   │   │   │   ├── use-auth.ts
│   │   │   │   ├── use-ads.ts
│   │   │   │   └── use-chat.ts
│   │   │   ├── utils/
│   │   │   │   ├── cn.ts             # Class name merger
│   │   │   │   ├── format.ts         # Date, price formatters
│   │   │   │   └── validation.ts
│   │   │   └── constants.ts
│   │   │
│   │   ├── types/
│   │   │   ├── ad.ts
│   │   │   ├── user.ts
│   │   │   └── api.ts
│   │   │
│   │   ├── public/
│   │   │   ├── icons/
│   │   │   └── images/
│   │   │
│   │   ├── .env.local
│   │   ├── .eslintrc.json
│   │   ├── next.config.js
│   │   ├── package.json
│   │   ├── tailwind.config.ts
│   │   └── tsconfig.json
│   │
│   └── api/                          # NestJS Backend
│       ├── src/
│       │   ├── modules/
│       │   │   ├── auth/
│       │   │   │   ├── auth.controller.ts
│       │   │   │   ├── auth.service.ts
│       │   │   │   ├── auth.module.ts
│       │   │   │   ├── dto/
│       │   │   │   │   ├── send-otp.dto.ts
│       │   │   │   │   └── verify-otp.dto.ts
│       │   │   │   ├── guards/
│       │   │   │   │   └── jwt-auth.guard.ts
│       │   │   │   └── strategies/
│       │   │   │       └── jwt.strategy.ts
│       │   │   │
│       │   │   ├── users/
│       │   │   │   ├── users.controller.ts
│       │   │   │   ├── users.service.ts
│       │   │   │   ├── users.module.ts
│       │   │   │   ├── dto/
│       │   │   │   │   ├── create-user.dto.ts
│       │   │   │   │   └── update-user.dto.ts
│       │   │   │   └── entities/
│       │   │   │       └── user.entity.ts
│       │   │   │
│       │   │   ├── ads/
│       │   │   │   ├── ads.controller.ts
│       │   │   │   ├── ads.service.ts
│       │   │   │   ├── ads.module.ts
│       │   │   │   ├── dto/
│       │   │   │   │   ├── create-ad.dto.ts
│       │   │   │   │   ├── update-ad.dto.ts
│       │   │   │   │   └── search-ad.dto.ts
│       │   │   │   └── entities/
│       │   │   │       └── ad.entity.ts
│       │   │   │
│       │   │   ├── categories/
│       │   │   │   ├── categories.controller.ts
│       │   │   │   ├── categories.service.ts
│       │   │   │   ├── categories.module.ts
│       │   │   │   ├── dto/
│       │   │   │   └── entities/
│       │   │   │
│       │   │   ├── chat/
│       │   │   │   ├── chat.controller.ts
│       │   │   │   ├── chat.service.ts
│       │   │   │   ├── chat.module.ts
│       │   │   │   ├── dto/
│       │   │   │   └── entities/
│       │   │   │
│       │   │   └── admin/
│       │   │       ├── admin.controller.ts
│       │   │       ├── admin.service.ts
│       │   │       ├── admin.module.ts
│       │   │       ├── dto/
│       │   │       └── guards/
│       │   │           └── admin.guard.ts
│       │   │
│       │   ├── shared/
│       │   │   ├── database/
│       │   │   │   ├── database.module.ts
│       │   │   │   └── database.service.ts
│       │   │   ├── redis/
│       │   │   │   ├── redis.module.ts
│       │   │   │   └── redis.service.ts
│       │   │   ├── storage/
│       │   │   │   ├── storage.module.ts
│       │   │   │   └── storage.service.ts
│       │   │   ├── decorators/
│       │   │   │   ├── current-user.decorator.ts
│       │   │   │   └── roles.decorator.ts
│       │   │   ├── filters/
│       │   │   │   └── http-exception.filter.ts
│       │   │   ├── interceptors/
│       │   │   │   ├── logging.interceptor.ts
│       │   │   │   └── transform.interceptor.ts
│       │   │   ├── pipes/
│       │   │   │   └── validation.pipe.ts
│       │   │   └── utils/
│       │   │       ├── helpers.ts
│       │   │       └── constants.ts
│       │   │
│       │   ├── config/
│       │   │   ├── app.config.ts
│       │   │   ├── database.config.ts
│       │   │   ├── jwt.config.ts
│       │   │   └── storage.config.ts
│       │   │
│       │   ├── app.module.ts
│       │   └── main.ts
│       │
│       ├── prisma/
│       │   ├── schema.prisma
│       │   ├── seed.ts
│       │   └── migrations/
│       │
│       ├── test/
│       │   └── app.e2e-spec.ts
│       │
│       ├── .env
│       ├── .eslintrc.js
│       ├── nest-cli.json
│       ├── package.json
│       └── tsconfig.json
│
├── packages/
│   ├── ui/                           # Shared UI Components
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── button.tsx
│   │   │   │   ├── input.tsx
│   │   │   │   ├── card.tsx
│   │   │   │   ├── badge.tsx
│   │   │   │   ├── modal.tsx
│   │   │   │   └── index.ts
│   │   │   ├── styles/
│   │   │   │   └── globals.css
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── types/                        # Shared Types
│   │   ├── src/
│   │   │   ├── user.ts
│   │   │   ├── ad.ts
│   │   │   ├── category.ts
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── config/                       # Shared Config
│       ├── eslint-config/
│       │   └── index.js
│       └── tsconfig/
│           ├── base.json
│           ├── nextjs.json
│           └── nestjs.json
│
├── docs/
│   ├── architecture/
│   │   └── system-design.md
│   ├── api/
│   │   └── endpoints.md
│   ├── ux/
│   │   └── wireframes.md
│   └── ui/
│       └── design-system.md
│
├── .gitignore
├── .nvmrc
├── package.json                      # Root package.json
├── pnpm-workspace.yaml
├── turbo.json
├── README.md
└── docker-compose.yml                # For PostgreSQL + Redis
```

---

## Naming Conventions

### Files
- **Components:** `kebab-case` (e.g., `ad-card.tsx`)
- **Pages:** `kebab-case` (e.g., `page.tsx`, `layout.tsx`)
- **Utils/Helpers:** `kebab-case` (e.g., `format-date.ts`)
- **Types/Interfaces:** `PascalCase` in file content, `kebab-case` file name
- **Constants:** `SCREAMING_SNAKE_CASE`

### Folders
- **kebab-case** for all folders (e.g., `user-management`, `ad-detail`)

### Code
- **Components:** `PascalCase` (e.g., `AdCard`, `UserProfile`)
- **Functions:** `camelCase` (e.g., `formatPrice`, `getUserById`)
- **Hooks:** `camelCase` with `use` prefix (e.g., `useAuth`, `useAds`)
- **Types/Interfaces:** `PascalCase` (e.g., `User`, `AdResponse`)
- **Enums:** `PascalCase` (e.g., `AdStatus`, `UserRole`)

---

## Environment Variables

### `apps/web/.env.local`
```bash
# API
NEXT_PUBLIC_API_URL=http://localhost:3001/api/v1

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME="Plasma Lunar Marketplace"

# OAuth (Optional for MVP)
NEXT_PUBLIC_GOOGLE_CLIENT_ID=your_google_client_id

# Upload
NEXT_PUBLIC_MAX_FILE_SIZE=10485760
NEXT_PUBLIC_MAX_IMAGES=8
```

### `apps/api/.env`
```bash
# Server
NODE_ENV=development
PORT=3001
API_PREFIX=api/v1

# Database
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/marketplace?schema=public"

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# JWT
JWT_ACCESS_SECRET=your_super_secret_access_key_change_in_production
JWT_REFRESH_SECRET=your_super_secret_refresh_key_change_in_production
JWT_ACCESS_EXPIRATION=15m
JWT_REFRESH_EXPIRATION=7d

# Storage (Cloudflare R2 / S3)
STORAGE_ENDPOINT=https://your-account-id.r2.cloudflarestorage.com
STORAGE_ACCESS_KEY=your_access_key
STORAGE_SECRET_KEY=your_secret_key
STORAGE_BUCKET=marketplace-uploads
STORAGE_PUBLIC_URL=https://cdn.your-domain.com

# OTP (Twilio or local for dev)
OTP_PROVIDER=local
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
TWILIO_PHONE_NUMBER=+1234567890

# CORS
CORS_ORIGIN=http://localhost:3000

# Rate Limiting
THROTTLE_TTL=60
THROTTLE_LIMIT=100
```

### `docker-compose.yml`
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: marketplace-db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: marketplace
    ports:
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    container_name: marketplace-redis
    ports:
      - '6379:6379'
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

## Setup Commands

### 1. Repository Initialization
```bash
# Create project directory
mkdir plasma-lunar-marketplace
cd plasma-lunar-marketplace

# Initialize git
git init
git branch -M main

# Create .gitignore
cat > .gitignore << 'EOF'
# Dependencies
node_modules/
.pnp/
.pnp.js

# Environment
.env
.env.local
.env.*.local
apps/web/.env.local
apps/api/.env

# Build outputs
.next/
dist/
build/
out/

# Logs
*.log
npm-debug.log*
pnpm-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.idea/
.vscode/
*.swp
*.swo

# Testing
coverage/

# Prisma
apps/api/prisma/migrations/
EOF

# Create README
cat > README.md << 'EOF'
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
EOF
```

### 2. Monorepo Setup
```bash
# Initialize pnpm (if not installed: npm i -g pnpm)
pnpm init

# Create workspace config
cat > pnpm-workspace.yaml << 'EOF'
packages:
  - 'apps/*'
  - 'packages/*'
EOF

# Install Turborepo
pnpm add -D turbo

# Create turbo config
cat > turbo.json << 'EOF'
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {},
    "type-check": {}
  }
}
EOF

# Update root package.json
cat > package.json << 'EOF'
{
  "name": "plasma-lunar-marketplace",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "type-check": "turbo run type-check"
  },
  "devDependencies": {
    "turbo": "^1.11.0"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  }
}
EOF
```

### 3. Next.js Setup
```bash
# Create Next.js app
pnpm create next-app@latest apps/web \
  --typescript \
  --tailwind \
  --app \
  --src-dir=false \
  --import-alias="@/*" \
  --no-git

# Navigate to web app
cd apps/web

# Install additional dependencies
pnpm add axios @tanstack/react-query zustand
pnpm add -D @types/node

# Create .env.local
cat > .env.local << 'EOF'
NEXT_PUBLIC_API_URL=http://localhost:3001/api/v1
NEXT_PUBLIC_APP_URL=http://localhost:3000
EOF

cd ../..
```

### 4. NestJS Setup
```bash
# Create NestJS app
cd apps
npx @nestjs/cli new api --package-manager pnpm --skip-git

# Navigate to API
cd api

# Install dependencies
pnpm add @nestjs/config @nestjs/jwt @nestjs/passport passport passport-jwt
pnpm add @prisma/client class-validator class-transformer
pnpm add bcrypt
pnpm add -D @types/passport-jwt @types/bcrypt prisma

# Initialize Prisma
npx prisma init

# Create .env
cat > .env << 'EOF'
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/marketplace?schema=public"
JWT_ACCESS_SECRET=dev_access_secret_change_in_production
JWT_REFRESH_SECRET=dev_refresh_secret_change_in_production
EOF

cd ../..
```

### 5. Shared Packages Setup
```bash
# Create packages/ui
mkdir -p packages/ui/src/components
cd packages/ui

# Initialize package
cat > package.json << 'EOF'
{
  "name": "@marketplace/ui",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "lint": "eslint ."
  },
  "peerDependencies": {
    "react": "^18.0.0"
  }
}
EOF

cd ../..

# Create packages/types
mkdir -p packages/types/src
cd packages/types

cat > package.json << 'EOF'
{
  "name": "@marketplace/types",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "types": "./src/index.ts"
}
EOF

cd ../..
```

### 6. Docker Setup
```bash
# Create docker-compose.yml (already shown above)
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: marketplace-db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: marketplace
    ports:
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    container_name: marketplace-redis
    ports:
      - '6379:6379'
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
EOF

# Start services
docker-compose up -d
```

### 7. First Commit
```bash
git add .
git commit -m "Initial project setup: Next.js + NestJS monorepo"
```

---

## Development Rules (FROZEN)

### ❌ Prohibited
1. **No UX Changes:** All screens, flows, and user journeys are frozen per `ux_wireframe_spec.md`.
2. **No Feature Additions:** Only implement features defined in `implementation_plan.md` (Phase 1 MVP).
3. **No Design Deviations:** Follow `ui_design_system.md` exactly (colors, typography, spacing).
4. **No Schema Changes:** Database schema is frozen per `implementation_plan.md` unless critical bug.
5. **No Endpoint Additions:** API routes are defined in spec. No new routes without approval.

### ✅ Required
1. **Follow Naming Conventions:** Strict adherence to conventions above.
2. **TypeScript Strict Mode:** All code must pass `strict: true`.
3. **No `any` Types:** Explicit typing required.
4. **Component Isolation:** Each component in its own file.
5. **Error Handling:** All API calls must have try-catch with user-friendly messages.

### Code Review Checklist
- [ ] Matches wireframe spec exactly
- [ ] Uses design system colors/typography
- [ ] TypeScript strict, no `any`
- [ ] Responsive (mobile-first)
- [ ] Error states handled
- [ ] Loading states implemented
- [ ] No console logs in production code

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start all apps in dev mode |
| `pnpm --filter web dev` | Start Next.js only |
| `pnpm --filter api dev` | Start NestJS only |
| `pnpm build` | Build all apps |
| `pnpm lint` | Lint all code |
| `docker-compose up -d` | Start PostgreSQL + Redis |
| `docker-compose down` | Stop databases |

---

> **Status:** Ready for development kickoff.
