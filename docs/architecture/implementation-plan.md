# Plasma Lunar Marketplace - MVP Development Specification

> **Document Type:** Development Handoff Specification  
> **Version:** 2.0 (Post-CTO Review)  
> **Timeline:** 8 Weeks  
> **Ready For:** Development Company

---

## Executive Summary

Build a classified ads marketplace (OLX-style) with:
- **Phase 1 (This Doc):** Core ads, search, chat, admin moderation
- **Phase 2 (Later):** Payments, boosts, featured listings
- **Phase 3 (Later):** Multi-vendor shops, ratings, analytics

---

## 1. Tech Stack (Final)

| Layer | Technology | Notes |
|-------|------------|-------|
| **Frontend** | Next.js 14 (App Router) | TypeScript, SSR for SEO |
| **Backend** | NestJS (Modular Monolith) | TypeScript, REST API |
| **Database** | PostgreSQL 15 | With pg_trgm for search |
| **Cache** | Redis 7 | Sessions, rate limiting |
| **Storage** | Cloudflare R2 | Image storage, CDN |
| **Auth** | JWT + OTP (Twilio/local) | Phone-first |
| **Hosting** | Single VPS (4 vCPU, 8GB) | DigitalOcean/Hetzner |

---

## 2. Database Schema (MVP)

### 2.1 Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone VARCHAR(15) UNIQUE NOT NULL,
  phone_verified BOOLEAN DEFAULT FALSE,
  email VARCHAR(255) UNIQUE,
  password_hash VARCHAR(255),
  name VARCHAR(100),
  avatar_url VARCHAR(500),
  role VARCHAR(20) DEFAULT 'user' CHECK (role IN ('user', 'admin', 'super_admin')),
  status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'suspended', 'banned')),
  is_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  last_login_at TIMESTAMP
);

CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_status ON users(status);
```

### 2.2 Categories Table
```sql
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  parent_id INTEGER REFERENCES categories(id) ON DELETE CASCADE,
  name VARCHAR(100) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  icon_url VARCHAR(500),
  display_order INTEGER DEFAULT 0,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_categories_parent ON categories(parent_id);
CREATE INDEX idx_categories_slug ON categories(slug);
```

### 2.3 Ads Table
```sql
CREATE TABLE ads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  category_id INTEGER NOT NULL REFERENCES categories(id),
  title VARCHAR(150) NOT NULL,
  slug VARCHAR(200) UNIQUE NOT NULL,
  description TEXT,
  price DECIMAL(12,2),
  price_type VARCHAR(20) DEFAULT 'fixed' CHECK (price_type IN ('fixed', 'negotiable', 'contact')),
  condition VARCHAR(20) CHECK (condition IN ('new', 'used_like_new', 'used_good', 'used_fair')),
  city VARCHAR(100) NOT NULL,
  area VARCHAR(100),
  status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('draft', 'pending', 'active', 'sold', 'expired', 'rejected', 'removed')),
  rejection_reason TEXT,
  views_count INTEGER DEFAULT 0,
  favorites_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP DEFAULT (NOW() + INTERVAL '30 days'),
  approved_at TIMESTAMP,
  
  -- Full-text search
  search_vector TSVECTOR GENERATED ALWAYS AS (
    setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
    setweight(to_tsvector('english', coalesce(description, '')), 'B')
  ) STORED
);

CREATE INDEX idx_ads_user ON ads(user_id);
CREATE INDEX idx_ads_category ON ads(category_id);
CREATE INDEX idx_ads_status ON ads(status);
CREATE INDEX idx_ads_city ON ads(city);
CREATE INDEX idx_ads_created ON ads(created_at DESC);
CREATE INDEX idx_ads_search ON ads USING GIN(search_vector);
```

### 2.4 Ad Images Table
```sql
CREATE TABLE ad_images (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ad_id UUID NOT NULL REFERENCES ads(id) ON DELETE CASCADE,
  url VARCHAR(500) NOT NULL,
  thumbnail_url VARCHAR(500),
  display_order INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_ad_images_ad ON ad_images(ad_id);
```

### 2.5 Favorites Table
```sql
CREATE TABLE favorites (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  ad_id UUID NOT NULL REFERENCES ads(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, ad_id)
);

CREATE INDEX idx_favorites_user ON favorites(user_id);
```

### 2.6 Conversations Table
```sql
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ad_id UUID NOT NULL REFERENCES ads(id) ON DELETE CASCADE,
  buyer_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  seller_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  last_message_at TIMESTAMP DEFAULT NOW(),
  buyer_unread_count INTEGER DEFAULT 0,
  seller_unread_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(ad_id, buyer_id)
);

CREATE INDEX idx_conversations_buyer ON conversations(buyer_id);
CREATE INDEX idx_conversations_seller ON conversations(seller_id);
```

### 2.7 Messages Table
```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  sender_id UUID NOT NULL REFERENCES users(id),
  content TEXT NOT NULL,
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_messages_conversation ON messages(conversation_id, created_at);
```

### 2.8 Reports Table
```sql
CREATE TABLE reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reporter_id UUID NOT NULL REFERENCES users(id),
  reported_type VARCHAR(20) NOT NULL CHECK (reported_type IN ('ad', 'user')),
  reported_id UUID NOT NULL,
  reason VARCHAR(50) NOT NULL,
  description TEXT,
  status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'reviewed', 'actioned', 'dismissed')),
  admin_notes TEXT,
  reviewed_by UUID REFERENCES users(id),
  reviewed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_reports_status ON reports(status);
```

### 2.9 OTP Table
```sql
CREATE TABLE otp_codes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone VARCHAR(15) NOT NULL,
  code VARCHAR(6) NOT NULL,
  purpose VARCHAR(20) NOT NULL CHECK (purpose IN ('login', 'register', 'reset')),
  expires_at TIMESTAMP NOT NULL,
  used_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_otp_phone ON otp_codes(phone, purpose);
```

### 2.10 Admin Audit Log
```sql
CREATE TABLE admin_audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  admin_id UUID NOT NULL REFERENCES users(id),
  action VARCHAR(50) NOT NULL,
  target_type VARCHAR(20) NOT NULL,
  target_id UUID NOT NULL,
  details JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_audit_admin ON admin_audit_logs(admin_id);
CREATE INDEX idx_audit_created ON admin_audit_logs(created_at DESC);
```

---

## 3. API Endpoints (MVP)

### 3.1 Authentication
```
POST   /api/v1/auth/send-otp        # Send OTP to phone
POST   /api/v1/auth/verify-otp      # Verify OTP, return JWT
POST   /api/v1/auth/refresh         # Refresh access token
POST   /api/v1/auth/logout          # Invalidate refresh token
POST   /api/v1/auth/google          # Google OAuth login
```

### 3.2 Users
```
GET    /api/v1/users/me             # Get current user profile
PUT    /api/v1/users/me             # Update profile
GET    /api/v1/users/:id            # Get public profile
GET    /api/v1/users/me/ads         # My posted ads
GET    /api/v1/users/me/favorites   # My saved ads
```

### 3.3 Categories
```
GET    /api/v1/categories           # List all (tree structure)
GET    /api/v1/categories/:slug     # Single category with children
```

### 3.4 Ads
```
GET    /api/v1/ads                  # List/search ads (paginated)
POST   /api/v1/ads                  # Create new ad
GET    /api/v1/ads/:slug            # Get single ad
PUT    /api/v1/ads/:id              # Update ad (owner only)
DELETE /api/v1/ads/:id              # Delete ad (owner only)
POST   /api/v1/ads/:id/mark-sold    # Mark as sold
POST   /api/v1/ads/:id/favorite     # Toggle favorite
POST   /api/v1/ads/:id/report       # Report ad
POST   /api/v1/ads/upload-image     # Upload image (returns URL)
```

### 3.5 Chat
```
GET    /api/v1/chat/conversations           # My conversations
POST   /api/v1/chat/conversations           # Start conversation (ad_id)
GET    /api/v1/chat/conversations/:id       # Get messages
POST   /api/v1/chat/conversations/:id       # Send message
GET    /api/v1/chat/unread-count            # Total unread
```

### 3.6 Admin Endpoints
```
# Dashboard
GET    /api/v1/admin/stats                  # Overview stats

# Moderation
GET    /api/v1/admin/ads/pending            # Ads awaiting approval
GET    /api/v1/admin/ads/reported           # Reported ads
PUT    /api/v1/admin/ads/:id/approve        # Approve ad
PUT    /api/v1/admin/ads/:id/reject         # Reject with reason
PUT    /api/v1/admin/ads/:id/remove         # Remove ad

# Users
GET    /api/v1/admin/users                  # List users
GET    /api/v1/admin/users/:id              # User detail
PUT    /api/v1/admin/users/:id/suspend      # Suspend user
PUT    /api/v1/admin/users/:id/ban          # Ban user
PUT    /api/v1/admin/users/:id/activate     # Reactivate

# Categories
POST   /api/v1/admin/categories             # Create category
PUT    /api/v1/admin/categories/:id         # Update category
DELETE /api/v1/admin/categories/:id         # Delete category
PUT    /api/v1/admin/categories/reorder     # Reorder categories

# Reports
GET    /api/v1/admin/reports                # List reports
PUT    /api/v1/admin/reports/:id            # Update report status
```

---

## 4. User Flows (MVP)

### 4.1 Post an Ad (3 Steps)
```
Step 1: Select Category
  └── Show L1 categories → User selects → Show L2 → User selects

Step 2: Add Details
  ├── Title (required, max 150 chars)
  ├── Description (optional, max 4000 chars)
  ├── Price (required) + Price Type (fixed/negotiable/contact)
  ├── Condition (new/used_like_new/used_good/used_fair)
  ├── City (required, dropdown)
  ├── Area (optional, text)
  └── Images (min 1, max 8, drag to reorder)

Step 3: Preview & Submit
  └── Review → Submit → Show "Pending Approval" message
```

### 4.2 Search & Browse
```
Homepage
├── Search bar (full-text)
├── City selector (top bar)
├── Category grid (L1)
└── Recent ads feed

Category Page
├── Breadcrumb (Home > Category)
├── Subcategory tabs (L2)
├── Filter sidebar (price range, condition)
├── Sort (newest, price low/high)
└── Ad grid (paginated, 20/page)

Ad Detail Page
├── Image gallery (swipe)
├── Title, price, condition badge
├── Seller card (name, member since, ad count)
├── Description
├── Location map (static)
├── "Chat with Seller" button
└── Similar ads carousel
```

### 4.3 Chat Flow
```
1. Buyer clicks "Chat" on ad
2. System creates/resumes conversation
3. Buyer sends first message
4. Seller sees notification + inbox update
5. Messages fetched via polling (5s interval)
```

---

## 5. Admin Dashboard (MVP)

### 5.1 Sidebar Navigation
```
├── Dashboard (overview stats)
├── Moderation
│   ├── Pending Ads (count badge)
│   ├── Reported Ads
│   └── Reported Users
├── Users
│   ├── All Users
│   └── Banned Users
├── Categories
│   └── Manage Categories
└── Settings
    └── Keyword Blacklist
```

### 5.2 Dashboard Stats Cards
```
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ Total Users │ │ Active Ads  │ │ Pending Ads │ │ Reports     │
│   1,234     │ │   5,678     │ │     23      │ │     5       │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
```

### 5.3 Moderation Queue UI
```
┌────────────────────────────────────────────────────────────┐
│ [Image] │ Title                    │ Seller    │ Actions   │
├─────────┼──────────────────────────┼───────────┼───────────┤
│ 📷      │ iPhone 14 Pro Max        │ Ali Khan  │ ✅ ❌ 👁  │
│ 📷      │ Honda Civic 2020         │ Sara Ali  │ ✅ ❌ 👁  │
└────────────────────────────────────────────────────────────┘

✅ = Approve    ❌ = Reject (opens modal for reason)    👁 = Preview
```

### 5.4 Category Manager UI
```
┌───────────────────────────────────────────────────────────┐
│ 📁 Electronics                          [Edit] [Delete]   │
│   ├── 📁 Mobile Phones                  [Edit] [Delete]   │
│   ├── 📁 Laptops                        [Edit] [Delete]   │
│   └── 📁 Tablets                        [Edit] [Delete]   │
│ 📁 Vehicles                             [Edit] [Delete]   │
│   ├── 📁 Cars                           [Edit] [Delete]   │
│   └── 📁 Motorcycles                    [Edit] [Delete]   │
└───────────────────────────────────────────────────────────┘
                                        [+ Add Category]
```

---

## 6. Security & Rate Limiting

| Endpoint Pattern | Rate Limit | Window |
|------------------|------------|--------|
| `/auth/send-otp` | 3 requests | 1 hour |
| `/auth/*` | 10 requests | 1 min |
| `/ads (POST)` | 5 requests | 1 hour |
| `/chat/* (POST)` | 30 requests | 1 min |
| `/admin/*` | 100 requests | 1 min |
| `*` (default) | 100 requests | 1 min |

---

## 7. Image Upload Specs

| Spec | Value |
|------|-------|
| Max file size | 10 MB |
| Allowed types | JPEG, PNG, WebP |
| Max images per ad | 8 |
| Storage format | WebP (converted) |
| Thumbnail size | 300x300 |
| Full size | Max 1200px width |

---

## 8. Seeded Data (Required at Launch)

### 8.1 Categories (Minimum)
```
Electronics
├── Mobile Phones
├── Laptops & Computers
├── Tablets
└── Accessories

Vehicles
├── Cars
├── Motorcycles
└── Auto Parts

Property
├── Houses
├── Apartments
└── Plots & Land

Fashion
├── Men's Clothing
├── Women's Clothing
└── Watches

Home & Garden
├── Furniture
├── Appliances
└── Decor
```

### 8.2 Cities (Pakistan - Top 10)
```
Lahore, Karachi, Islamabad, Rawalpindi, Faisalabad,
Multan, Peshawar, Quetta, Sialkot, Gujranwala
```

---

## 9. Deployment Checklist

- [ ] VPS provisioned (4 vCPU, 8GB RAM)
- [ ] PostgreSQL 15 installed
- [ ] Redis 7 installed
- [ ] Nginx configured (SSL, reverse proxy)
- [ ] Cloudflare R2 bucket created
- [ ] Twilio/SMS gateway configured
- [ ] Google OAuth app created
- [ ] Domain DNS configured
- [ ] SSL certificate (Let's Encrypt)
- [ ] PM2 for Node process management
- [ ] Backup cron job (daily)

---

## 10. Success Criteria (Week 8)

| Metric | Target |
|--------|--------|
| User registration working | ✅ |
| Ad posting with images | ✅ |
| Search returning results | ✅ |
| Chat sending/receiving | ✅ |
| Admin can approve/reject | ✅ |
| Admin can manage categories | ✅ |
| Mobile responsive | ✅ |
| Page load < 3 seconds | ✅ |

---

## Appendix: File Structure

```
/plasma-lunar-marketplace
├── /apps
│   ├── /web                    # Next.js frontend
│   │   ├── /app
│   │   │   ├── /(public)       # Public routes
│   │   │   ├── /(auth)         # Auth routes
│   │   │   ├── /(dashboard)    # User dashboard
│   │   │   └── /admin          # Admin dashboard
│   │   ├── /components
│   │   └── /lib
│   │
│   └── /api                    # NestJS backend
│       ├── /src
│       │   ├── /modules
│       │   │   ├── /auth
│       │   │   ├── /users
│       │   │   ├── /ads
│       │   │   ├── /categories
│       │   │   ├── /chat
│       │   │   └── /admin
│       │   ├── /shared
│       │   └── /config
│       └── /prisma
│
├── /packages
│   └── /shared                 # Shared types, utils
│
└── docker-compose.yml          # Local dev setup
```

---

> **Document Ready For:** Development Team Handoff  
> **Estimated Timeline:** 8 Weeks  
> **Next Phase:** Monetization (Payments, Boosts, Featured)

