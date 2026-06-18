# Product Requirements Document (PRD) - DAU Event Booking System (Sleazzy)

## 1. Product Overview
The **DAU Event Booking System** (formerly known as Sleazzy) is a comprehensive campus venue and event management application designed specifically for **Dhirubhai Ambani University (DA-IICT)**. It facilitates the scheduling, management, and public display of events organized by various university clubs and committees.

The platform streamlines the process of venue allocation by providing clubs with a dashboard to request slots, and administrators with a centralized control panel to approve, reject, and manage these requests. Additionally, it offers a public-facing calendar for the student body to discover upcoming activities.

## 2. Target Audience & User Roles
The system utilizes a role-based access control (RBAC) architecture with three primary perspectives:

1. **Public Users (Unauthenticated):**
   - Can view the public "Happening on Campus" calendar.
   - Can filter and discover approved, public-facing events (Co-Curricular and Open for All).
   - Cannot see `closed_club` events.

2. **Club Representatives (Role: `club`):**
   - Can register and log in to a dedicated Club Dashboard.
   - Can request venue bookings (specifying time, venue, event name, and event type).
   - Can view their booking history and current status (Pending, Approved, Rejected).
   - Can manage their club's committee members (e.g., Convenor, Dy. Convener, Core).
   - Can upload post-event documents (indents, reports).

3. **Administrators (Role: `admin`):**
   - Have full control over the platform via the Admin Dashboard.
   - Review pending booking requests and approve/reject them.
   - Manage the list of available venues and registered clubs.
   - View global master schedules to resolve conflicts.

## 3. Technology Stack
The application is built on a modern, full-stack TypeScript architecture.

### Frontend
- **Framework:** React 18 with Vite.
- **Styling:** Tailwind CSS with a custom-built "Aurora" glassmorphism design system.
- **Brand Theme:** DAU Signature Colors (Navy Blue `#2C3E8A`, Red-Orange `#E84E36`, Yellow `#FDC02F`). Supports dynamic Light/Dark/System modes.
- **State Management & Forms:** React Hook Form, Zod (validation).
- **Key Libraries:** `framer-motion` (animations), `react-big-calendar` (scheduling), `lucide-react` (icons), `socket.io-client` (real-time).

### Backend
- **Framework:** Node.js with Express.
- **Database:** PostgreSQL (Neon DB). Connected via `pg` pool.
- **Authentication:** Custom JWT-based authentication (replaces previous Supabase Auth).
- **Real-time:** `socket.io` for live booking status updates and notifications.

## 4. Core Features & System Workflows

### 4.1. Venue Booking Workflow
1. **Request:** A club logs in, selects a venue, and submits a booking request specifying the date, time, and event details.
2. **Pending State:** The booking is marked as `pending` and appears in the Admin's queue.
3. **Review:** The admin reviews the request against the master schedule.
4. **Resolution:** The admin approves or rejects the request. 
5. **Notification:** A real-time WebSocket event (`booking:status_changed`) is emitted to the club, triggering an instant toast notification on their dashboard.

### 4.2. Event Visibility & Types
Events are categorized into three privacy tiers:
- **Co-Curricular:** Publicly visible on the Landing Page.
- **Open for All:** Publicly visible on the Landing Page.
- **Closed Club:** Excluded from the `/api/public-bookings` endpoint. Only visible to the club members internally.

### 4.3. Club Roster Management
Clubs can maintain a directory of their committee members. The system supports custom designations (Convenor, Dy. Convener, Core, etc.) and allows clubs to upload member avatars and details.

### 4.4. Theming & UI/UX
The UI utilizes a highly polished, interactive design system generated via UI-UX Pro Max.
- **Glassmorphism:** Frosted glass cards and modals (`.glass-card`).
- **Dynamic Radial Backgrounds:** Ambient background blobs that animate slowly, strictly adhering to the DAU tri-color palette.
- **Micro-interactions:** Smooth hover states, popovers, and loading shimmers implemented via Framer Motion.

## 5. Database Schema Structure
The PostgreSQL database consists of the following primary tables (found in `server/migrations`):

- `auth.users` / `public.profiles`: Stores core user credentials and RBAC roles (`admin`, `club`).
- `public.clubs`: Stores club metadata (`name`, `group_category`, `email`). Linked to profiles.
- `public.venues`: Represents bookable spaces.
- `public.bookings`: The core transactional table. Tracks `club_id`, `venue_id`, `start_time`, `end_time`, `status` (pending/approved/rejected), and `event_type`.
- `public.events`: Derived table for public-facing event metadata.
- `public.club_members`: Tracks individual students belonging to specific clubs.
- `public.notifications`: Stores persistent alerts for users.

## 6. API Architecture
The backend is structured into modular Express routers (`server/src/routes/`):
- `/api/auth/*`: Registration, Login (JWT issuance), Profile fetching.
- `/api/admin/*`: Protected routes for admin operations (fetching pending bookings, approving/rejecting).
- `/api/public-bookings`: Unauthenticated route for the public calendar (filters out closed club events).
- `/api/bookings`: Protected routes for clubs to create and manage their own bookings.
- `/api/club-members`: CRUD operations for club rosters.
- `/api/events`: General event management.

## 7. Future Extensibility & Agent Guidelines
For AI agents interacting with this repository in the future:
1. **Styling Edits:** Always respect the DAU color palette defined in `client/src/index.css`. Do not introduce arbitrary colors (like purple or pink) unless explicitly requested. Ensure `dark` mode variants maintain accessibility contrast.
2. **Database Queries:** The backend uses raw SQL queries via the `pg` pool (`req.app.locals.db`). Avoid introducing ORMs unless part of a major refactor. Always use parameterized queries (`$1, $2`) to prevent SQL injection.
3. **Real-time Events:** If adding new features that require immediate user feedback, utilize the existing `socket.io` infrastructure defined in `server/src/server.ts`.
4. **Authentication:** Do not revert to Supabase Auth. The system relies on standard JWT tokens passed via HTTP headers and Socket auth handshakes.

## 8. Setup & Run Instructions

### Prerequisites
- Node.js (v18 or higher)
- npm or yarn or pnpm
- PostgreSQL Database URL (e.g., Neon DB)

### Environment Variables
1. **Server (`server/.env`)**:
   ```env
   DATABASE_URL="postgres://user:password@host/dbname"
   JWT_SECRET="your_secure_random_string"
   PORT=4000
   ```
2. **Client (`client/.env`)**:
   *(If applicable, specify any `VITE_API_URL` or equivalent)*

### Installation & Execution (All Systems: Windows, macOS, Linux)

To run the full stack locally during development, you will need two terminal instances.

**Terminal 1: Start the Backend (Server)**
```bash
cd server
npm install
npm run dev
```

**Terminal 2: Start the Frontend (Client)**
```bash
cd client
npm install
npm run dev
```

### Production Build
To create a production build of the frontend and run the server:
```bash
# 1. Build the client
cd client
npm run build

# 2. Serve from backend
cd ../server
npm install
npm start
```
