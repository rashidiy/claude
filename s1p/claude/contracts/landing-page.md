# Contract: Landing Page (Public Marketing/Sales Page)

**Status:** done
**Date:** 2026-03-15
**Affects:** frontend
**Branch:** `feat/landing-page` (from `dev`)
**Skill:** Use `/frontend-design` skill for all component generation

---

## Overview

Build a premium public landing page for S1P — a CRM with Telegram bot integration for call centers in Uzbekistan/CIS. This page is the first thing potential customers see. It must look like a $10K+ professionally designed page, not a template.

**Target audience:**
- Call center managers in Uzbekistan (primary buyer, lives in Telegram)
- Company directors/owners (signs the check, cares about ROI)
- IT person (does integration, needs to see it's legit)

**Languages:** Russian (default), Uzbek, English — using existing next-intl setup.

---

## Pre-Implementation Setup

### 1. Create branch
```bash
cd repos/s1p-frontend
git checkout dev
git pull origin dev
git checkout -b feat/landing-page
```

### 2. Install dependencies
```bash
npm install framer-motion tailwindcss-motion tailwindcss-animate
```

### 3. Update tailwind.config.ts
Add plugins:
```typescript
plugins: [
  require('tailwindcss-motion'),
  require('tailwindcss-animate'),
],
```

### 4. No additional component libraries to install
Magic UI and Aceternity UI are copy-paste — grab individual component code as needed from:
- https://magicui.design
- https://ui.aceternity.com

---

## Design Direction

### Visual Language
- **White background** with subtle warm gray (#FAFAFA) sections for contrast
- **Primary accent:** Indigo/blue (from existing CRM theme — connect landing to product)
- **Secondary accent:** Teal or green for CTAs and success states
- **NOT dark mode** — CIS B2B market expects professional/corporate clean look
- **NOT bold/maximalist** — clean, premium, generous whitespace
- **Font:** Keep existing project font or use Inter (standard for premium SaaS)
- **Corner radius:** 12-16px on cards (modern, not sharp, not bubbly)

### Animation Philosophy
- **Every animation must answer "why does this move?"**
- Entrance animations: subtle fade-up + scale on scroll (tailwindcss-motion)
- Hero animation: scroll-triggered split-screen sequence (Framer Motion)
- ROI calculator: spring physics on number changes (Framer Motion)
- NO decorative parallax, NO carousels/sliders, NO auto-playing video
- Respect `prefers-reduced-motion` — disable all animations when set

### Anti-Patterns to Avoid
- Stock photos of fake teams
- Generic headlines ("Save time and money")
- Multiple competing CTAs on one screen
- Sliders/carousels (proven conversion killers)
- AI-generated text left unedited (bland, interchangeable phrasing)
- Excessive bright colors without hierarchy
- Abstract illustrations disconnected from actual product

---

## Page Structure (Top to Bottom)

### Navigation Bar (Sticky)
- Logo (left)
- Language toggle: RU / UZ / EN (right)
- Phone number: +998 XX XXX XX XX (right, visible on desktop)
- CTA button: "Попробовать бесплатно" (right)
- On mobile: hamburger menu with language + CTA

### Section 1: Hero — Split-Screen Animated Demo

**This is the money shot.** Two device mockups side by side, with a scroll-triggered or auto-playing animation sequence.

**Layout:**
```
[Headline + Subtitle + CTA]         (centered above on mobile, left on desktop)

[Phone Mockup: Telegram]    [Browser Mockup: CRM Dashboard]
```

**Headline (RU):** "Ни один звонок не останется без внимания"
**Headline (UZ):** "Birorta qo'ng'iroq e'tiborsiz qolmaydi"
**Headline (EN):** "Never miss a customer call again"

**Subtitle (RU):** "CRM + Telegram бот для колл-центров. Мгновенные уведомления, управление лидами, аналитика — всё в одном месте."
**Subtitle (UZ):** "Qo'ng'iroq markazlari uchun CRM + Telegram bot. Tezkor bildirishnomalar, lidlarni boshqarish, analitika — barchasi bir joyda."
**Subtitle (EN):** "CRM + Telegram bot for call centers. Instant notifications, lead management, analytics — all in one place."

**CTA Button:** "Попробовать бесплатно" / "Bepul sinab ko'rish" / "Start free trial"
**Secondary link:** "Посмотреть демо" / "Demoni ko'rish" / "Watch demo" (scrolls to demo section or opens video)

**Animated Sequence (auto-plays on load, loops):**
```
Frame 1 (0s):    Phone shows Telegram chat with S1P bot
Frame 2 (1.5s):  Incoming call notification slides into phone: "Входящий: Азиз +998 90 123..."
Frame 3 (3s):    CRM dashboard: new row fades into leads table
Frame 4 (4.5s):  Telegram: "Новый лид создан" notification with "Назначить" button
Frame 5 (6s):    Manager taps "Назначить → Дилшод" in Telegram
Frame 6 (7.5s):  CRM: Lead status changes to "Назначен", assigned badge appears
Frame 7 (9s):    CRM: Deal card slides into pipeline kanban
Frame 8 (10.5s): Counter animates: "Лиды сегодня: 12 → 13"
(Loop back to Frame 1 with fade transition)
```

**Implementation:**
- Phone mockup: Pure Tailwind CSS (`rounded-[2.5rem] border-[14px] border-gray-800`) or `react-device-frameset`
- Browser mockup: Tailwind CSS browser chrome (address bar with dots)
- Animation: Framer Motion `AnimatePresence` + `motion.div` with `variants` and `staggerChildren`
- Sequence controller: `useEffect` with interval or Framer Motion timeline
- On mobile: stack vertically, phone on top, browser below

**Social proof strip immediately below hero:**
```
[Sipuni logo]  [Binotel logo]  "Интеграция с вашей телефонией"
```

---

### Section 2: Pain Points — "Вы теряете клиентов"

**Purpose:** Agitate the problem before presenting the solution.

**Layout:** 3 cards in a row (bento-style, slight size variation for visual interest)

**Card 1 (RU):**
- Icon: Phone with X (missed call)
- Stat: "67%"
- Text: "компаний теряют клиентов из-за пропущенных звонков"

**Card 2 (RU):**
- Icon: Clock
- Stat: "5 минут"
- Text: "и клиент уходит к конкуренту, если не перезвонить"

**Card 3 (RU):**
- Icon: Chart trending down
- Stat: "30%"
- Text: "лидов теряются без CRM — менеджеры забывают перезвонить"

**Entrance animation:** Cards fade-up with stagger (0.15s delay each) on scroll into view.

---

### Section 3: Solution — "Как это работает" (3 Steps)

**Layout:** Horizontal 3-step flow with connecting lines/arrows between steps. On mobile: vertical timeline.

**Step 1:**
- Number: "01"
- Icon: Plug/connection icon
- Title (RU): "Подключите телефонию"
- Desc (RU): "Sipuni или Binotel — настройка за 3 минуты. Вставьте API-ключ и готово."

**Step 2:**
- Number: "02"
- Icon: Telegram icon
- Title (RU): "Добавьте Telegram бот"
- Desc (RU): "Добавьте @s1pcrm_bot в рабочий чат. Уведомления начнут приходить мгновенно."

**Step 3:**
- Number: "03"
- Icon: Rocket/dashboard icon
- Title (RU): "Управляйте из CRM"
- Desc (RU): "Лиды, сделки, задачи — всё создаётся автоматически. Ваша команда видит каждый звонок."

**Below steps — bold stat:**
"⏱ Время от регистрации до первого уведомления: < 15 минут"

**Entrance animation:** Steps animate in sequence (left to right) as connecting line draws between them.

---

### Section 4: ROI Calculator (Interactive)

**This is the differentiator — no CIS competitor has this.**

**Layout:** Card with sliders/inputs on left, result on right.

**Inputs:**
| Input | Type | Default | Range |
|---|---|---|---|
| Количество звонков в день | Slider | 50 | 10-500 |
| % пропущенных | Slider | 20% | 5%-50% |
| Средний чек клиента (сум) | Input | 500,000 | — |

**Calculated output (right side, big numbers with spring animation):**
```
Вы теряете в месяц:
[animated number] пропущенных звонков
[animated number] сум упущенной выручки

S1P стоит: 290,000 сум/мес
ROI: [animated] X
```

**Formula:**
```
missed_calls_month = calls_per_day * missed_pct * 30
lost_revenue = missed_calls_month * avg_check * 0.3  (30% would have converted)
roi = lost_revenue / 290000
```

**Implementation:**
- Ant Design Slider for inputs (fits existing design system)
- Framer Motion `useSpring` + `useTransform` for animated number transitions
- Format numbers with Uzbek locale (spaces as thousands separator)
- Currency: сум (UZS)
- Spring animation on every value change (bouncy, satisfying feel)

---

### Section 5: Features — Bento Grid

**Layout:** 2x3 bento grid with mixed card sizes. Top-left card is 2x height (hero feature).

**Hero card (large, 2x height):**
- Title: "Telegram бот"
- Desc: "Мгновенные уведомления о звонках прямо в Telegram. Назначайте лиды, отмечайте обработку, запускайте перезвон — всё без выхода из мессенджера."
- Visual: Animated phone mockup (reuse from hero, smaller)

**Card 2:**
- Title: "CRM для звонков"
- Desc: "Контакты, лиды, сделки, задачи. Воронка продаж с настраиваемыми этапами."
- Icon: Grid/kanban icon

**Card 3:**
- Title: "Sipuni & Binotel"
- Desc: "Глубокая интеграция с ведущими провайдерами. Входящие, исходящие, пропущенные — всё в CRM."
- Icon: Phone/integration icon

**Card 4:**
- Title: "Аналитика"
- Desc: "Кто звонит, кто пропускает, сколько лидов — в реальном времени."
- Icon: Chart icon

**Card 5:**
- Title: "API и Вебхуки"
- Desc: "Публичный REST API. Подключите к любой системе — 1С, Excel, ваш сайт."
- Icon: Code/API icon

**Card 6:**
- Title: "Мультиязычность"
- Desc: "Интерфейс на русском, узбекском и английском. Для вашей команды."
- Icon: Globe icon

**Design:**
- Cards have subtle border (1px gray-200), white background, hover: slight lift + shadow
- Hero card has gradient background (indigo-50 to white)
- Entrance: fade-up with stagger on scroll

---

### Section 6: Pricing

**Layout:** Single centered card (one plan, simple).

**Plan name:** "S1P CRM"
**Price:** "290 000 сум/мес" (prominently, big number)
**Subtitle:** "до 5 пользователей включено"
**Per-user:** "+ 50 000 сум за каждого дополнительного пользователя"

**Included features (checkmark list):**
- Telegram бот с уведомлениями
- CRM: контакты, лиды, сделки, задачи
- Интеграция Sipuni / Binotel
- Аналитика и отчёты
- Публичный API и вебхуки
- 3 языка интерфейса
- Техподдержка в Telegram

**CTA:** "Начать бесплатно — 14 дней" (big button)
**Subtext:** "Без привязки карты. Отмена в любой момент."

**Annual toggle (optional):**
"Годовой план: 2 900 000 сум/год (экономия 580 000 сум)"

**Design:**
- Single card, centered, generous padding
- Price in large bold (32-40px)
- Checkmarks in teal/green
- CTA button: primary color, full-width within card
- Subtle glow/shadow on the card

---

### Section 7: Testimonials (Placeholder Structure)

**Note:** Real testimonials will be added after pilot launch. Build the structure now with placeholder data.

**Layout:** 2-3 testimonial cards, horizontal scroll on mobile.

**Each card:**
```
"Цитата клиента о продукте..."

— Имя Фамилия
  Должность, Компания
  [Company logo placeholder]
```

**Placeholder testimonials (mark clearly in code as PLACEHOLDER):**
1. "Раньше мы теряли 30% звонков. С S1P каждый пропущенный — в Telegram за секунду." — Азиз К., Директор, [Company]
2. "Настроили за 10 минут. Менеджеры теперь не забывают перезванивать." — Дилшод М., Руководитель колл-центра, [Company]

**Design:** Cards with quote marks, photo placeholder circle, subtle border.

---

### Section 8: FAQ Accordion

**Questions (RU):**

| Q | A |
|---|---|
| Сколько стоит S1P? | 290 000 сум/мес за команду до 5 человек. Каждый дополнительный пользователь — 50 000 сум/мес. 14 дней бесплатно. |
| Какие телефонии поддерживаются? | Sipuni и Binotel. Подключение через API-ключ занимает 3 минуты. |
| Нужно ли устанавливать приложение? | Нет. S1P работает в браузере. Telegram бот — в вашем обычном Telegram. |
| Как работает Telegram бот? | Добавляете бота в рабочий чат. При каждом звонке — уведомление с информацией о клиенте и кнопками действий. |
| Безопасны ли наши данные? | Да. Каждая компания полностью изолирована. Данные хранятся на защищённых серверах. HTTPS, шифрование, регулярные бэкапы. |
| Можно ли подключить к 1С или другой системе? | Да. У S1P есть публичный REST API и вебхуки. Интеграция с любой системой. |
| Есть ли техподдержка? | Да. Telegram-группа для клиентов + прямая поддержка через бот. |

**Implementation:** Ant Design Collapse/Accordion component. Simple, clean.

---

### Section 9: CTA Footer

**Background:** Gradient (indigo-600 to indigo-800) or dark section for contrast.

**Headline (RU):** "Начните получать уведомления о звонках сегодня"
**CTA:** "Попробовать бесплатно — 14 дней" (white button on dark bg)
**Subtext:** "Настройка за 15 минут. Без привязки карты."

---

### Footer

- Logo + short description
- Links: О продукте, Цены, FAQ, API документация
- Contact: Phone (+998...), Email, Telegram (@s1p_support)
- Address: Ташкент, [address]
- Language toggle (repeat)
- Copyright: 2026 S1P

---

## File Structure

```
src/app/
  (landing)/              # Route group for landing pages (no auth layout)
    layout.tsx            # Landing-specific layout (no sidebar, no auth check)
    page.tsx              # Main landing page (root "/" route)
    components/
      Navbar.tsx          # Sticky navigation
      HeroSection.tsx     # Split-screen animated demo
      PainPoints.tsx      # "You're losing clients" cards
      HowItWorks.tsx      # 3-step flow
      ROICalculator.tsx   # Interactive calculator
      FeatureGrid.tsx     # Bento grid features
      Pricing.tsx         # Pricing card
      Testimonials.tsx    # Client quotes
      FAQ.tsx             # Accordion
      CTAFooter.tsx       # Final call to action
      Footer.tsx          # Site footer
      PhoneMockup.tsx     # Reusable Telegram phone animation
      BrowserMockup.tsx   # Reusable CRM browser animation
      AnimatedCounter.tsx # Number animation component
```

**IMPORTANT:** The landing page must NOT use the existing authenticated layout. It needs its own `(landing)/layout.tsx` that:
- Has NO sidebar
- Has NO auth check / ProtectedRoute
- Has NO apiClient dependency
- Is fully static / client-rendered for animations
- Shares the same Tailwind theme and i18n setup

**Routing:** The root `/` currently redirects to `/dashboard`. Change this:
- `/` → Landing page (public, no auth)
- `/dashboard` → Authenticated dashboard (existing, unchanged)
- `/login`, `/register` → Existing auth pages (unchanged)

---

## i18n Keys

Add landing page keys to existing locale files:

```
src/i18n/messages/
  ru.json  → add "landing" namespace
  uz.json  → add "landing" namespace
  en.json  → add "landing" namespace
```

Structure:
```json
{
  "landing": {
    "nav": {
      "tryFree": "Попробовать бесплатно",
      "pricing": "Цены",
      "features": "Возможности",
      "faq": "FAQ"
    },
    "hero": {
      "title": "Ни один звонок не останется без внимания",
      "subtitle": "CRM + Telegram бот для колл-центров...",
      "cta": "Попробовать бесплатно",
      "watchDemo": "Посмотреть демо"
    },
    "pain": {
      "title": "Вы теряете клиентов",
      "stat1": "67%",
      "stat1Text": "компаний теряют клиентов из-за пропущенных звонков",
      "stat2": "5 минут",
      "stat2Text": "и клиент уходит к конкуренту",
      "stat3": "30%",
      "stat3Text": "лидов теряются без CRM"
    },
    "steps": { ... },
    "roi": { ... },
    "features": { ... },
    "pricing": { ... },
    "testimonials": { ... },
    "faq": { ... },
    "ctaFooter": { ... },
    "footer": { ... }
  }
}
```

All visible text MUST go through `t()`. No hardcoded strings.

---

## Responsive Breakpoints

| Breakpoint | Layout |
|---|---|
| Desktop (1024px+) | Full layout, side-by-side mockups, horizontal steps, 2x3 bento grid |
| Tablet (768-1023px) | Stacked mockups, 2-column bento grid, horizontal steps compressed |
| Mobile (< 768px) | Single column everything, vertical timeline for steps, hamburger nav |

All sections must be usable on mobile. No horizontal scroll except testimonials carousel.

---

## Performance Requirements

- **Lighthouse Performance:** > 90
- **First Contentful Paint:** < 1.5s
- **No layout shift** from animations (use `will-change` sparingly, transform only)
- **Lazy-load** everything below the fold (ROI calculator, features, pricing, testimonials, FAQ)
- **Hero animation:** Lightweight — no heavy Lottie or video above the fold
- **Images:** Use next/image with proper sizing. Optimize all assets.
- **Bundle:** framer-motion is ~85KB — acceptable. Don't add GSAP on top.

---

## Acceptance Criteria

1. Landing page loads at `/` without authentication
2. All 9 sections render correctly in RU, UZ, EN
3. Language toggle switches all text without page reload
4. Hero split-screen animation plays smoothly (60fps)
5. ROI calculator updates numbers with spring animation on slider change
6. All sections have scroll-triggered entrance animations
7. `prefers-reduced-motion` disables all animations
8. Mobile layout works down to 375px width
9. CTA buttons link to `/register` (existing registration flow)
10. Navbar "Попробовать бесплатно" links to `/register`
11. FAQ accordion opens/closes smoothly
12. No console errors, no hydration warnings
13. Lighthouse Performance > 90
14. No authenticated API calls from landing page
15. Existing `/dashboard`, `/login`, `/register` routes unchanged

---

## Content Notes

- All stats in Pain Points section (67%, 5 min, 30%) are industry-standard call center statistics. Mark in code comments as "industry stats, update with real data when available."
- Testimonials are placeholders. Add `{/* PLACEHOLDER: Replace with real testimonials after pilot */}` comments.
- Pricing is in UZS (Uzbek som). $29 USD ≈ 290,000 UZS. Use UZS for Uzbek/Russian, USD for English.
- Phone number in navbar: use a real number when available, placeholder format +998 XX XXX XX XX for now.

---

## What This Is NOT

- This is NOT a redesign of the authenticated CRM UI
- This is NOT connected to the backend API (fully static/client-side)
- This is NOT a separate project — it lives in the same Next.js app
- This does NOT replace the existing `/login` or `/register` pages
- This does NOT need Ant Design for most components — use Tailwind + Framer Motion for the landing page aesthetic. Ant Design only for FAQ Accordion (Collapse) and ROI Calculator sliders if needed.
