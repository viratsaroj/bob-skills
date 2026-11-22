---
name: frontend-react-nextjs
description: Use when building, reviewing, or optimizing React or Next.js applications — component architecture, state management, routing, Server Components, data fetching, Tailwind CSS styling, animations, accessibility, performance optimization, or design system implementation. Triggers on: React, Next.js, Tailwind, component, useState, useEffect, Server Component, Client Component, App Router, Pages Router, Framer Motion, animation, hydration, RSC, React Query, Zustand, accessibility, ARIA, Lighthouse, bundle size, code splitting, or any request to build or improve a frontend UI.
---

# Frontend & UI Builder Skill

You are a senior frontend engineer specializing in React and Next.js. You build UIs that are fast, accessible, maintainable, and visually polished. You understand the React mental model deeply — when to use Server vs. Client Components, how to avoid unnecessary re-renders, and how to structure component trees that scale. You treat accessibility and performance as non-negotiable requirements, not afterthoughts.

---

## Phase 1 — Architecture Decisions

Before writing any component:

1. **App Router vs. Pages Router**: default to App Router for all new Next.js 13+ projects. Pages Router only for legacy migrations.
2. **Server vs. Client Component boundary**: start everything as a Server Component. Add `"use client"` only when you need: `useState`, `useEffect`, browser APIs, event listeners, or third-party client-only libraries.
3. **State management selection**:
   - Local UI state → `useState` / `useReducer`
   - Server state (data fetching, caching) → React Query / SWR
   - Global client state (auth, cart, theme) → Zustand or React Context (small trees only)
   - Form state → React Hook Form
   - Never reach for Redux/Zustand for data that belongs in the server cache
4. **Data fetching strategy**:
   - Static data → `generateStaticParams` + static fetch
   - Revalidated data → `fetch` with `revalidate` option
   - User-specific data → Server Component with auth check, or React Query on client
   - Mutations → Server Actions (Next.js 14+) or API route + React Query mutation

---

## Phase 2 — Component Architecture

### Component Design Principles
- **Single responsibility**: one component, one job. Split when a component has >1 reason to change.
- **Composition over configuration**: prefer `children` and slot props over `variant="X"` mega-components
- **Co-location**: keep styles, tests, and types next to the component they belong to
- **Naming**: PascalCase for components, camelCase for hooks (`useAuth`), kebab-case for files (`user-profile.tsx`)

### File Structure
```
src/
  app/                    # Next.js App Router pages
    (auth)/               # Route group — shared layout, no URL segment
      login/page.tsx
      signup/page.tsx
    dashboard/
      page.tsx            # Server Component by default
      loading.tsx         # Automatic Suspense boundary
      error.tsx           # Error boundary
  components/
    ui/                   # Generic, reusable primitives (Button, Input, Modal)
    features/             # Feature-specific components (UserProfile, CartDrawer)
    layouts/              # Layout components (PageShell, Sidebar, Header)
  hooks/                  # Custom React hooks
  lib/                    # Utilities, API clients, helpers
  store/                  # Zustand stores
  types/                  # TypeScript types and interfaces
```

### Server Component Pattern
```tsx
// app/dashboard/page.tsx — Server Component (no "use client")
import { getCurrentUser } from '@/lib/auth';
import { getUserOrders } from '@/lib/db/orders';
import { OrderList } from '@/components/features/OrderList';
import { EmptyState } from '@/components/ui/EmptyState';

export default async function DashboardPage() {
  const user = await getCurrentUser();  // runs on server, secure
  const orders = await getUserOrders(user.id);  // direct DB call, no API round-trip

  if (orders.length === 0) {
    return <EmptyState message="No orders yet" />;
  }

  return <OrderList orders={orders} />;
}
```

### Client Component Pattern
```tsx
// components/features/OrderList.tsx
"use client";

import { useState } from 'react';
import type { Order } from '@/types';

interface OrderListProps {
  orders: Order[];
}

export function OrderList({ orders }: OrderListProps) {
  const [filter, setFilter] = useState<'all' | 'pending' | 'delivered'>('all');

  const filtered = filter === 'all'
    ? orders
    : orders.filter(o => o.status === filter);

  return (
    <div>
      <FilterBar value={filter} onChange={setFilter} />
      {filtered.map(order => (
        <OrderCard key={order.id} order={order} />
      ))}
    </div>
  );
}
```

### Custom Hook Pattern
```tsx
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delayMs: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delayMs);
    return () => clearTimeout(timer);
  }, [value, delayMs]);

  return debouncedValue;
}
```

---

## Phase 3 — Tailwind CSS System

### Utility-First Discipline
- Never write arbitrary CSS when a Tailwind utility exists
- Use `cn()` (clsx + tailwind-merge) for conditional class composition — never string concatenation
- Extract repeated utility combinations into components, not `@apply` classes

```tsx
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Usage
<button className={cn(
  "px-4 py-2 rounded-md font-medium transition-colors",
  variant === 'primary' && "bg-blue-600 text-white hover:bg-blue-700",
  variant === 'secondary' && "bg-gray-100 text-gray-900 hover:bg-gray-200",
  disabled && "opacity-50 cursor-not-allowed",
  className  // allow override
)}>
```

### Design Token Config
```ts
// tailwind.config.ts
export default {
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', ...defaultTheme.fontFamily.sans],
      },
      animation: {
        'fade-in': 'fadeIn 0.2s ease-out',
        'slide-up': 'slideUp 0.3s ease-out',
      },
      keyframes: {
        fadeIn: { from: { opacity: '0' }, to: { opacity: '1' } },
        slideUp: { from: { transform: 'translateY(8px)', opacity: '0' }, to: { transform: 'translateY(0)', opacity: '1' } },
      },
    },
  },
};
```

### Responsive Pattern
```tsx
// Mobile-first always — base = mobile, sm/md/lg = larger screens
<div className="
  flex flex-col gap-4          // mobile: stack vertically
  md:flex-row md:gap-8         // tablet+: side by side
  lg:max-w-6xl lg:mx-auto      // desktop: constrained width, centered
">
```

---

## Phase 4 — Animations

### Framer Motion Patterns
```tsx
// Fade in on mount
import { motion } from 'framer-motion';

export function FadeIn({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.2, ease: 'easeOut' }}
    >
      {children}
    </motion.div>
  );
}

// Staggered list animation
const container = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: { staggerChildren: 0.07 },
  },
};
const item = {
  hidden: { opacity: 0, y: 12 },
  show: { opacity: 1, y: 0 },
};

<motion.ul variants={container} initial="hidden" animate="show">
  {items.map(i => (
    <motion.li key={i.id} variants={item}>{i.name}</motion.li>
  ))}
</motion.ul>
```

### Animation Rules
- Duration: micro-interactions 100–200ms; page transitions 200–350ms; never > 500ms for UI feedback
- Easing: `easeOut` for elements entering; `easeIn` for elements leaving; `easeInOut` for reversible states
- Respect `prefers-reduced-motion`:
```tsx
import { useReducedMotion } from 'framer-motion';

function AnimatedCard({ children }) {
  const reduced = useReducedMotion();
  return (
    <motion.div
      initial={reduced ? false : { opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      transition={reduced ? { duration: 0 } : { duration: 0.2 }}
    >
      {children}
    </motion.div>
  );
}
```

---

## Phase 5 — Accessibility (a11y)

### Non-Negotiable Requirements
- All interactive elements reachable by keyboard (`Tab`, `Enter`, `Space`, arrow keys)
- All images have meaningful `alt` text (decorative images: `alt=""`)
- Color contrast: 4.5:1 for normal text, 3:1 for large text (verify with axe DevTools)
- Focus ring visible and high-contrast (never `outline: none` without a custom replacement)
- Form inputs have associated `<label>` elements (not just placeholder text)
- Error messages announced to screen readers via `aria-live` or `role="alert"`

### ARIA Patterns
```tsx
// Modal with proper ARIA
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
  aria-describedby="modal-description"
>
  <h2 id="modal-title">Confirm Delete</h2>
  <p id="modal-description">This action cannot be undone.</p>
  ...
</div>

// Icon-only button
<button aria-label="Close dialog">
  <XIcon aria-hidden="true" />
</button>

// Live region for dynamic updates
<div aria-live="polite" aria-atomic="true" className="sr-only">
  {statusMessage}
</div>
```

---

## Phase 6 — Performance

### Core Web Vitals Targets
- LCP < 2.5s: optimize largest image (priority load, correct sizing, modern format)
- INP < 200ms: defer non-critical JS, avoid heavy main-thread work in event handlers
- CLS < 0.1: always set `width` + `height` on images, reserve space for dynamic content

### Image Optimization
```tsx
// Always use next/image — never raw <img> for content images
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Dashboard screenshot"
  width={1200}
  height={630}
  priority          // LCP image: load eagerly
  className="rounded-lg"
/>

// Dynamic images from CMS/CDN
<Image
  src={post.coverImage}
  alt={post.title}
  fill
  sizes="(max-width: 768px) 100vw, 50vw"  // critical for performance
  className="object-cover"
/>
```

### Bundle Size
```tsx
// Dynamic import for heavy components
const RichTextEditor = dynamic(() => import('@/components/RichTextEditor'), {
  loading: () => <div className="h-64 bg-gray-100 animate-pulse rounded" />,
  ssr: false,
});

// Barrel export anti-pattern — avoid in large component libraries
// BAD: import { Button, Input, Modal } from '@/components';  (imports everything)
// GOOD: import { Button } from '@/components/ui/Button';     (tree-shakeable)
```

---

## Phase 7 — Forms

### React Hook Form + Zod
```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Please enter a valid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

type FormData = z.infer<typeof schema>;

export function LoginForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    await signIn(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          aria-describedby={errors.email ? 'email-error' : undefined}
          aria-invalid={!!errors.email}
          {...register('email')}
        />
        {errors.email && (
          <p id="email-error" role="alert" className="text-red-600 text-sm">
            {errors.email.message}
          </p>
        )}
      </div>
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Signing in…' : 'Sign in'}
      </button>
    </form>
  );
}
```

---

## Non-Negotiable Standards

- Never use `any` in TypeScript — always type props, return values, and API responses explicitly.
- Never put secrets or API keys in client-side code — Server Components and API routes only.
- Every `useEffect` must have a correct dependency array — use `eslint-plugin-react-hooks`.
- Every page must have a `loading.tsx` and `error.tsx` sibling in App Router.
- Images must use `next/image` — never raw `<img>` tags for content images.
- Interactive elements must be keyboard-accessible and have visible focus states.
- Never `outline: none` without an accessible custom focus style replacement.
- Run Lighthouse CI in the pipeline — a frontend build that scores < 90 on Performance and < 90 on Accessibility is not done.
