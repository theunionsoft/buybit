# Component Specification & Implementation Guide

> **Stack:** Next.js + Tailwind CSS + Framer Motion + Lucide Icons  
> **Theme:** Midnight Luxury (Dark Mode First)  
> **Location:** `packages/ui/src/components` & `apps/web/components`

---

## 1. Atoms (Base UI Lib)

### 1.1 Button (`ui/button.tsx`)

**Visuals:**
- **Primary:** "The Aurora" Gradient background, white text, subtle glow.
- **Secondary:** Transparent bg, 1px gradient border, text gradient.
- **Ghost:** Transparent, white text, hover bg-white/5.

**Props:**
```typescript
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost' | 'destructive';
  size?: 'sm' | 'md' | 'lg' | 'icon';
  loading?: boolean;
  leftIcon?: React.ReactNode;
}
```

**Tailwind Classes:**
- Base: `inline-flex items-center justify-center rounded-xl font-medium transition-all active:scale-95 disabled:opacity-50 disabled:pointer-events-none`
- Primary: `bg-gradient-to-r from-indigo-500 via-purple-500 to-rose-400 text-white shadow-lg shadow-indigo-500/25 hover:shadow-indigo-500/40 hover:scale-[1.02]`
- Secondary: `border border-white/10 bg-white/5 hover:bg-white/10 backdrop-blur-md text-white`
- Size MD: `h-11 px-6 text-sm`

**Micro-interaction:**
- Hover: Scale 1.02, Shadow expands.
- Click: Scale 0.96.

---

### 1.2 Input (`ui/input.tsx`)

**Visuals:**
- Floating label style or sleek placeholder.
- Glassmorphism background.

**Props:**
```typescript
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
  leftIcon?: React.ReactNode;
}
```

**Tailwind Classes:**
- Base: `w-full bg-white/5 border border-white/10 rounded-xl px-4 h-12 text-white placeholder:text-white/40 focus:outline-none focus:border-indigo-500/50 focus:ring-1 focus:ring-indigo-500/50 transition-all`
- Error: `border-rose-500/50 focus:border-rose-500`

---

### 1.3 Badge (`ui/badge.tsx`)

**Visuals:**
- Small pill, glass background, specific border color based on variant.

**Variants:**
- `luxury`: Gold/Rose border, gradient text.
- `verified`: Blue border, check icon.
- `urgent`: Red pulsating border.

**Tailwind Classes:**
- Base: `inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-semibold backdrop-blur-md`
- Luxury: `border border-rose-200/20 bg-rose-500/10 text-rose-200`

---

## 2. Molecules (Compound Components)

### 2.1 AdCard (`components/ads/ad-card.tsx`)

**Structure:**
```tsx
<div className="group relative overflow-hidden rounded-2xl bg-slate-900 border border-white/5">
  {/* Image Aspect 4:3 */}
  <div className="aspect-[4/3] w-full overflow-hidden bg-slate-800">
    <Image 
      src={image} 
      className="h-full w-full object-cover transition-transform duration-500 group-hover:scale-110"
    />
    <Badge className="absolute top-3 left-3">Featured</Badge>
    <Button variant="icon" className="absolute top-3 right-3 bg-black/40 backdrop-blur">
      <HeartIcon />
    </Button>
  </div>
  
  {/* Content */}
  <div className="p-4 space-y-2">
    <div className="flex justify-between items-start">
      <h3 className="font-heading text-lg font-bold text-white group-hover:text-indigo-400 transition-colors">
        {title}
      </h3>
      <p className="font-mono text-lg font-bold text-rose-200">{price}</p>
    </div>
    <div className="flex items-center text-xs text-white/60 space-x-2">
      <span>{location}</span>
      <span>•</span>
      <span>{timeAgo}</span>
    </div>
  </div>
</div>
```

**Micro-interaction:**
- Parent Hover: Image zooms, Title changes color, Card lifts y (-4px).
- Heart Click: Confetti explosion.

---

### 2.2 CategoryCard (`components/category/category-card.tsx`)

**Visuals:**
- Square or rectangular card.
- Icon centered or top-left.
- Gradient border on hover.

**Tailwind Classes:**
- Container: `flex flex-col items-center justify-center p-6 rounded-2xl bg-white/5 border border-white/5 hover:border-indigo-500/50 hover:bg-white/10 transition-all cursor-pointer group`
- Icon: `w-12 h-12 text-indigo-400 group-hover:scale-110 transition-transform`

---

### 2.3 SearchBar (`components/layout/header/search-bar.tsx`)

**Visuals:**
- Large pill shape.
- "The Hero Input" on homepage.

**Tailwind Classes:**
- Container: `relative w-full max-w-2xl`
- Input: `w-full h-14 pl-12 pr-32 rounded-full bg-white/10 border border-white/10 focus:bg-white/15 focus:border-indigo-500/50 backdrop-blur-xl text-lg text-white placeholder:text-white/50 transition-all`
- Button: `absolute right-2 top-2 bottom-2` (Primary Gradient)

---

## 3. Organisms (Complex Sections)

### 3.1 Header/Navbar (`components/layout/header.tsx`)

**Behavior:**
- Sticky, Glassmorphism.
- Hides/Shows on scroll (optional).

**Structure:**
```tsx
<header className="sticky top-0 z-50 w-full border-b border-white/5 bg-slate-950/80 backdrop-blur-xl supports-[backdrop-filter]:bg-slate-950/60">
  <div className="container flex h-16 items-center">
    <Logo />
    <DesktopNav />
    <MobileNavToggle />
    <div className="ml-auto flex items-center space-x-4">
      <SearchTrigger />
      <UserMenu />
      <Button variant="primary">Sell Item</Button>
    </div>
  </div>
</header>
```

---

### 3.2 ChatWindow (`components/chat/window.tsx`)

**Visuals:**
- Sidebar left (conversations), Main right (messages).
- Mobile: Full screen, switch between list/detail.

**Components:**
- `MessageBubble`:
  - Self: `bg-gradient-to-r from-indigo-600 to-purple-600 text-white rounded-t-2xl rounded-bl-2xl ml-auto`
  - Other: `bg-white/10 text-white rounded-t-2xl rounded-br-2xl`

---

## 4. Screen Implementation Guide

### 4.1 Post Ad Wizard (`/post-ad`)

**State Management:**
- Use `zustand` store `usePostAdStore` to hold form state across steps.

**Steps:**
1. **Category:** Grid of `CategoryCard`. Click -> set category -> auto advance.
2. **Details:** specific form fields based on category.
3. **Images:** Drag & drop zone. Preview grid.
4. **Review:** Read-only summary card.

**Animations:**
- `AnimatePresence` (framer-motion) for step transitions (Slide left/right).

### 4.2 Ad Detail Page (`/ad/[slug]`)

**Layout:**
- **Desktop:** 2 Columns. Left (66%): Gallery + Desc. Right (33%): Price + Seller + Actions (Sticky).
- **Mobile:** Vertical stack. Sticky bottom bar for "Chat/Call".

**Gallery:**
- Main image with aspect-ratio.
- Scrollable thumbnail strip below.

---

## 5. Premium Copy & Labels

### Buttons
- `Post Ad` -> **Sell Item**
- `Chat` -> **Make Offer**
- `Search` -> **Discover**

### Empty States
- **Favorites:** "Your Collection is Empty. Curate your luxury wishlist."
- **Messages:** "Quiet Airwaves. Start a negotiation to see messages here."

### Headings
- **Featured:** "Curated Selection"
- **Trending:** "In High Demand"
- **New:** "Fresh Arrivals"

---

## 6. Development Checklist

- [ ] **Mobile First:** Always start writing classes for mobile, then `md:` for desktop.
- [ ] **Touch Targets:** Ensure interactive elements are `min-h-[44px]` and `min-w-[44px]`.
- [ ] **Strict Types:** Define props interfaces for *every* component.
- [ ] **Client Components:** Add `'use client'` directive only where interaction is needed.
- [ ] **Images:** Use `next/image` with `placeholder="blur"`.
- [ ] **Icons:** Use `lucide-react` with stroke width `1.5` for elegance.

---

> **Note:** This spec is to be used in conjunction with `ui_design_system.md` for specific color hex codes and font families.
