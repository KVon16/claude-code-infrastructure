# Frontend Development Guide - Super Feynman
## React 18+ with Tailwind CSS

Adapted from production MUI patterns for rapid Tailwind CSS development.

---

## Table of Contents

- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Component Patterns](#component-patterns)
- [Styling with Tailwind](#styling-with-tailwind)
- [State Management](#state-management)
- [Data Fetching](#data-fetching)
- [Routing](#routing)
- [Loading & Error States](#loading--error-states)
- [Forms & Validation](#forms--validation)
- [Audio Recording](#audio-recording)
- [Complete Examples](#complete-examples)

---

## Quick Start

### New Component Checklist

- [ ] Use functional components with hooks
- [ ] Props with JSDoc comments
- [ ] Tailwind classes for styling
- [ ] `useState` for local state
- [ ] `useEffect` for side effects (with cleanup!)
- [ ] Loading states (no early returns)
- [ ] Error handling
- [ ] Memoize expensive computations with `useMemo`
- [ ] Memoize callbacks with `useCallback`

### Project Setup

```bash
# Create React app with Vite
npm create vite@latest super-feynman-frontend -- --template react

# Install dependencies
cd super-feynman-frontend
npm install

# Install Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Install additional dependencies
npm install react-router-dom axios
```

### Tailwind Configuration

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: '#3B82F6', // Blue
          dark: '#2563EB',
          light: '#60A5FA',
        },
        success: {
          DEFAULT: '#10B981', // Green
          dark: '#059669',
        },
        warning: {
          DEFAULT: '#F59E0B', // Yellow
        },
        background: {
          DEFAULT: '#F9FAFB',
        }
      },
    },
  },
  plugins: [],
}
```

---

## Project Structure

```
src/
├── components/           # Reusable components
│   ├── LoadingSpinner.jsx
│   ├── Modal.jsx
│   ├── Button.jsx
│   └── Header.jsx
├── pages/               # Page components (screens)
│   ├── Home.jsx
│   ├── CourseDetail.jsx
│   ├── LectureDetail.jsx
│   └── ReviewSession.jsx
├── services/            # API integration
│   └── api.js
├── hooks/               # Custom hooks
│   ├── useCourses.js
│   ├── useLectures.js
│   └── useAudioRecorder.js
├── utils/               # Utility functions
│   └── helpers.js
├── App.jsx
├── main.jsx
└── index.css
```

---

## Component Patterns

### Functional Component Template

```javascript
import React, { useState, useEffect, useCallback } from 'react';

/**
 * ComponentName - Brief description
 *
 * @param {Object} props
 * @param {string} props.title - Component title
 * @param {Function} props.onAction - Callback when action occurs
 */
const ComponentName = ({ title, onAction }) => {
    // 1. State declarations
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    // 2. Effects
    useEffect(() => {
        // Fetch data or setup
        fetchData();

        // Cleanup function
        return () => {
            // Clean up subscriptions, timers, etc.
        };
    }, []); // Dependencies

    // 3. Event handlers (memoized)
    const handleAction = useCallback(() => {
        onAction?.();
    }, [onAction]);

    // 4. Helper functions
    const fetchData = async () => {
        try {
            setLoading(true);
            // API call
            setData(result);
        } catch (err) {
            setError(err.message);
        } finally {
            setLoading(false);
        }
    };

    // 5. Render logic
    if (error) {
        return (
            <div className="text-red-500 p-4">
                Error: {error}
            </div>
        );
    }

    return (
        <div className="container mx-auto p-4">
            <h1 className="text-2xl font-bold mb-4">{title}</h1>
            {loading ? (
                <div className="flex justify-center">
                    <LoadingSpinner />
                </div>
            ) : (
                <div className="bg-white rounded-lg shadow p-6">
                    {/* Your content */}
                </div>
            )}
        </div>
    );
};

export default ComponentName;
```

### Key Principles

1. **No Early Returns for Loading States** (unless it's an error)
   - Prevents layout shift
   - Better UX

2. **Cleanup in useEffect**
   ```javascript
   useEffect(() => {
       const timer = setTimeout(() => {}, 1000);
       return () => clearTimeout(timer); // Always cleanup!
   }, []);
   ```

3. **Memoization**
   - `useCallback` for functions passed as props
   - `useMemo` for expensive computations

---

## Styling with Tailwind

### Common Patterns

#### Container Layout
```javascript
<div className="container mx-auto px-4 py-8 max-w-6xl">
    {/* Content */}
</div>
```

#### Card Component
```javascript
<div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
    <h2 className="text-xl font-semibold mb-2">Card Title</h2>
    <p className="text-gray-600">Card content</p>
</div>
```

#### Button Styles
```javascript
// Primary Button
<button className="bg-primary text-white px-6 py-2 rounded-full font-semibold hover:bg-primary-dark transition-colors">
    Click Me
</button>

// Secondary Button
<button className="border-2 border-gray-300 text-gray-700 px-6 py-2 rounded-full font-semibold hover:bg-gray-100 transition-colors">
    Cancel
</button>

// Danger Button
<button className="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600 transition-colors">
    Delete
</button>
```

#### Modal Overlay
```javascript
<div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div className="bg-white rounded-lg shadow-xl p-6 max-w-md w-full mx-4">
        {/* Modal content */}
    </div>
</div>
```

#### Grid Layout
```javascript
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {items.map(item => (
        <div key={item.id} className="...">
            {item.name}
        </div>
    ))}
</div>
```

#### Status Badges
```javascript
const statusColors = {
    'Not Started': 'bg-gray-200 text-gray-700',
    'Reviewing': 'bg-yellow-200 text-yellow-800',
    'Understood': 'bg-green-200 text-green-800',
    'Mastered': 'bg-green-500 text-white'
};

<span className={`px-3 py-1 rounded-full text-sm font-medium ${statusColors[status]}`}>
    {status}
</span>
```

#### Responsive Design
```javascript
// Mobile: stack vertically, Desktop: side-by-side
<div className="flex flex-col md:flex-row gap-4">
    <div className="flex-1">Left</div>
    <div className="flex-1">Right</div>
</div>

// Hide on mobile, show on desktop
<div className="hidden md:block">Desktop only</div>

// Show on mobile, hide on desktop
<div className="block md:hidden">Mobile only</div>
```

---

## State Management

### Local State (useState)

```javascript
const [courses, setCourses] = useState([]);
const [selectedCourse, setSelectedCourse] = useState(null);
const [isModalOpen, setIsModalOpen] = useState(false);
```

### Lifting State Up

```javascript
// Parent component manages state
const Home = () => {
    const [courses, setCourses] = useState([]);
    const [showAddModal, setShowAddModal] = useState(false);

    return (
        <>
            <CourseList courses={courses} onAdd={() => setShowAddModal(true)} />
            {showAddModal && (
                <AddCourseModal
                    onClose={() => setShowAddModal(false)}
                    onSubmit={(course) => {
                        setCourses([...courses, course]);
                        setShowAddModal(false);
                    }}
                />
            )}
        </>
    );
};
```

### Context for Global State (Optional)

```javascript
// contexts/AppContext.js
import React, { createContext, useState, useContext } from 'react';

const AppContext = createContext();

export const AppProvider = ({ children }) => {
    const [user, setUser] = useState(null);

    return (
        <AppContext.Provider value={{ user, setUser }}>
            {children}
        </AppContext.Provider>
    );
};

export const useApp = () => useContext(AppContext);
```

---

## Data Fetching

### API Service Layer

```javascript
// services/api.js
import axios from 'axios';

const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:3001/api';

const apiClient = axios.create({
    baseURL: API_BASE_URL,
    headers: {
        'Content-Type': 'application/json',
    },
});

// Courses
export const courseApi = {
    getAll: () => apiClient.get('/courses'),
    create: (name) => apiClient.post('/courses', { name }),
    delete: (id) => apiClient.delete(`/courses/${id}`),
};

// Lectures
export const lectureApi = {
    getByCourseId: (courseId) => apiClient.get(`/courses/${courseId}/lectures`),
    create: (courseId, name, fileContent) =>
        apiClient.post('/lectures', { courseId, name, fileContent }),
    delete: (id) => apiClient.delete(`/lectures/${id}`),
};

// Concepts
export const conceptApi = {
    getByLectureId: (lectureId) => apiClient.get(`/lectures/${lectureId}/concepts`),
    updateProgress: (conceptId, status) =>
        apiClient.patch(`/concepts/${conceptId}/progress`, { status }),
};

export default apiClient;
```

### Custom Hook for Data Fetching

```javascript
// hooks/useCourses.js
import { useState, useEffect } from 'react';
import { courseApi } from '../services/api';

export const useCourses = () => {
    const [courses, setCourses] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    const fetchCourses = async () => {
        try {
            setLoading(true);
            const response = await courseApi.getAll();
            setCourses(response.data.data);
        } catch (err) {
            setError(err.message);
        } finally {
            setLoading(false);
        }
    };

    useEffect(() => {
        fetchCourses();
    }, []);

    const addCourse = async (name) => {
        try {
            const response = await courseApi.create(name);
            setCourses([...courses, response.data.data]);
            return response.data.data;
        } catch (err) {
            throw new Error(err.response?.data?.error?.message || 'Failed to create course');
        }
    };

    const deleteCourse = async (id) => {
        try {
            await courseApi.delete(id);
            setCourses(courses.filter(c => c.id !== id));
        } catch (err) {
            throw new Error(err.response?.data?.error?.message || 'Failed to delete course');
        }
    };

    return { courses, loading, error, addCourse, deleteCourse, refetch: fetchCourses };
};
```

---

## Routing

### React Router Setup

```javascript
// App.jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import CourseDetail from './pages/CourseDetail';
import LectureDetail from './pages/LectureDetail';
import ReviewSession from './pages/ReviewSession';

function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/course/:courseId" element={<CourseDetail />} />
                <Route path="/lecture/:lectureId" element={<LectureDetail />} />
                <Route path="/review/:conceptId" element={<ReviewSession />} />
            </Routes>
        </BrowserRouter>
    );
}

export default App;
```

### Navigation

```javascript
import { useNavigate, useParams } from 'react-router-dom';

const Component = () => {
    const navigate = useNavigate();
    const { courseId } = useParams();

    const goToCourse = (id) => {
        navigate(`/course/${id}`);
    };

    const goBack = () => {
        navigate(-1); // Go back one page
    };

    return (
        <div>
            <button onClick={goBack} className="...">← Back</button>
        </div>
    );
};
```

---

## Loading & Error States

### CRITICAL RULE: No Early Returns

```javascript
// ❌ BAD - Causes layout shift
const Component = () => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);

    if (loading) return <LoadingSpinner />;

    return <div>{data}</div>;
};

// ✅ GOOD - Consistent layout
const Component = () => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);

    return (
        <div className="min-h-screen">
            <Header /> {/* Always visible */}
            <div className="container mx-auto p-4">
                {loading ? (
                    <div className="flex justify-center items-center h-64">
                        <LoadingSpinner />
                    </div>
                ) : (
                    <div>{data}</div>
                )}
            </div>
        </div>
    );
};
```

### Loading Spinner Component

```javascript
// components/LoadingSpinner.jsx
const LoadingSpinner = ({ size = 'md', text = '' }) => {
    const sizes = {
        sm: 'w-8 h-8',
        md: 'w-12 h-12',
        lg: 'w-16 h-16',
    };

    return (
        <div className="flex flex-col items-center justify-center">
            <div className={`${sizes[size]} border-4 border-gray-200 border-t-primary rounded-full animate-spin`} />
            {text && <p className="mt-2 text-gray-600">{text}</p>}
        </div>
    );
};

export default LoadingSpinner;
```

### Error Display

```javascript
const ErrorMessage = ({ message, onRetry }) => (
    <div className="bg-red-50 border border-red-200 rounded-lg p-4 my-4">
        <div className="flex items-start">
            <div className="flex-shrink-0">
                <svg className="h-5 w-5 text-red-400" viewBox="0 0 20 20" fill="currentColor">
                    <path fillRule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clipRule="evenodd" />
                </svg>
            </div>
            <div className="ml-3 flex-1">
                <h3 className="text-sm font-medium text-red-800">Error</h3>
                <p className="mt-1 text-sm text-red-700">{message}</p>
                {onRetry && (
                    <button onClick={onRetry} className="mt-2 text-sm font-medium text-red-600 hover:text-red-500">
                        Try again →
                    </button>
                )}
            </div>
        </div>
    </div>
);
```

---

## Forms & Validation

### Controlled Inputs

```javascript
const AddCourseForm = ({ onSubmit, onCancel }) => {
    const [courseName, setCourseName] = useState('');
    const [error, setError] = useState('');

    const handleSubmit = (e) => {
        e.preventDefault();

        // Validation
        if (!courseName.trim()) {
            setError('Course name is required');
            return;
        }

        if (courseName.length < 3) {
            setError('Course name must be at least 3 characters');
            return;
        }

        onSubmit(courseName);
    };

    return (
        <form onSubmit={handleSubmit} className="space-y-4">
            <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">
                    Course Name
                </label>
                <input
                    type="text"
                    value={courseName}
                    onChange={(e) => {
                        setCourseName(e.target.value);
                        setError(''); // Clear error on change
                    }}
                    className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-primary focus:border-transparent"
                    placeholder="Enter course name"
                />
                {error && (
                    <p className="mt-1 text-sm text-red-600">{error}</p>
                )}
            </div>
            <div className="flex justify-end space-x-2">
                <button
                    type="button"
                    onClick={onCancel}
                    className="px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-100"
                >
                    Cancel
                </button>
                <button
                    type="submit"
                    className="px-4 py-2 bg-primary text-white rounded-lg hover:bg-primary-dark"
                >
                    Create Course
                </button>
            </div>
        </form>
    );
};
```

### File Upload

```javascript
const FileUploadInput = ({ onFileSelect }) => {
    const [fileName, setFileName] = useState('');

    const handleFileChange = (e) => {
        const file = e.target.files[0];
        if (file) {
            if (file.name.endsWith('.txt')) {
                setFileName(file.name);
                const reader = new FileReader();
                reader.onload = (event) => {
                    onFileSelect(event.target.result);
                };
                reader.readAsText(file);
            } else {
                alert('Only .txt files are allowed');
                e.target.value = '';
            }
        }
    };

    return (
        <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
                Upload Notes
            </label>
            <input
                type="file"
                accept=".txt"
                onChange={handleFileChange}
                className="hidden"
                id="file-upload"
            />
            <label
                htmlFor="file-upload"
                className="cursor-pointer inline-flex items-center px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50"
            >
                <svg className="w-5 h-5 mr-2" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M8 4a3 3 0 00-3 3v4a5 5 0 0010 0V7a1 1 0 112 0v4a7 7 0 11-14 0V7a5 5 0 0110 0v4a3 3 0 11-6 0V7a1 1 0 012 0v4a1 1 0 102 0V7a3 3 0 00-3-3z" />
                </svg>
                {fileName || 'Choose File'}
            </label>
        </div>
    );
};
```

---

## Audio Recording

### useAudioRecorder Hook

```javascript
// hooks/useAudioRecorder.js
import { useState, useRef } from 'react';

export const useAudioRecorder = () => {
    const [isRecording, setIsRecording] = useState(false);
    const [audioBlob, setAudioBlob] = useState(null);
    const mediaRecorderRef = useRef(null);
    const chunksRef = useRef([]);

    const startRecording = async () => {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
            const mediaRecorder = new MediaRecorder(stream);
            mediaRecorderRef.current = mediaRecorder;
            chunksRef.current = [];

            mediaRecorder.ondataavailable = (e) => {
                chunksRef.current.push(e.data);
            };

            mediaRecorder.onstop = () => {
                const blob = new Blob(chunksRef.current, { type: 'audio/webm' });
                setAudioBlob(blob);
                stream.getTracks().forEach(track => track.stop());
            };

            mediaRecorder.start();
            setIsRecording(true);
        } catch (error) {
            console.error('Failed to start recording:', error);
            alert('Microphone access denied');
        }
    };

    const stopRecording = () => {
        if (mediaRecorderRef.current && isRecording) {
            mediaRecorderRef.current.stop();
            setIsRecording(false);
        }
    };

    const clearRecording = () => {
        setAudioBlob(null);
        chunksRef.current = [];
    };

    return {
        isRecording,
        audioBlob,
        startRecording,
        stopRecording,
        clearRecording,
    };
};
```

### Microphone Button Component

```javascript
const MicrophoneButton = ({ onTranscription }) => {
    const { isRecording, audioBlob, startRecording, stopRecording, clearRecording } = useAudioRecorder();
    const [isTranscribing, setIsTranscribing] = useState(false);

    useEffect(() => {
        if (audioBlob) {
            transcribeAudio(audioBlob);
        }
    }, [audioBlob]);

    const transcribeAudio = async (blob) => {
        setIsTranscribing(true);
        try {
            const formData = new FormData();
            formData.append('file', blob, 'recording.webm');

            const response = await axios.post('/api/transcribe', formData);
            onTranscription(response.data.text);
            clearRecording();
        } catch (error) {
            console.error('Transcription failed:', error);
            alert('Failed to transcribe audio');
        } finally {
            setIsTranscribing(false);
        }
    };

    return (
        <button
            onClick={isRecording ? stopRecording : startRecording}
            disabled={isTranscribing}
            className={`
                w-12 h-12 rounded-full flex items-center justify-center transition-all
                ${isRecording ? 'bg-red-500 animate-pulse' : 'bg-primary hover:bg-primary-dark'}
                ${isTranscribing ? 'opacity-50 cursor-not-allowed' : ''}
            `}
        >
            <svg className="w-6 h-6 text-white" fill="currentColor" viewBox="0 0 20 20">
                <path fillRule="evenodd" d="M7 4a3 3 0 016 0v4a3 3 0 11-6 0V4zm4 10.93A7.001 7.001 0 0017 8a1 1 0 10-2 0A5 5 0 015 8a1 1 0 00-2 0 7.001 7.001 0 006 6.93V17H6a1 1 0 100 2h8a1 1 0 100-2h-3v-2.07z" clipRule="evenodd" />
            </svg>
        </button>
    );
};
```

---

## Complete Examples

### Home Screen (Course List)

```javascript
// pages/Home.jsx
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useCourses } from '../hooks/useCourses';
import LoadingSpinner from '../components/LoadingSpinner';
import Modal from '../components/Modal';

const Home = () => {
    const navigate = useNavigate();
    const { courses, loading, error, addCourse, deleteCourse } = useCourses();
    const [showModal, setShowModal] = useState(false);
    const [courseName, setCourseName] = useState('');

    const handleCreateCourse = async () => {
        if (!courseName.trim()) return;

        try {
            const newCourse = await addCourse(courseName);
            setShowModal(false);
            setCourseName('');
            navigate(`/course/${newCourse.id}`);
        } catch (err) {
            alert(err.message);
        }
    };

    return (
        <div className="min-h-screen bg-background">
            {/* Header */}
            <header className="bg-white shadow-sm">
                <div className="container mx-auto px-4 py-6">
                    <h1 className="text-3xl font-bold text-center text-primary">
                        Super Feynman
                    </h1>
                </div>
            </header>

            {/* Main Content */}
            <main className="container mx-auto px-4 py-8">
                {loading ? (
                    <div className="flex justify-center py-20">
                        <LoadingSpinner size="lg" />
                    </div>
                ) : error ? (
                    <div className="text-center py-20">
                        <p className="text-red-500">{error}</p>
                    </div>
                ) : courses.length === 0 ? (
                    <div className="text-center py-20">
                        <p className="text-gray-600 text-lg mb-6">
                            Welcome, add your first course to get started
                        </p>
                        <button
                            onClick={() => setShowModal(true)}
                            className="bg-primary text-white px-8 py-3 rounded-full font-semibold hover:bg-primary-dark transition-colors"
                        >
                            Add a Course
                        </button>
                    </div>
                ) : (
                    <>
                        <div className="flex justify-between items-center mb-6">
                            <h2 className="text-2xl font-semibold">My Courses</h2>
                            <button
                                onClick={() => setShowModal(true)}
                                className="bg-primary text-white px-6 py-2 rounded-full hover:bg-primary-dark"
                            >
                                + Add Course
                            </button>
                        </div>

                        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                            {courses.map(course => (
                                <div
                                    key={course.id}
                                    onClick={() => navigate(`/course/${course.id}`)}
                                    className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow cursor-pointer"
                                >
                                    <div className="flex justify-between items-start">
                                        <h3 className="text-xl font-semibold">{course.name}</h3>
                                        <button
                                            onClick={(e) => {
                                                e.stopPropagation();
                                                if (confirm('Delete this course?')) {
                                                    deleteCourse(course.id);
                                                }
                                            }}
                                            className="text-red-500 hover:text-red-700"
                                        >
                                            ×
                                        </button>
                                    </div>
                                </div>
                            ))}
                        </div>
                    </>
                )}
            </main>

            {/* Add Course Modal */}
            {showModal && (
                <Modal onClose={() => setShowModal(false)}>
                    <h2 className="text-xl font-semibold mb-4">Add New Course</h2>
                    <input
                        type="text"
                        value={courseName}
                        onChange={(e) => setCourseName(e.target.value)}
                        placeholder="Course Name"
                        className="w-full px-4 py-2 border border-gray-300 rounded-lg mb-4 focus:ring-2 focus:ring-primary focus:border-transparent"
                        onKeyPress={(e) => e.key === 'Enter' && handleCreateCourse()}
                    />
                    <div className="flex justify-end space-x-2">
                        <button
                            onClick={() => setShowModal(false)}
                            className="px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-100"
                        >
                            Cancel
                        </button>
                        <button
                            onClick={handleCreateCourse}
                            className="px-4 py-2 bg-primary text-white rounded-lg hover:bg-primary-dark"
                        >
                            Create Course
                        </button>
                    </div>
                </Modal>
            )}
        </div>
    );
};

export default Home;
```

### Modal Component

```javascript
// components/Modal.jsx
import React, { useEffect } from 'react';

const Modal = ({ children, onClose }) => {
    useEffect(() => {
        // Prevent body scroll when modal is open
        document.body.style.overflow = 'hidden';
        return () => {
            document.body.style.overflow = 'unset';
        };
    }, []);

    return (
        <div
            className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
            onClick={onClose}
        >
            <div
                className="bg-white rounded-lg shadow-xl p-6 max-w-md w-full mx-4"
                onClick={(e) => e.stopPropagation()}
            >
                {children}
            </div>
        </div>
    );
};

export default Modal;
```

---

## Best Practices Summary

✅ **DO:**
- Use functional components with hooks
- Memoize callbacks with `useCallback`
- Cleanup effects (timers, subscriptions)
- Use Tailwind utility classes
- Keep components focused and small
- Extract reusable components
- Handle loading and error states
- Validate user inputs

❌ **DON'T:**
- Use early returns for loading states (causes layout shift)
- Forget to cleanup useEffect
- Put too much logic in components
- Inline complex functions
- Forget key props in lists
- Mix styling approaches (stick to Tailwind)

---

## Next Steps

1. Set up React project with Vite
2. Configure Tailwind CSS
3. Create component structure
4. Build page components (Home, CourseDetail, etc.)
5. Implement API integration
6. Add audio recording functionality
7. Style with Tailwind CSS
8. Test user flows

Good luck with your hackathon! 🎨
