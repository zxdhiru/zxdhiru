# Zephyr 2k26 — Full Site Audit Report
**URL:** https://zephyr-2k26.vercel.app  
**Scanned:** April 27, 2026  
**Auditor:** Claude (Anthropic)

---

## Routes Scanned

| Route | Status |
|---|---|
| `/` | ✅ Accessible |
| `/events` | ✅ Accessible |
| `/events/empower-art-gallery` | ✅ Accessible |
| `/events/voices-of-tomorrow` | ✅ Accessible |
| `/events/rhythms-of-resilience` | ✅ Accessible |
| `/events/tech-matriarchs` | ✅ Accessible |
| `/events/she-innovates-hackathon` | ✅ Accessible |
| `/login` | ✅ Accessible |
| `/dashboard` | ❓ Not linked / unknown |
| `/register` | ❓ Not linked / unknown |

---

## Summary Scorecard

| Category | Score | Severity |
|---|---|---|
| Navigation & UX | 4/10 | 🔴 Critical |
| Content Completeness | 5/10 | 🔴 Critical |
| Registration Flow | 2/10 | 🔴 Critical |
| Footer & Links | 3/10 | 🟠 High |
| SEO & Meta | 4/10 | 🟠 High |
| Accessibility | 5/10 | 🟠 High |
| Mobile UX | Unknown | 🟡 Medium |
| Legal Pages | 0/10 | 🟠 High |

---

## 🔴 Critical Issues

### 1. Navigation Bar is Missing
**Where:** Every page  
**Problem:** There is no visible navigation menu on any page. Users have no way to jump between Home, Events, Login, or any other section without manually going back or clicking inline links. This is a fundamental UX failure — it isolates every page and kills discoverability.

**Fix:**
```jsx
// Add a sticky Navbar component to your root layout
<nav className="sticky top-0 z-50 flex justify-between items-center px-8 py-4 bg-black/80 backdrop-blur">
  <Link href="/">ZEPHYR 2k26</Link>
  <div className="flex gap-6">
    <Link href="/events">Events</Link>
    <Link href="/schedule">Schedule</Link> {/* add this page */}
    <Link href="/login">Login</Link>
  </div>
</nav>
```
Add it to `app/layout.tsx` so it appears on every route.

---

### 2. Events Page Has No Filtering / Search
**Where:** `/events`  
**Problem:** All 5 events are dumped in a grid with no way to filter by category (Cultural, Technical, Gala), day, or entry fee. As events scale, this becomes unusable. The Navkriti page you built has tabbed event filtering — apply the same pattern here.

**Fix:** Add filter tabs at the top of `/events`:
```jsx
const categories = ["All", "Cultural", "Technical", "Gala"];
const [active, setActive] = useState("All");
const filtered = events.filter(e => active === "All" || e.category === active);
```

---

### 3. Registration / Participation Flow is Broken
**Where:** Every event detail page (e.g. `/events/empower-art-gallery`)  
**Problem:** Each event page ends with a **"Login to participate"** link, but clicking it just takes the user to `/login`. After login (Google OAuth), there is no redirect back to the event, no registration form, no confirmation, no ticket — nothing. The user journey is:

> See event → click participate → login → **dead end**

There is no post-login dashboard, no registration form, no ticket generation, and no team-creation flow (critical for group events like the Hackathon that require 2–4 members).

**Fix:**
- Add `?redirect=/events/she-innovates-hackathon` param to the login URL so the user lands back on the event page after auth.
- Add a `/events/[slug]/register` route with a proper form (name, email, phone, team members for group events).
- Show a confirmation screen / send a confirmation email via Resend.
- Create a `/dashboard` page showing all registered events.

---

### 4. Footer Social Links Are Dead (`#` placeholders)
**Where:** Every page footer  
**Problem:** Instagram, LinkedIn, and Twitter all link to `#`. For a public-facing festival site this is embarrassing — visitors looking to follow the event on social find nothing.

**Fix:** Replace immediately:
```jsx
<a href="https://instagram.com/zephyrfest2026" target="_blank">Instagram</a>
<a href="https://linkedin.com/company/zephyrfest" target="_blank">LinkedIn</a>
<a href="https://twitter.com/zephyrfest2026" target="_blank">Twitter</a>
```

---

### 5. Footer "Media Kit" and "Privacy Policy" Links Go Nowhere
**Where:** Every page footer  
**Problem:** Both `Media Kit` and `Privacy Policy` are `href="#"` — entirely non-functional. Privacy Policy is a **legal requirement** if you're collecting user data (Google OAuth login = collecting data).

**Fix:**
- Create `/privacy-policy` page (required by Google OAuth terms too).
- Create `/media-kit` as either a PDF download or a proper page.
- Create `/terms` (referenced on the login page but has no actual route).

---

## 🟠 High Priority Issues

### 6. Login Page References "Terms of Service" — Page Doesn't Exist
**Where:** `/login`  
**Problem:** The login page says *"By continuing, you agree to our Terms of Service and Privacy Policy"* but neither page exists. This is a legal liability, and Google OAuth may flag this during review.

**Fix:** Create `/terms` and `/privacy-policy` as static Next.js pages.

---

### 7. No Countdown Timer to the Festival
**Where:** Homepage `/`  
**Problem:** The festival is May 15–17, 2026 — less than 3 weeks away. There is no countdown creating urgency. This is a massive missed opportunity for conversions (event registrations).

**Fix:**
```jsx
// Simple countdown component
const [timeLeft, setTimeLeft] = useState(calculateTimeLeft("2026-05-15"));
// Display: 17 days | 4 hours | 22 minutes | 09 seconds
```
Place it prominently on the homepage hero, just below the CTA button.

---

### 8. No Schedule / Timetable Page
**Where:** Missing route  
**Problem:** The homepage describes a 3-day schedule loosely ("Day 01 / Day 02 / Day 03") but there is no dedicated `/schedule` page with a full timetable. Attendees cannot plan their 3 days without a proper schedule view.

**Fix:** Create `/schedule` with a day-by-day timeline or tab layout showing all events, their times, and venues in one place.

---

### 9. SEO Meta Tags Are Incomplete
**Where:** All pages  
**Problem:** Every page shares the same generic title `"Women Empowerment | Zephyr Fest 2026"`. Individual event pages have no unique `<title>` or `<meta description>`. There is no Open Graph (og:image, og:title) or Twitter Card metadata, meaning sharing links on WhatsApp / Twitter will show blank previews.

**Fix in Next.js App Router (`layout.tsx` / `page.tsx`):**
```tsx
// In each event page
export const metadata: Metadata = {
  title: "She-Innovates Hackathon | Zephyr 2k26",
  description: "A 24-hour collaborative hackathon for female-led teams. ₹150 entry. $5000 prize pool.",
  openGraph: {
    title: "She-Innovates Hackathon | Zephyr 2k26",
    description: "...",
    images: ["/assets/events/hackathon.png"],
  },
};
```

---

### 10. No 404 Page
**Where:** Any invalid route  
**Problem:** Visiting an undefined route likely shows a bare Next.js default error page, breaking the visual experience and losing users.

**Fix:** Create `app/not-found.tsx`:
```tsx
export default function NotFound() {
  return (
    <div className="min-h-screen flex flex-col items-center justify-center text-center">
      <h1 className="text-6xl font-bold">404</h1>
      <p>This page doesn't exist. But Zephyr does.</p>
      <Link href="/">← Back to Home</Link>
    </div>
  );
}
```

---

### 11. Event Detail Pages Lack Key Information
**Where:** All `/events/[slug]` pages  
**Problem:** The "About this Event" section is a single paragraph. Missing:
- **Judging criteria** (for competitive events like Hackathon, Poetry Slam)
- **Rules and eligibility** (Who can participate? Age limit? College only?)
- **Required materials** (What to bring? Laptop? Art supplies?)
- **Speakers/Judges** (Who are the panelists for Tech Matriarchs?)
- **Registration deadline** (Is May 14 the last day to register?)
- **Contact for queries** (A WhatsApp number or email for event-specific questions)

---

### 12. Inconsistent Prize Pool Data
**Where:** Events list vs event detail pages  
**Problem:** Some events show prize pools in `$` (dollars) — e.g. `$5000` for the Hackathon, `$500` for Voices of Tomorrow — while entry fees are in `₹` (rupees). This is confusing for an Indian college fest. Pick one currency and be consistent.

---

## 🟡 Medium Priority Issues

### 13. No "About" Page for the Fest
**Where:** Missing route  
**Problem:** There is no `/about` page explaining what Zephyr 2k26 is, which college/organization runs it, the committee, mission, or sponsors. First-time visitors have no way to verify legitimacy.

**Fix:** Create `/about` with: organizing institution, team/committee, sponsors, mission statement.

---

### 14. No Sponsors / Partners Section
**Where:** Homepage and footer  
**Problem:** Most college fests have sponsors. If Zephyr does, they need to be acknowledged. If sponsors are being sought, a "Sponsor Us" CTA or link to a sponsorship deck is needed.

---

### 15. The "Explore Events" CTA on Homepage Scrolls to Nothing
**Where:** `/` Hero section  
**Problem:** The hero CTA "Explore Events" links to `/events` which is correct, but there is no visual transition or scroll animation — it's just a cold page load. Minor but affects polish.

---

### 16. "Discover the Future" Scroll Indicator (if any) Should Be Functional
**Where:** `/` Hero  
**Problem:** There is a "Discover the Future" text below the hero. This appears to be a scroll prompt but has no scroll arrow or animation to indicate it.

---

### 17. Login Page Has No Email/Password Fallback
**Where:** `/login`  
**Problem:** Only Google OAuth is available. Attendees without Google accounts (some older participants or institutional emails with restrictions) have no alternative. Consider adding an email OTP option.

---

### 18. Group Event Registration Lacks Team Management UI
**Where:** `/events/she-innovates-hackathon`, `/events/rhythms-of-resilience`  
**Problem:** The Hackathon requires 2–4 members, and Rhythms of Resilience requires 3–10 members. There is no team creation, team joining via code, or member invitation flow anywhere on the site.

**Fix:** After login, provide:
1. "Create a team" → generates a 6-character invite code
2. "Join a team" → enter invite code
3. Show current team members before final registration submit

---

### 19. No Confirmation Email or Ticket After Registration
**Where:** Post-registration (flow doesn't exist yet)  
**Problem:** Even when the registration flow is built, ensure a confirmation email is sent via Resend with event details, date, time, venue, and a QR code ticket.

---

## 🟢 What's Working Well

- ✅ Visual design is elegant — dark theme, serif typography, strong hero imagery
- ✅ Event data is well-structured (date, time, entry fee, prize pool, capacity)  
- ✅ Individual event detail pages have a consistent and clean layout
- ✅ Google OAuth login is the right low-friction choice for Indian college fest attendees
- ✅ The 3-day journey section on the homepage effectively tells the festival story
- ✅ Shreya Ghoshal headliner reveal is a great hook

---

## Priority Action Plan

| Priority | Task | Effort |
|---|---|---|
| 🔴 P0 | Add global Navbar to `layout.tsx` | 1 hour |
| 🔴 P0 | Fix all `href="#"` footer links | 30 min |
| 🔴 P0 | Build post-login registration form + redirect | 1 day |
| 🔴 P0 | Create `/privacy-policy` and `/terms` pages | 2 hours |
| 🟠 P1 | Add countdown timer to homepage | 1 hour |
| 🟠 P1 | Add category filter tabs to `/events` | 2 hours |
| 🟠 P1 | Create `/schedule` page | 3 hours |
| 🟠 P1 | Add unique SEO metadata per page | 2 hours |
| 🟠 P1 | Create `app/not-found.tsx` | 30 min |
| 🟡 P2 | Add team management flow for group events | 1–2 days |
| 🟡 P2 | Add `/about` page | 2 hours |
| 🟡 P2 | Unify currency ($ → ₹) across prize pools | 30 min |
| 🟡 P2 | Add judges/speakers to event pages | 2 hours |
| 🟡 P2 | Add confirmation email via Resend | 3 hours |

---

*Report generated by Claude | zephyr-2k26.vercel.app | April 27, 2026*
