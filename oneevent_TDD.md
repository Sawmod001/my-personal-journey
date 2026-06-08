# OneEvent — Technical Tracking Document (TTD)

**Version:** 1.0
**Date:** June 2026
**Author:** OneEvent Engineering
**Status:** Active — Phase 1 Build

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Environment Variables](#3-environment-variables)
4. [Folder Structure](#4-folder-structure)
5. [Data Models](#5-data-models)
6. [API Routes](#6-api-routes)
7. [Booking State Machine](#7-booking-state-machine)
8. [Payment Flow](#8-payment-flow)
9. [Email Notifications](#9-email-notifications)
10. [Security Rules](#10-security-rules)
11. [Build Checklist — Week by Week](#11-build-checklist--week-by-week)
12. [Known Constraints](#12-known-constraints)

---

## 1. Project Overview

**OneEvent** is a venue and space booking marketplace for Nigeria and Africa. It connects guests who need a venue for their event with hosts who own spaces, managed through a transparent booking workflow with direct bank transfer payments.

| Item | Detail |
|---|---|
| Platform | Web application |
| Phase | Phase 1 — MVP |
| Framework | Next.js 15 (App Router) |
| Language | JavaScript (JSX) |
| Database | MongoDB Atlas |
| Deployment | Vercel |

### What Phase 1 includes

- Guest, Host, and Admin user roles
- Venue listing creation and admin approval
- Public venue browse, search, and filter
- Full booking request and status lifecycle
- Direct bank transfer payment with proof upload
- Email notifications via Resend
- Post-event review system
- Dispute filing and admin resolution

### What Phase 1 explicitly excludes

- No vendor marketplace
- No AI features
- No WhatsApp or SMS notifications
- No payment gateway (Stripe, Paystack, Flutterwave)
- No map view
- No real-time chat
- No mobile app

---

## 2. Tech Stack

| Layer | Technology | Purpose | Cost |
|---|---|---|---|
| Framework | Next.js 15 (App Router) | Pages, API routes, server components | Free |
| Language | JavaScript / JSX | All code | Free |
| Styling | Tailwind CSS | Utility-first styles | Free |
| UI Components | shadcn/ui | Pre-built accessible components | Free |
| Database | MongoDB Atlas (M0 free cluster) | All data storage | Free |
| ODM | Mongoose | Schema validation and DB access | Free |
| Auth | NextAuth v5 (Auth.js) | Sessions, email/password auth | Free |
| File Storage | Cloudinary | Venue photos, payment proof uploads | Free (25GB) |
| Email | Resend | Transactional emails | Free (3,000/month) |
| Validation | Zod | API request body validation | Free |
| Deployment | Vercel | Hosting, CI/CD from GitHub | Free |

### Key packages to install at project start

```bash
npm install next-auth@beta @auth/mongodb-adapter mongoose
npm install cloudinary
npm install resend
npm install zod
npm install @upstash/ratelimit @upstash/redis
npm install date-fns
npm install react-image-gallery
```

---

## 3. Environment Variables

Create `.env.local` at the project root. Never commit this file.
Create `.env.example` with empty values — commit this.

```env
# MongoDB
MONGODB_URI=mongodb+srv://<user>:<password>@cluster0.xxxxx.mongodb.net/oneevent

# NextAuth
NEXTAUTH_SECRET=your-long-random-secret-here
NEXTAUTH_URL=http://localhost:3000

# Cloudinary
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=

# Resend
RESEND_API_KEY=
RESEND_FROM_EMAIL=noreply@oneevent.com

# Upstash (rate limiting)
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=
```

---

## 4. Folder Structure

```
oneevent/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/page.jsx
│   │   │   ├── register/page.jsx
│   │   │   └── verify-email/page.jsx
│   │   │
│   │   ├── (main)/
│   │   │   ├── layout.jsx              # Navbar + Footer
│   │   │   ├── page.jsx                # Landing page
│   │   │   └── venues/
│   │   │       ├── page.jsx            # Browse + search + filter
│   │   │       └── [id]/page.jsx       # Venue detail + Book button
│   │   │
│   │   ├── dashboard/
│   │   │   ├── layout.jsx              # Sidebar shell — role-aware
│   │   │   ├── guest/
│   │   │   │   ├── page.jsx            # My bookings list
│   │   │   │   └── bookings/[id]/page.jsx
│   │   │   ├── host/
│   │   │   │   ├── page.jsx            # My venues + incoming requests
│   │   │   │   ├── venues/new/page.jsx
│   │   │   │   ├── venues/[id]/edit/page.jsx
│   │   │   │   └── bookings/[id]/page.jsx
│   │   │   └── admin/
│   │   │       ├── page.jsx            # Platform stats
│   │   │       ├── venues/page.jsx     # Approval queue
│   │   │       ├── users/page.jsx
│   │   │       └── disputes/page.jsx
│   │   │
│   │   └── api/
│   │       ├── auth/[...nextauth]/route.js
│   │       ├── venues/
│   │       │   ├── route.js            # GET (browse) / POST (create)
│   │       │   └── [id]/
│   │       │       ├── route.js        # GET / PUT / DELETE
│   │       │       └── approve/route.js
│   │       ├── bookings/
│   │       │   ├── route.js            # POST (create)
│   │       │   └── [id]/
│   │       │       ├── route.js        # GET (detail)
│   │       │       ├── status/route.js # PATCH (transition)
│   │       │       └── proof/route.js  # PATCH (upload proof URL)
│   │       ├── reviews/route.js        # POST (create)
│   │       └── upload/route.js         # POST (Cloudinary signed upload)
│   │
│   ├── components/
│   │   ├── ui/                         # shadcn auto-generated — do not edit
│   │   ├── venue/
│   │   │   ├── VenueCard.jsx
│   │   │   ├── VenueGallery.jsx
│   │   │   ├── VenueFilters.jsx
│   │   │   └── VenueForm.jsx
│   │   ├── booking/
│   │   │   ├── BookingForm.jsx
│   │   │   ├── BookingStatusBadge.jsx
│   │   │   ├── BookingTimeline.jsx
│   │   │   ├── PaymentCard.jsx
│   │   │   └── ProofUploader.jsx
│   │   ├── auth/
│   │   │   ├── LoginForm.jsx
│   │   │   └── RegisterForm.jsx
│   │   └── shared/
│   │       ├── Navbar.jsx
│   │       ├── Footer.jsx
│   │       ├── RoleGuard.jsx
│   │       ├── StatusBadge.jsx
│   │       └── EmptyState.jsx
│   │
│   ├── lib/
│   │   ├── db.js                       # MongoDB singleton connection
│   │   ├── auth.js                     # NextAuth config
│   │   ├── cloudinary.js               # Upload helper
│   │   ├── email.js                    # sendEmail() + 6 templates
│   │   ├── validations.js              # All Zod schemas
│   │   └── utils.js                    # generateBookingRef(), formatDate()
│   │
│   ├── models/
│   │   ├── User.js
│   │   ├── Venue.js
│   │   ├── Booking.js
│   │   └── Review.js
│   │
│   └── middleware.js                   # Route group protection (first pass only)
│
├── .env.local                          # NEVER commit
├── .env.example                        # Commit with empty values
└── next.config.js
```

---

## 5. Data Models

### 5.1 User

```js
// src/models/User.js
{
  name:            String (required, trim),
  email:           String (required, unique, lowercase),
  password:        String (hashed — handled by NextAuth),
  role:            enum ['guest', 'host', 'admin'] — default: 'guest',
  isEmailVerified: Boolean — default: false,
  phone:           String (optional),
  profilePhoto:    String — Cloudinary URL (optional),

  // Populated only when role === 'host'
  bankDetails: {
    bankName:      String,
    accountName:   String,
    accountNumber: String,
  },

  isSuspended: Boolean — default: false,
  timestamps:  true (createdAt, updatedAt)
}
```

### 5.2 Venue

```js
// src/models/Venue.js
{
  host:          ObjectId → User (required),
  name:          String (required, trim),
  description:   String (required),
  type:          enum ['event_hall','conference_room','studio',
                       'outdoor','lounge','other'],
  capacity:      Number (required),
  pricePerDay:   Number (required),
  pricePerHour:  Number (optional),
  address:       String (required),
  city:          String (required),
  state:         String (required),
  amenities:     [String],
  photos:        [String] — Cloudinary URLs, max 8,

  status:        enum ['listed','verified','verified_business',
                       'rejected','suspended'] — default: 'listed',
  rejectionNote: String (optional),
  verifiedAt:    Date (optional),

  // Indexes:
  // VenueSchema.index({ name: 'text', description: 'text', city: 'text' })
  // VenueSchema.index({ city: 1, status: 1, type: 1 })

  timestamps: true
}
```

### 5.3 Booking

```js
// src/models/Booking.js
{
  reference:       String (unique) — auto-generated via pre-save hook: OE-2026-000001,
  venue:           ObjectId → Venue (required),
  guest:           ObjectId → User (required),
  host:            ObjectId → User (required — denormalized),

  eventDate:       Date (required),
  eventEndDate:    Date (required),
  guestCount:      Number (required),
  specialRequests: String (optional),
  totalAmount:     Number (optional — host sets this on acceptance),

  status: enum [
    'pending',           // booking just submitted
    'accepted',          // host accepted
    'rejected',          // host rejected
    'awaiting_payment',  // system state after acceptance
    'payment_submitted', // guest uploaded proof
    'confirmed',         // host confirmed payment
    'completed',         // event has happened — reviews unlock
    'cancelled'          // either party cancelled
  ] — default: 'pending',

  paymentProofUrl: String (Cloudinary URL),

  // Snapshot of host bank details at time of acceptance
  // Never read live from User — always read from here
  hostBankSnapshot: {
    bankName:      String,
    accountName:   String,
    accountNumber: String,
  },

  // Full audit trail — append on every status change
  statusHistory: [{
    status:    String,
    changedAt: Date,
    changedBy: ObjectId → User,
  }],

  // Dispute (populated only if guest reports an issue)
  reportedIssue: {
    description:  String,
    evidenceUrls: [String],
    reportedAt:   Date,
    resolution:   String,
    resolvedAt:   Date,
  },

  // Indexes:
  // BookingSchema.index({ venue: 1, eventDate: 1 })
  // BookingSchema.index({ guest: 1 })
  // BookingSchema.index({ host: 1 })

  timestamps: true
}
```

### 5.4 Review

```js
// src/models/Review.js
{
  booking: ObjectId → Booking (required, unique), // unique prevents duplicates
  venue:   ObjectId → Venue (required),
  guest:   ObjectId → User (required),
  rating:  Number (required, min: 1, max: 5),
  comment: String (required, maxlength: 500),
  isFlagged: Boolean — default: false,

  timestamps: true
}
```

---

## 6. API Routes

### Auth
| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/api/auth/[...nextauth]` | Public | NextAuth handler |

### Venues
| Method | Route | Access | Description |
|---|---|---|---|
| GET | `/api/venues` | Public | Browse + search + filter |
| POST | `/api/venues` | Host | Create new venue listing |
| GET | `/api/venues/[id]` | Public | Single venue detail |
| PUT | `/api/venues/[id]` | Host (owner) | Edit venue |
| DELETE | `/api/venues/[id]` | Host (owner) | Delete venue |
| PATCH | `/api/venues/[id]/approve` | Admin | Approve or reject venue |

### Bookings
| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/api/bookings` | Guest | Create booking request |
| GET | `/api/bookings/[id]` | Owner (guest or host) | Booking detail |
| PATCH | `/api/bookings/[id]/status` | Host / Admin | Transition booking status |
| PATCH | `/api/bookings/[id]/proof` | Guest (owner) | Upload payment proof URL |

### Reviews
| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/api/reviews` | Guest (completed booking) | Submit review |

### Uploads
| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/api/upload` | Authenticated | Get Cloudinary signed upload URL |

---

## 7. Booking State Machine

### Allowed Transitions

```
pending           → accepted, rejected, cancelled
accepted          → awaiting_payment, cancelled
awaiting_payment  → payment_submitted, cancelled
payment_submitted → confirmed, cancelled
confirmed         → completed, cancelled
completed         → (terminal — no further transitions)
rejected          → (terminal)
cancelled         → (terminal)
```

### Double-Booking Prevention Query

Run this check **before** saving any new booking:

```js
const clash = await Booking.findOne({
  venue: venueId,
  status: { $in: ['accepted', 'awaiting_payment', 'payment_submitted', 'confirmed'] },
  eventDate:    { $lt: newEndDate },
  eventEndDate: { $gt: newStartDate },
})
if (clash) {
  return res.status(409).json({ error: 'Venue is unavailable for those dates' })
}
```

### Booking Reference Auto-Generation

```js
BookingSchema.pre('save', async function (next) {
  if (!this.reference) {
    const count = await mongoose.model('Booking').countDocuments()
    this.reference = `OE-${new Date().getFullYear()}-${String(count + 1).padStart(6, '0')}`
  }
  next()
})
```

---

## 8. Payment Flow

OneEvent does **not** handle money. Payments are made directly from guest to host via bank transfer.

### Step-by-step

1. Host fills in bank details on their profile (bank name, account name, account number).
2. Host accepts a booking → system snapshots `hostBankSnapshot` onto the Booking document.
3. Guest's booking detail page shows the payment card (only visible after status = `awaiting_payment`).
4. Guest transfers the amount to the host's account outside the platform.
5. Guest uploads a screenshot/photo of the transfer receipt via Cloudinary.
6. Status moves to `payment_submitted`.
7. Host views proof on their dashboard and clicks "Confirm receipt."
8. Status moves to `confirmed`. Both parties receive a confirmation email.

### Why snapshot bank details

If a host later edits or removes their bank details, all existing bookings retain the correct payment information that was valid at the time of acceptance. The snapshot is permanent on the Booking document.

---

## 9. Email Notifications

All emails sent via **Resend**. Managed in `src/lib/email.js` with one `sendEmail(template, data)` function.

| # | Email | Trigger | Recipient |
|---|---|---|---|
| 1 | New booking request | Booking created (pending) | Host |
| 2 | Booking accepted | Status → awaiting_payment | Guest |
| 3 | Booking rejected | Status → rejected | Guest |
| 4 | Payment proof received | Status → payment_submitted | Host |
| 5 | Booking confirmed | Status → confirmed | Guest + Host |
| 6 | Email verification | User registered | New user |

### Email function signature

```js
// src/lib/email.js
export async function sendEmail(template, data) {
  // template: 'booking_request' | 'booking_accepted' | 'booking_rejected'
  //           | 'proof_received' | 'booking_confirmed' | 'verify_email'
  // data: object with all required fields for that template
}
```

---

## 10. Security Rules

### Non-negotiable rules — every one must be implemented

**Rule 1 — Re-verify auth inside every API route**
Middleware can be bypassed (CVE-2025-29927). Every sensitive API route must call `getServerSession()` and check the session role before any database operation. Return 401 if no session, 403 if wrong role.

**Rule 2 — Validate every request body with Zod**
All POST and PATCH routes parse the body through a Zod schema in `src/lib/validations.js` before any database call. Wrong type, missing field, or oversized string → return 400.

**Rule 3 — Role-check ownership on sensitive reads**
The booking detail API must verify `session.user.id === booking.guest || session.user.id === booking.host` before returning data. Admin bypasses this check. Anyone else gets 403.

**Rule 4 — Validate uploads before Cloudinary**
Check `file.type` is one of `image/jpeg`, `image/png`, `image/webp`. Check file size ≤ 5MB. Return 400 before calling Cloudinary if either check fails.

**Rule 5 — Rate limit auth routes**
`/api/auth/login` and `/api/auth/register` are rate-limited to 5 requests per 15 minutes per IP using `@upstash/ratelimit`. Return 429 when exceeded.

**Rule 6 — Environment variables only for secrets**
MongoDB URI, NextAuth secret, Cloudinary keys, Resend API key → all in `.env.local`. `.env.local` is gitignored. Production secrets go in Vercel environment settings UI.

**Rule 7 — Strict status transitions**
The status PATCH route validates that the requested transition is in the allowed transitions map. Any unlisted transition returns 400. The client never controls status directly.

**Rule 8 — Keep Next.js at 15.2.3+**
Run `npm outdated` weekly. The middleware bypass vulnerability affects earlier versions.

---

## 11. Build Checklist — Week by Week

### Week 3 — Auth + project setup
- [ ] Init Next.js 15 with Tailwind and shadcn/ui
- [ ] Connect MongoDB Atlas free cluster
- [ ] Set up NextAuth v5 with MongoDB adapter
- [ ] Build register page (name, email, password, role selector)
- [ ] Build login page
- [ ] Add Zod validation to register and login inputs
- [ ] Create 3 empty dashboard pages (guest, host, admin)
- [ ] Deploy to Vercel — confirm live URL works
- [ ] Create `.env.example`

### Week 4 — Venue creation + admin approval
- [ ] Create Venue Mongoose model with all fields and indexes
- [ ] Build venue creation form (multi-step: details → photos → location)
- [ ] Wire Cloudinary photo upload (up to 8 photos)
- [ ] Venues save with status: `listed`
- [ ] Build admin approval queue (list pending venues, approve/reject buttons)
- [ ] Approved venues move to status: `verified`
- [ ] Build VenueCard component with badge
- [ ] Wire approval/rejection email to host

### Week 5 — Guest discovery
- [ ] Add MongoDB text index to Venue schema (name, description, city)
- [ ] Build public venue browse page
- [ ] Add text search (name + city)
- [ ] Add filters (type, state, capacity range, price range)
- [ ] Add sort options (newest, price low-high, high-low)
- [ ] Build full venue detail page (gallery, all fields, host contact)
- [ ] Add verification badge display
- [ ] Wire "Request to book" CTA (redirects to login if not authenticated)

### Week 6 — Booking system
- [ ] Create Booking Mongoose model with all fields and indexes
- [ ] Build booking request form (dates, guest count, special requests)
- [ ] Implement double-booking prevention query
- [ ] Implement auto-generated booking reference (pre-save hook)
- [ ] Build guest booking list + detail page with status timeline
- [ ] Build host dashboard: incoming requests with accept/reject
- [ ] Implement `hostBankSnapshot` on acceptance
- [ ] Show payment card to guest after acceptance
- [ ] Build proof upload for guest (Cloudinary → saves URL)
- [ ] Build host "Confirm receipt" button
- [ ] Wire all 4 booking status emails
- [ ] Implement strict status transition validation

### Week 7 — Reviews + disputes + email verification
- [ ] Build review form — gated by `status === 'completed'` in API
- [ ] Display reviews on venue detail page (average rating + list)
- [ ] Add "Report issue" button + evidence upload on booking detail
- [ ] Build dispute panel in admin dashboard
- [ ] Build admin "Mark as completed" action for confirmed bookings
- [ ] Add email verification flow (token generation + verify page)
- [ ] Block booking submission for unverified users

### Week 8 — Polish + Demo Day
- [ ] Add loading skeletons (shadcn Skeleton) on venue cards + booking lists
- [ ] Make all pages mobile responsive (Tailwind responsive prefixes)
- [ ] Seed database: 6 venues, bookings in multiple statuses, reviews
- [ ] Create 3 demo accounts: `guest@demo.com`, `host@demo.com`, `admin@demo.com`
- [ ] Write README with live URL, screenshots, and setup instructions
- [ ] Confirm live URL is working and seeded data is visible
- [ ] Prepare and rehearse 60-second Demo Day pitch

---

## 12. Known Constraints

| Constraint | Decision |
|---|---|
| No map view | Too much complexity for MVP. Venue address shown as text only. |
| No real-time updates | Booking status shown on page refresh. No WebSockets. |
| No automated "mark as completed" | Host or admin manually marks a confirmed booking as completed after the event date. Automate in Phase 2. |
| MongoDB free tier (512MB) | Sufficient for Demo Day and the months after. Upgrade when needed. |
| Resend free tier (3,000 emails/month) | Sufficient for testing and MVP launch. |
| No image compression | Cloudinary serves optimised versions automatically. No manual compression needed. |
| No booking cancellation policy | Cancellation simply changes status to `cancelled`. Refund policies are outside scope. |

---

*OneEvent TTD v1.0 — Phase 1 — June 2026*