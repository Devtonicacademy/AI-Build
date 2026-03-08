# School LMS (Antigravity Build Spec)

## Persona
You are an expert full-stack engineer and live-demo builder.  
You build small, polished apps that schools would actually use and show off.  
You prioritize clear scope, predictable outcomes, minimal moving parts, and clean, engaging UI with subtle animations.  
You write readable code and avoid unnecessary abstractions.

## Objective
Create a full-fledged but initially minimal **Learning Management System (LMS)** web app for schools.  
Core MVP lets schools manage courses, enroll students, track progress — with multi-school branding support so the same codebase can serve different institutions.

Data persists in **Firebase Firestore** (real-time, serverless, scalable for schools).

## Scope

### Include
- Multi-school support via dynamic branding (primary color, logo, name fetched from Firestore per school slug)
- Authentication (Firebase Auth – email/password + Google sign-in)
- Roles: Admin (school owner), Instructor, Student (simple role field in user doc)
- CRUD for Courses (title, description, modules as array/subcollection)
- Student enrollment in courses
- Basic progress tracking (e.g., mark module complete → percentage)
- Real-time updates (Firestore listeners)
- Search/filter courses by name
- Clean, animated UI (Framer Motion for page transitions, card hovers, list items)
- Responsive design (mobile-friendly for students/parents)

### Exclude (for MVP – add later)
- Payments / subscriptions
- Video hosting / streaming (embed YouTube/Vimeo)
- Quizzes / assessments (basic later)
- Forums / discussions
- Advanced analytics / reporting
- Parent/teacher portals as separate apps
- Over-engineered state management (use React Context + Firestore for now)

## Tech Stack (Preferred)
- **Frontend + Framework**: Next.js 14+ (App Router) + TypeScript
- **Styling**: Tailwind CSS + Shadcn/UI components (tables, cards, dialogs, toasts)
- **Animations**: Framer Motion (smooth page loads, button hovers, list animations)
- **Backend / DB**: Firebase (Firestore for data, Auth, Storage for logos/files)
- **Icons**: Lucide React
- **Form handling**: React Hook Form + Zod (validation)
- **Toasts**: Sonner or react-hot-toast
- **Deployment target**: Firebase Hosting (one-command deploy)

No heavy state libs (Redux/Zustand) unless needed later.

## Data Model

### Collections / Documents

**schools** (top-level)
- id: string (slug, e.g. "lagos-high")
- name: string (required)
- primaryColor: string (hex, e.g. "#007bff")
- logoUrl: string (Firebase Storage path)
- createdAt: timestamp

**users** (auth.uid as doc id)
- email: string
- displayName: string
- role: "admin" | "instructor" | "student"
- schoolId: string (references schools.id)
- createdAt: timestamp

**courses** (per school – can use collectionGroup or subcollection under schools/{schoolId}/courses)
- id: string (auto)
- title: string (required)
- description: string
- instructorId: string (user id)
- modules: array<{ title: string, content: string, type: "text"|"video"|"pdf", completedBy?: string[] }>
- enrolledStudents: array<string> (user ids) – or subcollection enrollments
- createdAt: timestamp
- updatedAt: timestamp

**enrollments** (optional subcollection courses/{courseId}/enrollments or separate)
- studentId: string
- progress: number (0-100)
- completedModules: array<string>

Use Firestore security rules to enforce: only school admins can create courses, students can only read/enroll in their school, etc.

## UI Page Layout (Single-page SPA feel with Next.js App Router)

- `/[schoolSlug]/` → Landing / public course catalog (branded)
- `/[schoolSlug]/auth/login` → Simple login
- `/[schoolSlug]/dashboard` → Role-based home
  - Admin: Create course button, list of courses
  - Student: Enrolled courses + progress cards
- `/[schoolSlug]/courses` → List all courses (search bar)
- `/[schoolSlug]/courses/[courseId]` → Course detail (modules list, enroll button if not enrolled)
- `/[schoolSlug]/admin/courses/new` → Add course form

Use root layout to fetch school branding → apply CSS vars (`--primary: var(--school-primary)`)

Empty state: Friendly message + illustration when no courses/enrollments

## Add Course / Module (Admin)

Inputs:
- Title (required)
- Description
- Modules (dynamic add/remove): title + content (text or embed url)

Button: "Create Course"

## Inventory-like Actions on Courses

- Table view: Title, Instructor, Enrolled count, Actions (Edit, Delete)
- Inline edit: e.g. change module title
- Delete: Confirm dialog
- Search: by title (client-side filter or Firestore query)

## Validation & Styling

- Clean, minimal Tailwind + Shadcn
- Required fields: red border + inline error
- Quantity-like (e.g. progress): number input, prevent negative
- User-friendly errors near fields (e.g. "Title cannot be empty")
- Framer Motion: fade-in pages, scale on card hover, AnimatePresence for adding modules

## Persistence & Setup

- Use Firebase SDK + React hooks (useFirestoreQuery, etc.)
- Include simple seed script: create 3–5 sample courses + users for a demo school
- Security rules stub (authenticated + schoolId match)

## Definition of Done

A user (after login):
1. Can see branded dashboard for their school
2. Admin can add a course → see it in list (real-time)
3. Student can enroll → see progress
4. Update progress/module → persists
5. Delete course (admin) → gone after confirm
6. Refresh page → data still there (Firestore)
7. Animations feel smooth and delightful
8. No console errors, mobile looks good

Vibe: Polished enough to demo to a principal tomorrow – clean, fast, real-time magic with Firebase.

Built for Lagos school vibes! 🚀
