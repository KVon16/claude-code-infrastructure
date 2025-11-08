# Quick Start Checklist - Super Feynman

Step-by-step implementation guide for hackathon success.

---

## Pre-Development Setup (30 minutes)

### ☐ 1. Project Initialization

```bash
# Create project directory
mkdir super-feynman
cd super-feynman

# Create backend
mkdir backend
cd backend
npm init -y
npm install express cors dotenv better-sqlite3 multer @anthropic-ai/sdk openai express-rate-limit
npm install -D nodemon

# Create frontend
cd ..
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
npm install react-router-dom axios
```

### ☐ 2. Environment Setup

```bash
# Backend .env
cd backend
echo "PORT=3001
NODE_ENV=development
ANTHROPIC_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here" > .env

# Frontend .env
cd ../frontend
echo "VITE_API_URL=http://localhost:3001/api" > .env
```

### ☐ 3. Folder Structure

```bash
cd backend
mkdir -p src/{controllers,services,routes,db,middleware,utils}
mkdir -p uploads/{audio,notes}

cd ../frontend
# src/ already exists from Vite
mkdir -p src/{components,pages,services,hooks,utils}
```

---

## Backend Development (3-4 hours)

### ☐ Phase 1: Database Setup (30 min)

**Priority: HIGH**

- [ ] Create `src/db/schema.sql` with all 4 tables
- [ ] Create `src/db/database.js` (SQLite connection)
- [ ] Create `src/db/queries.js` (all CRUD functions)
- [ ] Test database connection

**Files to create:**
- ✅ `backend/src/db/schema.sql`
- ✅ `backend/src/db/database.js`
- ✅ `backend/src/db/queries.js`

**Test:**
```bash
node -e "const db = require('./src/db/database'); console.log('DB works!');"
```

---

### ☐ Phase 2: Base Infrastructure (30 min)

**Priority: HIGH**

- [ ] Create `BaseController.js`
- [ ] Create error handler middleware
- [ ] Create basic Express app setup
- [ ] Create server entry point

**Files to create:**
- ✅ `backend/src/controllers/BaseController.js`
- ✅ `backend/src/middleware/errorHandler.js`
- ✅ `backend/src/app.js`
- ✅ `backend/src/server.js`

**Test:**
```bash
npm run dev
curl http://localhost:3001/health
```

---

### ☐ Phase 3: Course & Lecture Routes (1 hour)

**Priority: HIGH**

- [ ] Create `CourseController.js` (get, create, delete)
- [ ] Create `courseService.js`
- [ ] Create `courseRoutes.js`
- [ ] Create `LectureController.js` (get, create, delete)
- [ ] Create `lectureService.js`
- [ ] Create `lectureRoutes.js`
- [ ] Set up Multer for file uploads

**Files to create:**
- ✅ `backend/src/controllers/CourseController.js`
- ✅ `backend/src/services/courseService.js`
- ✅ `backend/src/routes/courseRoutes.js`
- ✅ `backend/src/controllers/LectureController.js`
- ✅ `backend/src/services/lectureService.js`
- ✅ `backend/src/routes/lectureRoutes.js`
- ✅ `backend/src/middleware/upload.js`

**Test:**
```bash
# Create course
curl -X POST http://localhost:3001/api/courses -H "Content-Type: application/json" -d '{"name":"Test Course"}'

# Get courses
curl http://localhost:3001/api/courses
```

---

### ☐ Phase 4: Anthropic Integration (1.5 hours)

**Priority: HIGH**

- [ ] Create `anthropicService.js`
- [ ] Implement `generateConcepts()` method
- [ ] Integrate concept generation into lecture creation
- [ ] Create `ConceptController.js` (get concepts, update progress)
- [ ] Create `conceptService.js`
- [ ] Create `conceptRoutes.js`

**Files to create:**
- ✅ `backend/src/services/anthropicService.js`
- ✅ `backend/src/controllers/ConceptController.js`
- ✅ `backend/src/services/conceptService.js`
- ✅ `backend/src/routes/conceptRoutes.js`

**Test:**
```bash
# Create lecture with concept generation
# Upload a .txt file and verify concepts are auto-generated
```

---

### ☐ Phase 5: Review Session Logic (1 hour)

**Priority: MEDIUM**

- [ ] Implement `startReviewConversation()` in anthropicService
- [ ] Implement `continueReviewConversation()` in anthropicService
- [ ] Implement `generateFeedback()` in anthropicService
- [ ] Add review routes to ConceptController
- [ ] Add rate limiting middleware

**Updates:**
- ✅ `backend/src/services/anthropicService.js` (add 3 new methods)
- ✅ `backend/src/controllers/ConceptController.js` (add review endpoints)
- ✅ `backend/src/middleware/rateLimiter.js`

**Test:**
```bash
# Start review session
curl -X POST http://localhost:3001/api/concepts/review/start \
  -H "Content-Type: application/json" \
  -d '{"conceptId":1,"audienceLevel":"classmate"}'
```

---

### ☐ Phase 6: Whisper Integration (1 hour)

**Priority: LOW** (can skip for MVP if time-constrained)

- [ ] Create `whisperService.js`
- [ ] Create transcribe route
- [ ] Set up audio file upload

**Files to create:**
- ✅ `backend/src/services/whisperService.js`
- ✅ `backend/src/routes/transcribeRoutes.js`

**Test:**
```bash
# Upload audio file
curl -X POST http://localhost:3001/api/transcribe -F "audio=@test.webm"
```

---

## Frontend Development (3-4 hours)

### ☐ Phase 1: Tailwind & Routing Setup (30 min)

**Priority: HIGH**

- [ ] Configure Tailwind CSS
- [ ] Set up React Router
- [ ] Create basic App.jsx structure

**Files to update:**
- ✅ `frontend/tailwind.config.js`
- ✅ `frontend/src/index.css`
- ✅ `frontend/src/App.jsx`

---

### ☐ Phase 2: Reusable Components (45 min)

**Priority: HIGH**

- [ ] Create `LoadingSpinner.jsx`
- [ ] Create `Modal.jsx`
- [ ] Create `Header.jsx`
- [ ] Create `ErrorMessage.jsx`

**Files to create:**
- ✅ `frontend/src/components/LoadingSpinner.jsx`
- ✅ `frontend/src/components/Modal.jsx`
- ✅ `frontend/src/components/Header.jsx`
- ✅ `frontend/src/components/ErrorMessage.jsx`

---

### ☐ Phase 3: API Service Layer (30 min)

**Priority: HIGH**

- [ ] Create `api.js` with axios setup
- [ ] Implement courseApi methods
- [ ] Implement lectureApi methods
- [ ] Implement conceptApi methods
- [ ] Implement reviewApi methods

**Files to create:**
- ✅ `frontend/src/services/api.js`

**Test:**
```javascript
// In browser console
import { courseApi } from './services/api';
courseApi.getAll().then(console.log);
```

---

### ☐ Phase 4: Custom Hooks (30 min)

**Priority: MEDIUM**

- [ ] Create `useCourses.js`
- [ ] Create `useLectures.js`
- [ ] Create `useConcepts.js`
- [ ] Create `useAudioRecorder.js` (if implementing Whisper)

**Files to create:**
- ✅ `frontend/src/hooks/useCourses.js`
- ✅ `frontend/src/hooks/useLectures.js`
- ✅ `frontend/src/hooks/useConcepts.js`
- ⚠️ `frontend/src/hooks/useAudioRecorder.js` (optional)

---

### ☐ Phase 5: Page Components (2 hours)

**Priority: HIGH**

Build pages in this order:

**1. Home Page (30 min)**
- [ ] Display course list
- [ ] Add course modal
- [ ] Delete course functionality
- [ ] Navigate to course detail

**Files:**
- ✅ `frontend/src/pages/Home.jsx`

---

**2. Course Detail Page (30 min)**
- [ ] Display lecture list
- [ ] Add lecture modal (with file upload)
- [ ] Delete lecture functionality
- [ ] Navigate to lecture detail
- [ ] Loading state while concepts generate

**Files:**
- ✅ `frontend/src/pages/CourseDetail.jsx`

---

**3. Lecture Detail Page (30 min)**
- [ ] Display concept list with status badges
- [ ] Sort by last reviewed
- [ ] Delete concept functionality
- [ ] Audience selection modal
- [ ] Navigate to review session

**Files:**
- ✅ `frontend/src/pages/LectureDetail.jsx`

---

**4. Review Session Page (30 min)**
- [ ] Chat interface layout
- [ ] Send message functionality
- [ ] Display conversation history
- [ ] Microphone button (if implementing Whisper)
- [ ] End session button
- [ ] Navigate to feedback screen

**Files:**
- ✅ `frontend/src/pages/ReviewSession.jsx`

---

**5. Feedback Screen (30 min)**
- [ ] Display feedback sections
- [ ] Progress update banner
- [ ] Retry button
- [ ] Back to concepts button

**Files:**
- ✅ `frontend/src/pages/FeedbackScreen.jsx`

---

### ☐ Phase 6: Styling Polish (1 hour)

**Priority: MEDIUM**

- [ ] Refine Tailwind classes for consistency
- [ ] Add hover states and transitions
- [ ] Responsive design adjustments
- [ ] Color-coded status badges
- [ ] Loading animations

---

## Testing & Bug Fixes (1-2 hours)

### ☐ End-to-End Testing

**Full User Flow:**

1. [ ] Create a course
2. [ ] Navigate into course
3. [ ] Upload a .txt lecture file
4. [ ] Wait for concepts to generate
5. [ ] View concepts list
6. [ ] Click on a concept
7. [ ] Select audience level
8. [ ] Have a conversation (5-10 turns)
9. [ ] End session
10. [ ] View feedback
11. [ ] Verify progress status updated
12. [ ] Go back to concepts list
13. [ ] Delete concept, lecture, course

### ☐ Error Scenarios

- [ ] Upload non-.txt file (should reject)
- [ ] Submit empty form (should validate)
- [ ] Network failure (should show error)
- [ ] Missing API keys (should fail gracefully)
- [ ] Invalid concept ID (should 404)

### ☐ Bug Fixes

- [ ] Fix any errors found during testing
- [ ] Add missing loading states
- [ ] Fix layout issues
- [ ] Improve error messages

---

## Optional Enhancements (If Time Permits)

### ☐ Speech-to-Text

- [ ] Implement full Whisper integration
- [ ] Add microphone button to chat
- [ ] Handle audio recording
- [ ] Display transcription status

### ☐ UI Polish

- [ ] Add animations
- [ ] Improve empty states
- [ ] Add icons
- [ ] Improve mobile responsiveness

### ☐ Additional Features

- [ ] Edit course/lecture names
- [ ] Search functionality
- [ ] Filter concepts by status
- [ ] Export progress data

---

## Deployment (Optional)

### ☐ Backend Deployment

**Platforms:**
- Render.com (free tier)
- Railway.app (free tier)
- Heroku (free tier ended)

**Steps:**
- [ ] Add `Procfile` or `render.yaml`
- [ ] Set environment variables in platform
- [ ] Deploy and test

### ☐ Frontend Deployment

**Platforms:**
- Vercel (recommended)
- Netlify
- GitHub Pages

**Steps:**
```bash
cd frontend
npm run build
# Deploy dist/ folder
```

---

## Hackathon Presentation Prep

### ☐ Demo Preparation

- [ ] Create sample course with real data
- [ ] Pre-upload a lecture with concepts
- [ ] Practice the demo flow
- [ ] Prepare talking points

### ☐ Documentation

- [ ] Update README with:
  - [ ] Project description
  - [ ] Tech stack
  - [ ] Setup instructions
  - [ ] API documentation
  - [ ] Screenshots

---

## Time Estimates

| Phase | Time | Priority |
|-------|------|----------|
| Project setup | 30 min | HIGH |
| Database layer | 30 min | HIGH |
| Base infrastructure | 30 min | HIGH |
| Course/Lecture APIs | 1 hour | HIGH |
| Anthropic integration | 1.5 hours | HIGH |
| Review session logic | 1 hour | MEDIUM |
| Whisper integration | 1 hour | LOW |
| Frontend setup | 30 min | HIGH |
| Components & hooks | 1 hour | HIGH |
| Page components | 2 hours | HIGH |
| Styling | 1 hour | MEDIUM |
| Testing | 1-2 hours | HIGH |
| **TOTAL** | **10-12 hours** | - |

---

## Minimum Viable Product (MVP)

If you're short on time, focus on these essentials:

### Must Have ✅
- Course CRUD
- Lecture upload with concept generation
- Concept list display
- Text-based chat review (no Whisper)
- Basic feedback display
- Progress tracking

### Can Skip ⚠️
- Speech-to-text (Whisper)
- Advanced styling
- Animations
- Edit functionality
- Search/filter

---

## Troubleshooting Common Issues

### Backend won't start
- Check `.env` file exists
- Verify API keys are valid
- Ensure port 3001 is free
- Check database file permissions

### Frontend can't connect to backend
- Verify backend is running
- Check CORS configuration
- Verify `VITE_API_URL` in `.env`
- Check browser console for errors

### Concepts not generating
- Verify Anthropic API key
- Check API request in network tab
- Look for JSON parsing errors
- Check backend logs

### Audio recording not working
- Check browser microphone permissions
- Verify Whisper API key
- Check file upload size limits
- Test with different browsers

---

## Success Criteria

### Core Features Working ✅
- [x] Create courses
- [x] Upload lectures and generate concepts
- [x] Interactive review conversations
- [x] Feedback and progress tracking
- [x] Clean, usable UI

### Bonus Points 🌟
- [x] Speech-to-text working
- [x] Beautiful UI/UX
- [x] Smooth animations
- [x] Mobile responsive
- [x] Error handling

---

## Final Checklist Before Submission

- [ ] All core features working
- [ ] No console errors
- [ ] Clean code (no commented-out code)
- [ ] Environment variables documented
- [ ] README updated
- [ ] Screenshots added
- [ ] Demo prepared
- [ ] Code commented
- [ ] Git commits clear and organized

---

Good luck with your hackathon! Follow this checklist and you'll have a working Super Feynman app in no time! 🚀

**Pro tip:** Start early, test often, and don't be afraid to simplify if you're running short on time. A working MVP is better than a half-finished feature-rich app!
