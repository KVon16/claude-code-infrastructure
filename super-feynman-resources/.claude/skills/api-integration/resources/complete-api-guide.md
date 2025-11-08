# API Integration Guide - Super Feynman
## Anthropic Claude API + OpenAI Whisper API

Complete guide for integrating AI services into Super Feynman.

---

## Table of Contents

- [Environment Setup](#environment-setup)
- [Anthropic Claude API](#anthropic-claude-api)
  - [Concept Generation](#concept-generation)
  - [Review Conversation](#review-conversation)
  - [Feedback Analysis](#feedback-analysis)
- [OpenAI Whisper API](#openai-whisper-api)
  - [Speech-to-Text Transcription](#speech-to-text-transcription)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Testing](#testing)

---

## Environment Setup

### Install Dependencies

```bash
# Backend
npm install @anthropic-ai/sdk openai dotenv
```

### Environment Variables

```bash
# .env file
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here
NODE_ENV=development
```

### Load Environment Variables

```javascript
// At the top of your app.js or server.js
require('dotenv').config();
```

---

## Anthropic Claude API

### Latest Model Information

As of January 2025, the latest Anthropic model is:
- **Model ID:** `claude-sonnet-4-20250514`
- **Context Window:** 200K tokens
- **Features:** Advanced reasoning, code generation, long context

Check [Anthropic's documentation](https://docs.anthropic.com/en/api/getting-started) for the latest models.

### Service Setup

```javascript
// services/anthropicService.js
const Anthropic = require('@anthropic-ai/sdk');

const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
});

// Model configuration
const MODEL = 'claude-sonnet-4-20250514';
const MAX_TOKENS = 2000;

class AnthropicService {
    // Will implement methods below
}

module.exports = new AnthropicService();
```

---

### Concept Generation

Generate bite-sized concepts from lecture notes.

```javascript
// services/anthropicService.js

async generateConcepts(lectureText) {
    try {
        const message = await client.messages.create({
            model: MODEL,
            max_tokens: MAX_TOKENS,
            messages: [{
                role: 'user',
                content: `You are an educational assistant helping students learn using the Feynman Technique.

Analyze the following lecture notes and break them down into 5-15 bite-sized, distinct concepts that a student should understand. Each concept should be:
- A specific, focused topic (not too broad)
- Something that can be explained in 2-3 minutes
- Testable through conversation

Return ONLY a valid JSON array with this exact structure:
[
  {
    "concept_name": "Brief name of the concept",
    "concept_description": "2-3 sentence description of what the student should understand"
  }
]

Lecture notes:
${lectureText}

Remember: Return ONLY the JSON array, no additional text.`
            }]
        });

        const responseText = message.content[0].text.trim();

        // Parse JSON response
        const concepts = JSON.parse(responseText);

        // Validate structure
        if (!Array.isArray(concepts)) {
            throw new Error('Response is not an array');
        }

        // Ensure each concept has required fields
        concepts.forEach(concept => {
            if (!concept.concept_name || !concept.concept_description) {
                throw new Error('Invalid concept structure');
            }
        });

        console.log(`[Anthropic] Generated ${concepts.length} concepts`);
        return concepts;

    } catch (error) {
        console.error('[Anthropic] Concept generation failed:', error);

        // Return fallback if JSON parsing fails
        if (error instanceof SyntaxError) {
            return [{
                concept_name: 'General Overview',
                concept_description: 'Review the main ideas from the lecture notes.'
            }];
        }

        throw error;
    }
}
```

**Usage:**

```javascript
// In lectureService.js
const anthropicService = require('./anthropicService');

async createLectureWithConcepts(courseId, name, fileContent) {
    const lecture = queries.createLecture(courseId, name, fileContent);

    try {
        const concepts = await anthropicService.generateConcepts(fileContent);

        for (const concept of concepts) {
            queries.createConcept(
                lecture.id,
                concept.concept_name,
                concept.concept_description
            );
        }
    } catch (error) {
        console.error('Failed to generate concepts:', error);
        // Lecture exists, but without AI-generated concepts
    }

    return lecture;
}
```

---

### Review Conversation

Interactive conversation to test student understanding using the Feynman Technique.

```javascript
// services/anthropicService.js

async startReviewConversation(conceptName, conceptDescription, audienceLevel) {
    // Map audience level to system prompt
    const audiencePrompts = {
        'classmate': 'a fellow college student in your class',
        'middle-schooler': 'a curious 13-year-old middle school student',
        'kid': 'a bright 8-year-old child'
    };

    const audience = audiencePrompts[audienceLevel] || audiencePrompts['classmate'];

    const systemPrompt = `You are a Socratic tutor helping a student learn using the Feynman Technique. The student will explain the concept: "${conceptName}".

Your role:
- Ask probing questions to test their understanding
- Identify gaps in their knowledge
- Encourage them to explain in simple terms (as if explaining to ${audience})
- NEVER directly teach or explain the concept yourself
- Ask follow-up questions about unclear explanations
- Challenge them with "why" and "how" questions
- Keep responses concise (2-3 sentences max)
- Be encouraging but honest about gaps

The student should be explaining to ${audience}, so watch for:
- Use of technical jargon without explanation
- Skipping important steps in reasoning
- Vague or circular explanations
- Missing key concepts from the description: ${conceptDescription}

Start by asking them to explain the concept in their own words.`;

    try {
        const message = await client.messages.create({
            model: MODEL,
            max_tokens: 300, // Shorter for quick responses
            system: systemPrompt,
            messages: [{
                role: 'user',
                content: 'I\'m ready to explain this concept.'
            }]
        });

        return {
            message: message.content[0].text,
            conversationHistory: [
                { role: 'user', content: 'I\'m ready to explain this concept.' },
                { role: 'assistant', content: message.content[0].text }
            ]
        };

    } catch (error) {
        console.error('[Anthropic] Failed to start conversation:', error);
        throw error;
    }
}

async continueReviewConversation(conversationHistory, userMessage) {
    try {
        // Add user's message to history
        const updatedHistory = [
            ...conversationHistory,
            { role: 'user', content: userMessage }
        ];

        const message = await client.messages.create({
            model: MODEL,
            max_tokens: 300,
            messages: updatedHistory
        });

        const assistantMessage = message.content[0].text;

        return {
            message: assistantMessage,
            conversationHistory: [
                ...updatedHistory,
                { role: 'assistant', content: assistantMessage }
            ]
        };

    } catch (error) {
        console.error('[Anthropic] Failed to continue conversation:', error);
        throw error;
    }
}
```

**Usage in Controller:**

```javascript
// controllers/ConceptController.js

async startReview(req, res) {
    try {
        const { conceptId, audienceLevel } = req.body;

        const concept = queries.getConceptById(conceptId);
        if (!concept) {
            return this.error(res, new Error('Concept not found'), 'startReview', 404);
        }

        const result = await anthropicService.startReviewConversation(
            concept.concept_name,
            concept.concept_description,
            audienceLevel
        );

        this.success(res, result);
    } catch (error) {
        this.error(res, error, 'startReview');
    }
}

async continueReview(req, res) {
    try {
        const { conversationHistory, userMessage } = req.body;

        const result = await anthropicService.continueReviewConversation(
            conversationHistory,
            userMessage
        );

        this.success(res, result);
    } catch (error) {
        this.error(res, error, 'continueReview');
    }
}
```

---

### Feedback Analysis

Analyze the review session and provide feedback.

```javascript
// services/anthropicService.js

async generateFeedback(conversationHistory, conceptName, conceptDescription) {
    try {
        const conversationText = conversationHistory
            .map(msg => `${msg.role === 'user' ? 'Student' : 'Tutor'}: ${msg.content}`)
            .join('\n\n');

        const message = await client.messages.create({
            model: MODEL,
            max_tokens: 1500,
            messages: [{
                role: 'user',
                content: `You are analyzing a Feynman Technique review session for the concept: "${conceptName}".

Expected understanding: ${conceptDescription}

Review session transcript:
${conversationText}

Analyze the student's performance and provide structured feedback in JSON format:
{
  "overallQuality": "2-3 sentence summary of how well they explained the concept",
  "clearParts": ["Array of specific things they explained well"],
  "unclearParts": ["Array of areas that were unclear or incomplete"],
  "jargonUsed": ["Array of technical terms they used without explaining"],
  "struggledWith": ["Array of specific aspects they had difficulty explaining"],
  "progressLevel": "one of: Not Started, Reviewing, Understood, Mastered"
}

Progress level guidelines:
- "Not Started": Never attempted or completely off-track
- "Reviewing": Made an attempt but major gaps in understanding
- "Understood": Explained most of the concept correctly with minor gaps
- "Mastered": Explained clearly and completely with proper analogies

Return ONLY the JSON object, no additional text.`
            }]
        });

        const feedbackText = message.content[0].text.trim();
        const feedback = JSON.parse(feedbackText);

        console.log(`[Anthropic] Generated feedback for concept: ${conceptName}`);
        return feedback;

    } catch (error) {
        console.error('[Anthropic] Feedback generation failed:', error);

        // Fallback feedback
        return {
            overallQuality: 'Unable to analyze the session. Please try again.',
            clearParts: [],
            unclearParts: ['Session analysis failed'],
            jargonUsed: [],
            struggledWith: [],
            progressLevel: 'Reviewing'
        };
    }
}
```

**Usage:**

```javascript
// controllers/ConceptController.js

async endReview(req, res) {
    try {
        const { conceptId, conversationHistory, audienceLevel } = req.body;

        const concept = queries.getConceptById(conceptId);
        if (!concept) {
            return this.error(res, new Error('Concept not found'), 'endReview', 404);
        }

        // Generate feedback
        const feedback = await anthropicService.generateFeedback(
            conversationHistory,
            concept.concept_name,
            concept.concept_description
        );

        // Update concept progress
        const updatedConcept = queries.updateConceptProgress(
            conceptId,
            feedback.progressLevel
        );

        // Save review session
        queries.createReviewSession(
            conceptId,
            audienceLevel,
            conversationHistory,
            feedback
        );

        this.success(res, {
            feedback,
            concept: updatedConcept
        });

    } catch (error) {
        this.error(res, error, 'endReview');
    }
}
```

---

## OpenAI Whisper API

### Latest Model Information

As of January 2025:
- **Model:** `whisper-1`
- **Supported formats:** mp3, mp4, mpeg, mpga, m4a, wav, webm
- **File size limit:** 25 MB

Check [OpenAI's documentation](https://platform.openai.com/docs/guides/speech-to-text) for updates.

### Service Setup

```javascript
// services/whisperService.js
const OpenAI = require('openai');
const fs = require('fs');

const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
});

class WhisperService {
    async transcribeAudio(audioFilePath) {
        try {
            console.log(`[Whisper] Transcribing audio file: ${audioFilePath}`);

            const transcription = await openai.audio.transcriptions.create({
                file: fs.createReadStream(audioFilePath),
                model: 'whisper-1',
                language: 'en', // Optional: specify language for better accuracy
                response_format: 'text' // Can also use 'json', 'srt', 'vtt'
            });

            console.log(`[Whisper] Transcription successful`);
            return transcription;

        } catch (error) {
            console.error('[Whisper] Transcription failed:', error);
            throw error;
        }
    }
}

module.exports = new WhisperService();
```

### API Route with File Upload

```javascript
// routes/transcribeRoutes.js
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const whisperService = require('../services/whisperService');

const router = express.Router();

// Configure multer for audio uploads
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        const uploadDir = 'uploads/audio';
        if (!fs.existsSync(uploadDir)) {
            fs.mkdirSync(uploadDir, { recursive: true });
        }
        cb(null, uploadDir);
    },
    filename: (req, file, cb) => {
        cb(null, `${Date.now()}-${file.originalname}`);
    }
});

const upload = multer({
    storage,
    limits: { fileSize: 25 * 1024 * 1024 }, // 25MB limit
    fileFilter: (req, file, cb) => {
        const allowedTypes = ['audio/webm', 'audio/wav', 'audio/mp3', 'audio/mpeg'];
        if (allowedTypes.includes(file.mimetype)) {
            cb(null, true);
        } else {
            cb(new Error('Invalid file type. Only audio files are allowed.'));
        }
    }
});

router.post('/', upload.single('audio'), async (req, res) => {
    try {
        if (!req.file) {
            return res.status(400).json({
                success: false,
                error: { message: 'No audio file provided' }
            });
        }

        const transcription = await whisperService.transcribeAudio(req.file.path);

        // Clean up uploaded file after transcription
        fs.unlinkSync(req.file.path);

        res.json({
            success: true,
            data: { text: transcription }
        });

    } catch (error) {
        console.error('[Transcribe Route] Error:', error);

        // Clean up file if it exists
        if (req.file && fs.existsSync(req.file.path)) {
            fs.unlinkSync(req.file.path);
        }

        res.status(500).json({
            success: false,
            error: { message: error.message || 'Transcription failed' }
        });
    }
});

module.exports = router;
```

### Frontend Integration

```javascript
// In your React component
const transcribeAudio = async (audioBlob) => {
    setIsTranscribing(true);

    try {
        const formData = new FormData();
        formData.append('audio', audioBlob, 'recording.webm');

        const response = await axios.post('/api/transcribe', formData, {
            headers: {
                'Content-Type': 'multipart/form-data'
            }
        });

        const transcribedText = response.data.data.text;
        return transcribedText;

    } catch (error) {
        console.error('Transcription error:', error);
        throw new Error('Failed to transcribe audio');
    } finally {
        setIsTranscribing(false);
    }
};
```

---

## Error Handling

### API Error Types

```javascript
// utils/apiErrors.js

class AnthropicError extends Error {
    constructor(message, statusCode = 500) {
        super(message);
        this.name = 'AnthropicError';
        this.statusCode = statusCode;
    }
}

class WhisperError extends Error {
    constructor(message, statusCode = 500) {
        super(message);
        this.name = 'WhisperError';
        this.statusCode = statusCode;
    }
}

module.exports = { AnthropicError, WhisperError };
```

### Retry Logic

```javascript
// utils/retryHelper.js

async function retryWithBackoff(fn, maxRetries = 3) {
    for (let attempt = 0; attempt < maxRetries; attempt++) {
        try {
            return await fn();
        } catch (error) {
            // Don't retry on client errors (4xx)
            if (error.status >= 400 && error.status < 500) {
                throw error;
            }

            // Last attempt, throw error
            if (attempt === maxRetries - 1) {
                throw error;
            }

            // Exponential backoff: 1s, 2s, 4s
            const delay = Math.pow(2, attempt) * 1000;
            console.log(`[Retry] Attempt ${attempt + 1} failed, retrying in ${delay}ms...`);
            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}

module.exports = { retryWithBackoff };
```

### Usage with Retry

```javascript
const { retryWithBackoff } = require('../utils/retryHelper');

async generateConceptsWithRetry(lectureText) {
    return await retryWithBackoff(async () => {
        return await this.generateConcepts(lectureText);
    });
}
```

---

## Rate Limiting

### Simple Rate Limiter

```javascript
// middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');

// Limit API calls to prevent quota exhaustion
const apiLimiter = rateLimit({
    windowMs: 60 * 1000, // 1 minute
    max: 10, // Max 10 requests per minute
    message: {
        success: false,
        error: {
            message: 'Too many requests, please try again later.'
        }
    }
});

module.exports = apiLimiter;
```

### Apply to Routes

```javascript
// app.js
const apiLimiter = require('./middleware/rateLimiter');

app.use('/api/concepts/generate', apiLimiter);
app.use('/api/transcribe', apiLimiter);
```

---

## Testing

### Test Anthropic Integration

```javascript
// Test in Node.js REPL or create a test script
const anthropicService = require('./services/anthropicService');

async function testConceptGeneration() {
    const sampleText = `
    Photosynthesis is the process by which plants convert light energy into chemical energy.
    It occurs in chloroplasts and requires sunlight, water, and carbon dioxide.
    The process produces glucose and oxygen as outputs.
    `;

    try {
        const concepts = await anthropicService.generateConcepts(sampleText);
        console.log('Generated concepts:', concepts);
    } catch (error) {
        console.error('Test failed:', error);
    }
}

testConceptGeneration();
```

### Test Whisper Integration

```bash
# Upload a test audio file
curl -X POST http://localhost:3001/api/transcribe \
  -F "audio=@test-recording.webm"
```

---

## Cost Optimization Tips

### Anthropic API
1. Use `max_tokens` to limit response length
2. Cache system prompts when possible
3. Use streaming for long conversations (not critical for hackathon)
4. Monitor token usage in responses

### OpenAI Whisper
1. Compress audio files before sending (reduce quality if acceptable)
2. Limit recording duration in frontend
3. Consider using `response_format: 'text'` (smallest response)

### General
1. Implement client-side validation before API calls
2. Cache responses when appropriate
3. Use rate limiting to prevent abuse
4. Add request timeouts

---

## Environment Best Practices

```bash
# .env.example (commit this to repo)
ANTHROPIC_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here
NODE_ENV=development
PORT=3001

# .env (DO NOT commit this)
# Add your actual keys here
```

```javascript
// Validate environment variables on startup
// In server.js or app.js

const requiredEnvVars = ['ANTHROPIC_API_KEY', 'OPENAI_API_KEY'];

requiredEnvVars.forEach(varName => {
    if (!process.env[varName]) {
        console.error(`[Error] Missing required environment variable: ${varName}`);
        process.exit(1);
    }
});

console.log('[Server] Environment variables validated ✓');
```

---

## Complete Integration Example

### Backend Route

```javascript
// routes/conceptRoutes.js
const express = require('express');
const ConceptController = require('../controllers/ConceptController');
const apiLimiter = require('../middleware/rateLimiter');

const router = express.Router();
const controller = new ConceptController();

router.post('/review/start', apiLimiter, (req, res) => controller.startReview(req, res));
router.post('/review/continue', apiLimiter, (req, res) => controller.continueReview(req, res));
router.post('/review/end', apiLimiter, (req, res) => controller.endReview(req, res));

module.exports = router;
```

### Frontend API Service

```javascript
// services/api.js
export const reviewApi = {
    startReview: (conceptId, audienceLevel) =>
        apiClient.post('/concepts/review/start', { conceptId, audienceLevel }),

    continueReview: (conversationHistory, userMessage) =>
        apiClient.post('/concepts/review/continue', { conversationHistory, userMessage }),

    endReview: (conceptId, conversationHistory, audienceLevel) =>
        apiClient.post('/concepts/review/end', { conceptId, conversationHistory, audienceLevel }),

    transcribeAudio: (audioBlob) => {
        const formData = new FormData();
        formData.append('audio', audioBlob, 'recording.webm');
        return apiClient.post('/transcribe', formData, {
            headers: { 'Content-Type': 'multipart/form-data' }
        });
    }
};
```

---

## Troubleshooting

### Common Issues

1. **"Invalid API key"**
   - Check `.env` file exists and is loaded
   - Verify API keys are correct
   - Ensure no extra spaces in `.env` values

2. **"Rate limit exceeded"**
   - Implement exponential backoff
   - Add rate limiting middleware
   - Consider upgrading API tier

3. **"Transcription failed"**
   - Check audio file format (webm, wav, mp3)
   - Verify file size < 25MB
   - Ensure microphone permissions granted

4. **"JSON parsing error"**
   - AI sometimes returns non-JSON text
   - Add robust error handling
   - Provide clear instructions in prompt

---

## Next Steps

1. Set up environment variables
2. Test Anthropic API with sample text
3. Test Whisper API with sample audio
4. Implement error handling and retries
5. Add rate limiting
6. Monitor API usage during development

Good luck with your API integrations! 🤖
