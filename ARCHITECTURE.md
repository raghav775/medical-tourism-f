# Technical Architecture Documentation

## System Overview

Global Health Travellers is a **frontend-only** medical tourism platform built with modern web technologies, designed for seamless backend integration.

---

## Technology Stack Rationale

### Why Next.js 14?

**Chosen for:**
1. **App Router**: Modern routing with layouts and server components support
2. **Built-in Optimization**: Automatic code splitting, image optimization
3. **SEO-Friendly**: Server-side rendering capabilities for medical content
4. **Production-Ready**: Zero-config deployment, excellent DX
5. **Healthcare-Specific Benefits**:
   - Fast page loads crucial for international users on varying connections
   - Built-in security features (CSRF protection, XSS prevention)
   - Scalable architecture for growing medical network

### Why TypeScript?

**Chosen for:**
1. **Type Safety**: Critical for medical data (patient info, appointments)
2. **IDE Support**: IntelliSense reduces bugs in complex booking flows
3. **Maintainability**: Easier for teams to collaborate on healthcare platform
4. **API Integration**: Strong typing for backend communication

### Why Tailwind CSS?

**Chosen for:**
1. **Rapid Development**: Build medical-grade UI quickly
2. **Consistency**: Design tokens ensure uniform healthcare UX
3. **Responsive**: Mobile-first approach for international users
4. **Customization**: Easy to create medical color palette
5. **Performance**: Purged CSS = smaller bundle sizes

### Why Framer Motion?

**Chosen for:**
1. **Smooth Animations**: Professional feel essential for trust
2. **Accessibility**: Respects `prefers-reduced-motion`
3. **React Integration**: Declarative animations for complex flows
4. **Performance**: Hardware-accelerated transforms

---

## Architecture Patterns

### Component Architecture

```
┌─────────────────────────────────────┐
│          App Router (Next.js)       │
│  ┌─────────────────────────────┐   │
│  │     Root Layout (layout.tsx) │   │
│  │  - Global styles             │   │
│  │  - Metadata                  │   │
│  │  - Font loading              │   │
│  └─────────────────────────────┘   │
│              │                      │
│              ▼                      │
│  ┌─────────────────────────────┐   │
│  │       Page (page.tsx)        │   │
│  │  - Component composition     │   │
│  │  - Section orchestration     │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│         Component Layer              │
│  ┌──────────┐  ┌──────────┐         │
│  │Navigation│  │   Hero   │         │
│  └──────────┘  └──────────┘         │
│  ┌──────────┐  ┌──────────┐         │
│  │Treatments│  │  Doctors │         │
│  └──────────┘  └──────────┘         │
│  ┌──────────┐  ┌──────────┐         │
│  │Hospitals │  │Consult.  │         │
│  └──────────┘  └──────────┘         │
│  ┌──────────┐  ┌──────────┐         │
│  │  Trust   │  │  Travel  │         │
│  └──────────┘  └──────────┘         │
│  ┌──────────┐                        │
│  │  Footer  │                        │
│  └──────────┘                        │
└─────────────────────────────────────┘
```

### State Management Pattern

**Current (Frontend-Only):**
```typescript
// Local component state
const [formData, setFormData] = useState({...})
const [activeFilter, setActiveFilter] = useState('all')
```

**For Backend Integration:**
```typescript
// Global state with Context API
const { user, login, logout } = useAuth()
const { bookings } = useBookings()

// Or with Zustand/Redux
const treatments = useTreatmentStore(state => state.treatments)
```

### Data Flow Pattern

**Frontend-Only (Current):**
```
User Interaction → Component State → UI Update
```

**With Backend (Future):**
```
User Interaction → API Call → Loading State → 
Backend Response → Update State → UI Update
```

---

## Component Design Principles

### 1. **Separation of Concerns**
Each component has a single responsibility:
- `Navigation`: Header and routing
- `Hero`: Trust-building and search
- `Treatments`: Treatment discovery
- `ConsultationFlow`: Booking process

### 2. **Composition over Inheritance**
Components are composed, not extended:
```typescript
// Good: Composition
<Card>
  <CardHeader />
  <CardBody />
</Card>

// Avoid: Inheritance
class DoctorCard extends Card {...}
```

### 3. **Props Drilling Prevention**
For deeply nested data, use:
- Context API for auth/theme
- Props for direct parent-child communication

### 4. **Accessibility First**
- Semantic HTML elements
- ARIA labels where needed
- Keyboard navigation support
- Focus management

---

## Styling Architecture

### CSS Cascade Strategy

```
1. Tailwind Base Layer (globals.css)
   ↓
2. Component Layer (@layer components)
   ↓
3. Utility Layer (@layer utilities)
   ↓
4. Component-Specific Styles
```

### Design Token System

```css
:root {
  /* Semantic colors */
  --color-trust: 14 165 233;
  --color-calm: 34 197 94;
  --color-care: 245 158 11;
  
  /* Shadows */
  --shadow-card: ...;
  --shadow-hover: ...;
}
```

### Responsive Design Strategy

**Mobile-First Approach:**
```jsx
// Base styles = mobile
<div className="px-4 py-8 
  // Tablet
  md:px-6 md:py-12 
  // Desktop
  lg:px-8 lg:py-16"
>
```

---

## Animation Strategy

### Performance-First Animations

**Good (Hardware Accelerated):**
```css
transform: translateY(20px);
opacity: 0;
```

**Avoid (Repaints):**
```css
margin-top: 20px;
height: auto;
```

### Animation Hierarchy

1. **Page Load**: Staggered reveals create flow
2. **User Interaction**: Hover, click feedback
3. **State Changes**: Smooth transitions
4. **Micro-interactions**: Subtle delight

---

## Backend Integration Architecture

### Recommended API Structure

```
/api
├── /auth
│   ├── /register
│   ├── /login
│   └── /logout
├── /treatments
│   ├── GET    /             (list)
│   ├── GET    /:id          (detail)
│   └── GET    /categories   (filters)
├── /doctors
│   ├── GET    /             (list)
│   ├── GET    /:id          (detail)
│   └── GET    /search       (filter)
├── /hospitals
│   ├── GET    /             (list)
│   ├── GET    /:id          (detail)
│   └── GET    /accredited   (filter)
├── /consultations
│   ├── POST   /             (create)
│   ├── GET    /:id          (detail)
│   ├── PATCH  /:id          (update)
│   └── DELETE /:id          (cancel)
└── /users
    ├── GET    /me           (profile)
    ├── PATCH  /me           (update)
    └── GET    /bookings     (history)
```

### Authentication Flow

```
1. User Registration
   POST /api/auth/register
   ↓
2. Email Verification
   (backend sends email)
   ↓
3. Login
   POST /api/auth/login
   ↓
4. Store JWT in httpOnly cookie
   ↓
5. Subsequent Requests
   Include cookie automatically
   ↓
6. Logout
   POST /api/auth/logout
   (clear cookie)
```

### Data Fetching Strategy

**Server Components (when available):**
```typescript
// app/doctors/page.tsx
async function DoctorsPage() {
  const doctors = await fetch('/api/doctors')
  return <DoctorsList doctors={doctors} />
}
```

**Client Components (current):**
```typescript
'use client'
import { useEffect, useState } from 'react'

function DoctorsPage() {
  const [doctors, setDoctors] = useState([])
  
  useEffect(() => {
    fetch('/api/doctors')
      .then(res => res.json())
      .then(setDoctors)
  }, [])
  
  return <DoctorsList doctors={doctors} />
}
```

---

## Security Architecture

### Frontend Security Measures

**1. Input Validation**
```typescript
// Validate all form inputs
const validateEmail = (email: string) => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}
```

**2. XSS Prevention**
```typescript
// React automatically escapes values
<div>{userInput}</div> // Safe

// Dangerous: dangerouslySetInnerHTML
// Only use with sanitized content
```

**3. CSRF Protection**
```typescript
// Next.js has built-in CSRF protection
// Ensure all POST requests use proper headers
```

**4. Sensitive Data**
```typescript
// Never store in localStorage
// ❌ localStorage.setItem('token', jwt)

// Use httpOnly cookies instead
// ✅ Set-Cookie: token=...; HttpOnly; Secure; SameSite=Strict
```

### Backend Security Requirements

**Must Implement:**
1. Rate limiting (prevent abuse)
2. Input sanitization (SQL injection)
3. Password hashing (bcrypt)
4. JWT expiration (short-lived tokens)
5. HTTPS only (TLS/SSL)
6. CORS configuration (allowed origins)
7. Data encryption at rest
8. Audit logging (track access)

---

## Performance Optimization

### Current Optimizations

1. **Code Splitting**
   - Automatic route-based splitting by Next.js
   - Dynamic imports for heavy components

2. **Image Optimization**
   - Ready for next/image component
   - WebP format support
   - Lazy loading below fold

3. **CSS Optimization**
   - Tailwind purge removes unused styles
   - Critical CSS inlined
   - PostCSS minification

4. **Animation Performance**
   - CSS transforms (GPU accelerated)
   - will-change hints for complex animations
   - Reduced motion support

### Future Optimizations

1. **Caching Strategy**
```typescript
// Cache treatments, doctors, hospitals
const SWR_CONFIG = {
  revalidateOnFocus: false,
  dedupingInterval: 60000,
}
```

2. **API Response Caching**
```typescript
// Service worker for offline support
// Cache API responses for 5 minutes
```

3. **Image CDN**
```typescript
// Use Cloudinary or similar
const imageUrl = `${CDN_URL}/w_800,q_auto,f_auto/doctor-image.jpg`
```

---

## Testing Strategy

### Unit Tests (Component Level)

```typescript
// Example: Treatments.test.tsx
import { render, screen } from '@testing-library/react'
import Treatments from './Treatments'

test('renders treatment cards', () => {
  render(<Treatments />)
  expect(screen.getByText('Cardiology')).toBeInTheDocument()
})
```

### Integration Tests (User Flows)

```typescript
// Example: booking-flow.test.tsx
test('completes consultation booking', async () => {
  const user = userEvent.setup()
  render(<ConsultationFlow />)
  
  // Step 1: Select treatment
  await user.click(screen.getByText('Cardiology'))
  await user.click(screen.getByText('Continue'))
  
  // Step 2: Schedule
  // ... and so on
})
```

### E2E Tests (Full Application)

```typescript
// Example: cypress/e2e/booking.cy.ts
describe('Consultation Booking', () => {
  it('books a consultation successfully', () => {
    cy.visit('/')
    cy.get('[data-testid="book-consultation"]').click()
    // ... complete flow
  })
})
```

---

## Deployment Architecture

### Recommended Stack

**Option 1: Vercel (Easiest)**
```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel
```

**Option 2: Docker + Any Cloud**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
CMD ["npm", "start"]
```

**Option 3: Static Export**
```javascript
// next.config.js
module.exports = {
  output: 'export',
}
```

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm ci
      - run: npm run build
      - run: npm run test
      - uses: vercel/actions/deploy@v1
```

---

## Monitoring & Analytics

### Recommended Tools

**1. Error Tracking**
- Sentry for error monitoring
- Track API failures, rendering errors

**2. Analytics**
- Google Analytics for user behavior
- Mixpanel for conversion tracking
- Hotjar for heatmaps

**3. Performance**
- Lighthouse CI for performance budgets
- Web Vitals tracking (CLS, LCP, FID)

**4. Uptime**
- Pingdom or UptimeRobot
- Alert on downtime

---

## Scalability Considerations

### Horizontal Scaling

**Static Assets:**
```
CDN → Multiple edge locations worldwide
↓
Reduced latency for international patients
```

**API Layer:**
```
Load Balancer
↓
API Server 1 | API Server 2 | API Server 3
↓
Database Cluster (Primary + Replicas)
```

### Database Architecture

**Recommended:**
- PostgreSQL for structured data (patients, doctors)
- Redis for sessions and caching
- S3 for medical documents
- Elasticsearch for search functionality

---

## Compliance & Regulations

### HIPAA Compliance (US)

**Required Measures:**
1. Encrypted data transmission (HTTPS/TLS)
2. Encrypted data at rest
3. Access controls and audit logs
4. Business Associate Agreements (BAAs)
5. Regular security assessments

### GDPR Compliance (EU)

**Required Measures:**
1. User consent for data collection
2. Right to data deletion
3. Data portability
4. Privacy policy transparency
5. Cookie consent management

### Medical Advertising

**Compliance:**
1. Verify all medical claims
2. Include necessary disclaimers
3. Follow country-specific regulations
4. No misleading marketing

---

## Internationalization (i18n)

### Implementation Strategy

```typescript
// Using next-i18next
import { useTranslation } from 'next-i18next'

function Hero() {
  const { t } = useTranslation('common')
  
  return (
    <h1>{t('hero.title')}</h1>
  )
}
```

**Language Files:**
```
/locales
├── /en
│   └── common.json
├── /es
│   └── common.json
├── /fr
│   └── common.json
└── /zh
    └── common.json
```

---

## Cost Estimation (Monthly)

**Hosting (Vercel Pro):** $20/month
**CDN (Cloudflare):** Free - $20/month
**Database (Heroku/AWS):** $25 - $100/month
**Monitoring (Sentry):** $26/month
**Analytics (GA):** Free
**Email Service (SendGrid):** $15/month

**Total Estimated:** $86 - $181/month for small-medium scale

---

## Conclusion

This architecture provides:
- ✅ Production-ready frontend
- ✅ Clear backend integration path
- ✅ Scalability from day one
- ✅ Security best practices
- ✅ Healthcare compliance readiness
- ✅ International user support

The platform is built to handle growth from hundreds to millions of users with proper backend implementation.
