# Intern Assignment: AI Tutor Chatbot with Contextual Images

## Goal
Build a **mini AI tutor chatbot** that:
1. Accepts a topic PDF upload (e.g., a chapter on "Production of Sound")
2. Uses an LLM to explain concepts from the PDF in a pedagogical, conversational way
3. Automatically displays relevant educational images (diagrams/illustrations) as the AI explains concepts
4. Allows users to ask follow-up questions and continue the learning conversation

This is a **standalone, testable project** — candidates don't need access to our full codebase. The deliverable is a working chatbot with UI that demonstrates contextual image integration with LLM-driven explanations.

## High-level requirements (must-have)
1. **PDF Upload & Extraction**: User uploads a chapter PDF → Extract text content
2. **LLM Integration**: Use an LLM (OpenAI/Gemini/Groq free tier) to explain the topic conversationally
3. **Image Metadata System**: Store 4-8 images per topic with metadata (title, keywords, description)
4. **Contextual Image Display**: As the AI explains, automatically show relevant images based on keyword matching or LLM function calls
5. **Chat Interface**: Simple chat UI where users can ask follow-up questions
6. **Deliverables**: Working chatbot (deployed or local), GitHub repo, sample PDF + images, demo video (3-5 min)

## Why This Assignment?
This tests the exact skills we need:
- ✅ LLM prompt engineering (pedagogical explanations)
- ✅ PDF text extraction and chunking
- ✅ Contextual image detection and display
- ✅ Clean UI/UX for learning experiences
- ✅ End-to-end integration (fully testable)

## Audience & Constraints
- Candidates build this from scratch — no access to our repo or API keys
- Use **free-tier services**: OpenAI free credits, Gemini API (free tier), Groq (free), Vercel/Railway/Render hosting
- Keep it simple: A single-page chat UI is fine (no fancy auth or database required)

## Complete User Flow

```
1. User uploads chapter PDF (e.g., "Sound - Class 8 Science Chapter")
   ↓
2. Backend extracts text from PDF
   ↓
3. User clicks "Start Learning" → Chat interface opens
   ↓
4. AI tutor introduces the topic: "Hi! Let's learn about sound production. When you strike a bell with a hammer..."
   ↓
5. [Image automatically appears] → "School Bell Vibration" diagram shows up next to the explanation
   ↓
6. User asks: "How do vocal cords produce sound?"
   ↓
7. AI explains vocal cords → [Image appears] → "Human Larynx Diagram"
   ↓
8. Conversation continues with contextual images appearing as concepts are explained
```

## Technical Architecture (What to Build)

### 1. Backend (Flask/FastAPI recommended)
- **POST /upload**: Accepts PDF file → Extract text → Return topic_id
- **POST /chat**: Accepts { topic_id, user_message } → Calls LLM → Returns AI response
- **GET /images/:topic_id**: Returns all available images for the topic
- **Static /images/**: Serves image files

### 2. Image Metadata System
Store images with metadata in a simple JSON file or in-memory structure:
```json
{
  "topic": "Production of Sound",
  "images": [
    {
      "id": "img_001",
      "filename": "school_bell_vibration.png",
      "title": "School Bell Vibration",
      "description": "Bell struck by hammer showing vibration and sound waves",
      "keywords": ["bell", "hammer", "vibration", "sound waves", "strike"],
      "display_order": 1
    },
    {
      "id": "img_002",
      "filename": "rubber_band_vibration.png",
      "title": "Rubber Band Vibration",
      "description": "Stretched rubber band producing sound when plucked",
      "keywords": ["rubber band", "pluck", "vibration", "stretched", "elasticity"],
      "display_order": 2
    }
  ]
}
```

### 3. Frontend (Simple HTML/JS or React)
- **Upload Section**: Drag-drop or file input for PDF
- **Chat Interface**: 
  - Message bubbles (user + AI)
  - Image cards that appear inline with AI responses
  - Input box for follow-up questions
- **Image Display**: Show image with title when detected (max 1-2 images per response to avoid clutter)

## What Candidates Must Build

### Core Deliverables:
1. **Working Chatbot Application**:
   - Backend API (Flask/FastAPI/Express)
   - Simple frontend UI (HTML+JS or React — keep it minimal)
   - PDF text extraction (PyPDF2, pdfplumber, or pdf-parse)
   - LLM integration (OpenAI, Gemini, or Groq API)
   - Image detection and display logic

2. **Sample Content**:
   - 1 sample PDF (we'll provide OR they find a NCERT chapter PDF)
   - 4-8 images for the topic with metadata JSON
   - Images should be educational diagrams/illustrations (can source from open education resources)

3. **Image Detection Strategy** (pick one or both):
   - **Keyword Matching**: Scan AI response for keywords from metadata → Show matching image
   - **LLM Function Calling**: Use OpenAI function calls or structured output to explicitly request images

4. **README with**:
   - Setup instructions (how to run locally)
   - LLM prompt design (how you instructed the AI to be pedagogical + mention concepts)
   - Image detection explanation (how you decide which image to show)
   - Sample conversation flow

5. **Demo Video (3-5 min)**:
   - Upload a PDF
   - Start a chat → Show AI explaining with images appearing contextually
   - Ask 2-3 follow-up questions
   - Explain your detection strategy briefly

6. **Deployment** (optional but preferred):
   - Deploy to Vercel/Railway/Render OR provide clear local run steps

## Integration Design Notes (what we expect)
Candidates should explain and demonstrate the following approaches. You don't have to implement both fully, but explain both and implement at least one:

A. Prompt Augmentation + Client-side Keyword Detection (Recommended starter)
- When a tutoring session starts, the frontend fetches `GET /topics/:topicId/images` and appends a compact list to the AI system prompt (title + short description + keywords only). Example:
  - "Available images for Production of Sound: 1) School Bell Vibration — keywords: bell, hammer, vibration; 2) Rubber Band Vibration — keywords: rubber band, pluck, vibration"
- When the AI sends a transcript chunk or final response, the frontend lowercases the text and searches for any keyword matches. If found, show the corresponding image card inline with the transcript.
- Keep shown-image tracking to avoid duplicates per session.

B. Function-call / Structured Request (Optional advanced)
- If using a model that supports function calls or structured responses, the AI can explicitly request an image with a small JSON payload, e.g.: {"action":"show_image","image_id":"img_002"}
- The frontend listens for such structured outputs and shows the requested image. This is more explicit and reduces false positives but requires model support or prompt-level conventions.

C. Hierarchical Filtering
- For scale, the service must expose images per `topicId` to avoid polluting prompts with cross-topic images. Only include images for the current topic in the prompt/detection.

D. Fallback & UX
- If multiple images match, show highest-priority (display_order) or show a small carousel with thumbnails.
- Provide a dismiss/close control. If user re-requests, show image again.

## Prompt Examples (to include in README)

1) Compact prompt augmentation (ideal for small lists):
```
You have access to the following visual aids for the topic "Production of Sound":
1) School Bell Vibration — keywords: bell, hammer, vibration
2) Rubber Band Vibration — keywords: rubber band, pluck, vibration
When you explain concepts, you may reference these images. The client will display an appropriate diagram when you mention a keyword.
```

2) Function-call convention (if function calling not available, mimic via JSON token):
```
If you want the client to show an image, emit a single line with JSON: {"show_image":"img_002"}
Do not include additional commentary on that line. Use this only when you explicitly want the learner to view a diagram.
```

## Sample client-side detection pseudocode (to include)
```js
// images = fetched metadata array
const shown = new Set();
function detectAndShow(aiText) {
  const text = aiText.toLowerCase();
  for (const img of images) {
    for (const kw of img.keywords) {
      if (text.includes(kw.toLowerCase()) && !shown.has(img.id)) {
        showImageCard(img);
        shown.add(img.id);
        return; // show one image at a time
      }
    }
  }
}
```

## Acceptance Criteria (what we will test)
- The API runs and responds on the public URL provided (or locally via clear run steps).
- `GET /topics/:topicId/images` returns the sample metadata and images.
- A short README explains the integration steps and shows example prompt augmentation.
- The candidate implements the client-side detection flow (HTML+JS or small React page) that consumes a simulated AI transcript and displays images.
- The demo video demonstrates the full flow end-to-end.

## Evaluation Rubric
- Functionality: 40% — API endpoints implemented; sample images served; client detection works
- Integration & Prompting: 25% — Clear, practical prompt snippets and detection strategy
- Code Quality: 15% — Clean, modular code and README
- Deployment & Demo: 10% — Live deployment or clear run instructions + demo video
- Thoughtfulness & Scalability: 10% — Notes on scale, hierarchical filtering, and fallback UX

## Interview Follow-ups
- How would you avoid false positives in keyword detection? (expect suggestions: weighted keywords, word boundaries, regex, embeddings fallback)
- How would you scale to thousands of images? (expect: per-topic images, database indexing, optional vector search)
- How would you integrate the function-call approach with Gemini/OpenAI? (expect: function schemas or structured tokens)
- How would you measure whether showing images helps learning? (A/B testing, engagement metrics)

## Starter hints & recommended tech
- Quick stack: Node + Express + low-effort frontend (static HTML + small JS) or FastAPI + tiny frontend
- Storage for demo: commit `sample-images/` and serve statically; show how to switch to Cloudinary or Supabase Storage
- Hosting: Vercel (static + serverless), Railway (backend), or Render — include one-click hints in README

## Submission
- GitHub repo link
- Public URL (deployed) or clear local run steps
- Demo video link (2–4 min)
- README explaining how to integrate with an AI tutor (include prompt samples and detection pseudocode)

---

*Notes for hiring team:* This assignment tests prompt engineering, basic backend APIs, frontend integration, and a pragmatic approach to show images contextually with an LLM-driven tutor. It keeps sensitive code/APIs private — candidates produce an integration that can be dropped into your tutor flow later.
