# Backend Development Guide - Super Feynman
## Express.js + JavaScript + SQLite

Adapted from production TypeScript/Prisma patterns for rapid hackathon development.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Core Patterns](#core-patterns)
- [Database Layer](#database-layer)
- [Routes & Controllers](#routes--controllers)
- [Services Layer](#services-layer)
- [Error Handling](#error-handling)
- [API Integration](#api-integration)
- [File Upload](#file-upload)
- [Complete Examples](#complete-examples)

---

## Architecture Overview

### Layered Architecture

```
HTTP Request
    ↓
Routes (routing only)
    ↓
Controllers (request/response handling)
    ↓
Services (business logic)
    ↓
Database (SQLite via better-sqlite3)
```

**Golden Rule:** Each layer has ONE responsibility.

- **Routes**: Define endpoints, attach middleware, delegate to controllers
- **Controllers**: Parse requests, call services, format responses
- **Services**: Business logic, orchestrate operations
- **Database**: Data persistence

---

## Project Structure

```
backend/
├── src/
│   ├── controllers/
│   │   ├── BaseController.js          # Base class for all controllers
│   │   ├── CourseController.js
│   │   ├── LectureController.js
│   │   └── ConceptController.js
│   ├── services/
│   │   ├── courseService.js
│   │   ├── lectureService.js
│   │   ├── conceptService.js
│   │   ├── anthropicService.js        # Claude API integration
│   │   └── whisperService.js          # OpenAI Whisper integration
│   ├── routes/
│   │   ├── courseRoutes.js
│   │   ├── lectureRoutes.js
│   │   └── conceptRoutes.js
│   ├── db/
│   │   ├── database.js                # SQLite connection
│   │   ├── schema.sql                 # Database schema
│   │   └── queries.js                 # SQL query functions
│   ├── middleware/
│   │   ├── errorHandler.js
│   │   ├── upload.js                  # Multer config
│   │   └── validation.js
│   ├── utils/
│   │   ├── logger.js
│   │   └── asyncHandler.js
│   ├── app.js                         # Express setup
│   └── server.js                      # HTTP server
├── uploads/                           # Temporary file storage
├── .env
└── package.json
```

---

## Core Patterns

### 1. Routes Only Route

Routes should ONLY:
- ✅ Define endpoints
- ✅ Attach middleware
- ✅ Delegate to controllers

Routes should NEVER:
- ❌ Contain business logic
- ❌ Access database directly
- ❌ Process complex data

**Example:**

```javascript
// routes/courseRoutes.js
const express = require('express');
const CourseController = require('../controllers/CourseController');

const router = express.Router();
const controller = new CourseController();

// ✅ GOOD: Clean delegation
router.get('/', (req, res) => controller.getCourses(req, res));
router.post('/', (req, res) => controller.createCourse(req, res));
router.delete('/:id', (req, res) => controller.deleteCourse(req, res));

module.exports = router;
```

### 2. Use BaseController

All controllers extend BaseController for consistent error handling and response formatting.

```javascript
// controllers/BaseController.js
class BaseController {
    /**
     * Send success response
     */
    success(res, data, message = null, statusCode = 200) {
        res.status(statusCode).json({
            success: true,
            message,
            data
        });
    }

    /**
     * Send error response
     */
    error(res, error, context, statusCode = 500) {
        console.error(`[${context}] Error:`, error);

        res.status(statusCode).json({
            success: false,
            error: {
                message: error.message || 'An error occurred',
                code: statusCode
            }
        });
    }

    /**
     * Validate required fields
     */
    validateRequired(fields, data, res) {
        const missing = fields.filter(field => !data[field]);

        if (missing.length > 0) {
            res.status(400).json({
                success: false,
                error: {
                    message: 'Missing required fields',
                    details: { missing }
                }
            });
            return false;
        }
        return true;
    }
}

module.exports = BaseController;
```

### 3. Services Contain Business Logic

```javascript
// services/courseService.js
const db = require('../db/database');
const queries = require('../db/queries');

class CourseService {
    async getAllCourses() {
        return queries.getAllCourses();
    }

    async createCourse(name) {
        if (!name || name.trim().length === 0) {
            throw new Error('Course name is required');
        }

        return queries.createCourse(name);
    }

    async deleteCourse(courseId) {
        const course = queries.getCourseById(courseId);
        if (!course) {
            throw new Error('Course not found');
        }

        // Delete associated lectures and concepts
        queries.deleteLecturesByCourseId(courseId);
        return queries.deleteCourse(courseId);
    }
}

module.exports = new CourseService();
```

---

## Database Layer

### Setup SQLite with better-sqlite3

```bash
npm install better-sqlite3
```

### Database Connection

```javascript
// db/database.js
const Database = require('better-sqlite3');
const path = require('path');
const fs = require('fs');

const dbPath = path.join(__dirname, '../../super-feynman.db');
const schemaPath = path.join(__dirname, 'schema.sql');

// Initialize database
const db = new Database(dbPath);
db.pragma('journal_mode = WAL'); // Better concurrency

// Create tables from schema if database is new
if (!fs.existsSync(dbPath)) {
    const schema = fs.readFileSync(schemaPath, 'utf-8');
    db.exec(schema);
    console.log('[Database] Schema created successfully');
}

module.exports = db;
```

### Database Schema

```sql
-- db/schema.sql
CREATE TABLE IF NOT EXISTS courses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS lectures (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    course_id INTEGER NOT NULL,
    name TEXT NOT NULL,
    file_content TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS concepts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    lecture_id INTEGER NOT NULL,
    concept_name TEXT NOT NULL,
    concept_description TEXT,
    progress_status TEXT DEFAULT 'Not Started',
    last_reviewed DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (lecture_id) REFERENCES lectures(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS review_sessions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    concept_id INTEGER NOT NULL,
    audience_level TEXT NOT NULL,
    conversation_history TEXT NOT NULL,
    feedback TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (concept_id) REFERENCES concepts(id) ON DELETE CASCADE
);

-- Indexes for better query performance
CREATE INDEX IF NOT EXISTS idx_lectures_course_id ON lectures(course_id);
CREATE INDEX IF NOT EXISTS idx_concepts_lecture_id ON concepts(lecture_id);
CREATE INDEX IF NOT EXISTS idx_concepts_progress ON concepts(progress_status);
CREATE INDEX IF NOT EXISTS idx_review_sessions_concept_id ON review_sessions(concept_id);
```

### Query Functions

```javascript
// db/queries.js
const db = require('./database');

// Courses
const getAllCourses = () => {
    return db.prepare('SELECT * FROM courses ORDER BY created_at DESC').all();
};

const getCourseById = (id) => {
    return db.prepare('SELECT * FROM courses WHERE id = ?').get(id);
};

const createCourse = (name) => {
    const stmt = db.prepare('INSERT INTO courses (name) VALUES (?)');
    const result = stmt.run(name);
    return getCourseById(result.lastInsertRowid);
};

const deleteCourse = (id) => {
    const stmt = db.prepare('DELETE FROM courses WHERE id = ?');
    return stmt.run(id);
};

// Lectures
const getLecturesByCourseId = (courseId) => {
    return db.prepare('SELECT * FROM lectures WHERE course_id = ? ORDER BY created_at DESC').all(courseId);
};

const createLecture = (courseId, name, fileContent) => {
    const stmt = db.prepare('INSERT INTO lectures (course_id, name, file_content) VALUES (?, ?, ?)');
    const result = stmt.run(courseId, name, fileContent);
    return db.prepare('SELECT * FROM lectures WHERE id = ?').get(result.lastInsertRowid);
};

const deleteLecture = (id) => {
    const stmt = db.prepare('DELETE FROM lectures WHERE id = ?');
    return stmt.run(id);
};

const deleteLecturesByCourseId = (courseId) => {
    const stmt = db.prepare('DELETE FROM lectures WHERE course_id = ?');
    return stmt.run(courseId);
};

// Concepts
const getConceptsByLectureId = (lectureId) => {
    return db.prepare(`
        SELECT * FROM concepts
        WHERE lecture_id = ?
        ORDER BY last_reviewed DESC, created_at DESC
    `).all(lectureId);
};

const createConcept = (lectureId, conceptName, conceptDescription, progressStatus = 'Not Started') => {
    const stmt = db.prepare(`
        INSERT INTO concepts (lecture_id, concept_name, concept_description, progress_status)
        VALUES (?, ?, ?, ?)
    `);
    const result = stmt.run(lectureId, conceptName, conceptDescription, progressStatus);
    return db.prepare('SELECT * FROM concepts WHERE id = ?').get(result.lastInsertRowid);
};

const updateConceptProgress = (conceptId, progressStatus) => {
    const stmt = db.prepare(`
        UPDATE concepts
        SET progress_status = ?, last_reviewed = CURRENT_TIMESTAMP
        WHERE id = ?
    `);
    stmt.run(progressStatus, conceptId);
    return db.prepare('SELECT * FROM concepts WHERE id = ?').get(conceptId);
};

const deleteConcept = (id) => {
    const stmt = db.prepare('DELETE FROM concepts WHERE id = ?');
    return stmt.run(id);
};

// Review Sessions
const createReviewSession = (conceptId, audienceLevel, conversationHistory, feedback = null) => {
    const stmt = db.prepare(`
        INSERT INTO review_sessions (concept_id, audience_level, conversation_history, feedback)
        VALUES (?, ?, ?, ?)
    `);
    const result = stmt.run(
        conceptId,
        audienceLevel,
        JSON.stringify(conversationHistory),
        feedback ? JSON.stringify(feedback) : null
    );
    return db.prepare('SELECT * FROM review_sessions WHERE id = ?').get(result.lastInsertRowid);
};

module.exports = {
    // Courses
    getAllCourses,
    getCourseById,
    createCourse,
    deleteCourse,
    // Lectures
    getLecturesByCourseId,
    createLecture,
    deleteLecture,
    deleteLecturesByCourseId,
    // Concepts
    getConceptsByLectureId,
    createConcept,
    updateConceptProgress,
    deleteConcept,
    // Review Sessions
    createReviewSession
};
```

---

## Routes & Controllers

### Controller Pattern

```javascript
// controllers/LectureController.js
const BaseController = require('./BaseController');
const lectureService = require('../services/lectureService');

class LectureController extends BaseController {
    async getLectures(req, res) {
        try {
            const { courseId } = req.params;
            const lectures = await lectureService.getLecturesByCourseId(courseId);
            this.success(res, lectures);
        } catch (error) {
            this.error(res, error, 'getLectures');
        }
    }

    async createLecture(req, res) {
        try {
            const { courseId, name, fileContent } = req.body;

            // Validate
            if (!this.validateRequired(['courseId', 'name', 'fileContent'], req.body, res)) {
                return;
            }

            const lecture = await lectureService.createLectureWithConcepts(
                courseId,
                name,
                fileContent
            );

            this.success(res, lecture, 'Lecture created successfully', 201);
        } catch (error) {
            this.error(res, error, 'createLecture');
        }
    }

    async deleteLecture(req, res) {
        try {
            const { id } = req.params;
            await lectureService.deleteLecture(id);
            this.success(res, null, 'Lecture deleted successfully');
        } catch (error) {
            this.error(res, error, 'deleteLecture', 404);
        }
    }
}

module.exports = LectureController;
```

---

## Services Layer

### Service with External API Integration

```javascript
// services/lectureService.js
const queries = require('../db/queries');
const anthropicService = require('./anthropicService');

class LectureService {
    async getLecturesByCourseId(courseId) {
        return queries.getLecturesByCourseId(courseId);
    }

    async createLectureWithConcepts(courseId, name, fileContent) {
        // 1. Create lecture in database
        const lecture = queries.createLecture(courseId, name, fileContent);

        // 2. Generate concepts using Claude API
        try {
            const concepts = await anthropicService.generateConcepts(fileContent);

            // 3. Save concepts to database
            for (const concept of concepts) {
                queries.createConcept(
                    lecture.id,
                    concept.concept_name,
                    concept.concept_description
                );
            }

            // 4. Fetch and return lecture with concepts
            return {
                ...lecture,
                concepts: queries.getConceptsByLectureId(lecture.id)
            };
        } catch (error) {
            console.error('[LectureService] Failed to generate concepts:', error);
            // Return lecture without concepts if API fails
            return lecture;
        }
    }

    async deleteLecture(lectureId) {
        const lecture = queries.getLectureById(lectureId);
        if (!lecture) {
            throw new Error('Lecture not found');
        }
        return queries.deleteLecture(lectureId);
    }
}

module.exports = new LectureService();
```

---

## Error Handling

### Async Error Wrapper

```javascript
// utils/asyncHandler.js
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

module.exports = asyncHandler;
```

### Global Error Handler Middleware

```javascript
// middleware/errorHandler.js
const errorHandler = (err, req, res, next) => {
    console.error('[Error Handler]', err);

    const statusCode = err.statusCode || 500;
    const message = err.message || 'Internal Server Error';

    res.status(statusCode).json({
        success: false,
        error: {
            message,
            code: statusCode,
            ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
        }
    });
};

module.exports = errorHandler;
```

### Usage in App

```javascript
// app.js
const express = require('express');
const errorHandler = require('./middleware/errorHandler');

const app = express();

// ... routes ...

// Error handler (MUST be last)
app.use(errorHandler);

module.exports = app;
```

---

## API Integration

See `api-integration-guide.md` for detailed Anthropic and Whisper integration.

Quick example:

```javascript
// services/anthropicService.js
const Anthropic = require('@anthropic-ai/sdk');

const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
});

class AnthropicService {
    async generateConcepts(lectureText) {
        const message = await client.messages.create({
            model: 'claude-sonnet-4-20250514',
            max_tokens: 2000,
            messages: [{
                role: 'user',
                content: `Break down the following lecture notes into 5-15 bite-sized, distinct concepts that a student should understand. Return as JSON array with 'concept_name' and 'concept_description' fields.\n\nLecture notes:\n${lectureText}`
            }]
        });

        const response = message.content[0].text;
        return JSON.parse(response);
    }
}

module.exports = new AnthropicService();
```

---

## File Upload

### Multer Configuration

```javascript
// middleware/upload.js
const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
        cb(null, `${Date.now()}-${file.originalname}`);
    }
});

const fileFilter = (req, file, cb) => {
    if (path.extname(file.originalname) === '.txt') {
        cb(null, true);
    } else {
        cb(new Error('Only .txt files are allowed'), false);
    }
};

const upload = multer({
    storage,
    fileFilter,
    limits: { fileSize: 5 * 1024 * 1024 } // 5MB
});

module.exports = upload;
```

### Usage in Routes

```javascript
// routes/lectureRoutes.js
const upload = require('../middleware/upload');

router.post('/',
    upload.single('file'),
    (req, res) => controller.createLecture(req, res)
);
```

---

## Complete Examples

### Full Route File

```javascript
// routes/courseRoutes.js
const express = require('express');
const CourseController = require('../controllers/CourseController');

const router = express.Router();
const controller = new CourseController();

router.get('/', (req, res) => controller.getCourses(req, res));
router.post('/', (req, res) => controller.createCourse(req, res));
router.delete('/:id', (req, res) => controller.deleteCourse(req, res));

module.exports = router;
```

### Full Controller

```javascript
// controllers/CourseController.js
const BaseController = require('./BaseController');
const courseService = require('../services/courseService');

class CourseController extends BaseController {
    async getCourses(req, res) {
        try {
            const courses = await courseService.getAllCourses();
            this.success(res, courses);
        } catch (error) {
            this.error(res, error, 'getCourses');
        }
    }

    async createCourse(req, res) {
        try {
            const { name } = req.body;

            if (!this.validateRequired(['name'], req.body, res)) {
                return;
            }

            const course = await courseService.createCourse(name);
            this.success(res, course, 'Course created', 201);
        } catch (error) {
            this.error(res, error, 'createCourse');
        }
    }

    async deleteCourse(req, res) {
        try {
            await courseService.deleteCourse(req.params.id);
            this.success(res, null, 'Course deleted');
        } catch (error) {
            this.error(res, error, 'deleteCourse', 404);
        }
    }
}

module.exports = CourseController;
```

### Express App Setup

```javascript
// app.js
const express = require('express');
const cors = require('cors');
const errorHandler = require('./middleware/errorHandler');

// Import routes
const courseRoutes = require('./routes/courseRoutes');
const lectureRoutes = require('./routes/lectureRoutes');
const conceptRoutes = require('./routes/conceptRoutes');

const app = express();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/courses', courseRoutes);
app.use('/api/lectures', lectureRoutes);
app.use('/api/concepts', conceptRoutes);

// Health check
app.get('/health', (req, res) => {
    res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Error handler (must be last)
app.use(errorHandler);

module.exports = app;
```

### Server Entry Point

```javascript
// server.js
require('dotenv').config();
const app = require('./app');

const PORT = process.env.PORT || 3001;

app.listen(PORT, () => {
    console.log(`[Server] Running on http://localhost:${PORT}`);
    console.log(`[Server] Environment: ${process.env.NODE_ENV || 'development'}`);
});
```

---

## Best Practices Summary

✅ **DO:**
- Use layered architecture (Routes → Controllers → Services → DB)
- Extend BaseController for all controllers
- Keep routes clean (just routing)
- Put business logic in services
- Use prepared statements for SQL queries
- Handle errors at every layer
- Validate inputs
- Log errors with context

❌ **DON'T:**
- Put business logic in routes
- Access database directly from controllers
- Forget error handling
- Expose sensitive data in errors
- Use string concatenation for SQL queries
- Skip input validation

---

## Next Steps

1. Set up your Express server using the patterns above
2. Create database schema and query functions
3. Build controllers for each resource
4. Implement services with business logic
5. Integrate Anthropic and Whisper APIs (see `api-integration-guide.md`)
6. Add error handling throughout
7. Test end-to-end

Good luck with your hackathon! 🚀
