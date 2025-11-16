# Intern Assignment: Contextual Image Service for AI Tutor

## Goal
Add a contextual image service to an existing AI tutoring flow so that when the AI tutor (voice/text) mentions a concept, the UI displays relevant educational images (diagrams/illustrations) alongside the transcript. The assignment is intentionally standalone — candidates should not need access to the full repo. The output should be a self-contained service and integration guide that can be integrated into the AI tutor later.

## High-level requirements (must-have)
- Implement a small image metadata service (REST API) that stores and serves image metadata and static image files (or URLs).
- Provide an integration guide and sample code showing how the AI tutoring frontend can:
  - Augment the system prompt with the topic's image context
  - Detect when the AI mentions a concept and request/show a relevant image
  - Support two detection modes: fast keyword detection (client-side) and an explicit function-call style (AI signals image request)
- Deliverables: GitHub repo, deployed service (free tier), README, sample prompt templates, sample image metadata JSON, and a short demo video (2–4 min).

## Audience & Constraints
- Candidates won't have access to your full repo or API keys. Build a standalone microservice that can be integrated later.
- Use only free-tier services for hosting/storage (Railway, Render, Vercel, Supabase free storage, Cloudinary free). Local filesystem for images is acceptable for demo but show how to swap to cloud storage.

## Minimal API Specification
(Expose these endpoints — simple JSON + static files)

1. GET /topics/:topicId/images
- Returns all images for a topic with metadata
- Response example:
```json
[{
  "id": "img_001",
  "url": "https://.../SchoolBellVibration.png",
  "title": "School Bell Vibration",
  "description": "Bell struck by hammer showing vibration and sound waves",
  "keywords": ["bell","hammer","vibration","sound waves"],
  "display_order": 1
}]
```

2. GET /images/:imageId
- Returns image file or 302 redirect to hosted URL

3. POST /images (admin)
- Upload an image and metadata (multipart/form-data)
- For the assignment a simple in-memory or file-based store is fine

4. GET /images/:imageId/metadata
- Returns the single image metadata JSON

5. (Optional) POST /search
- Accepts a short text or keywords, returns best-matching image(s)

## Candidate Tasks (deliver these)
1. Implement the API above (any stack: Node/Express, FastAPI, or simple Flask). Keep the code small and documented.
2. Provide a `sample-images/` folder with 4-8 images and metadata for one topic (e.g., "Production of Sound"). Include the metadata JSON file.
3. Provide a short integration guide (README) showing how a front-end AI tutor would use the service in two ways:
   - Prompt augmentation: include the output of `GET /topics/:topicId/images` in the system prompt so the LLM knows the visual aids available for the topic.
   - Real-time detection: on every AI response, run a detection routine (client-side) that scans the AI transcript for keywords from the metadata and shows a matching image. Also include a function-call style example where the AI returns a special token or JSON snippet to request an image explicitly (simulate if using a model without function-calling).
4. Provide sample prompt snippets to add to the AI system prompt (see "Prompt Examples" below).
5. Deploy the service to a free tier (Railway, Render, Vercel) or provide clear steps to run locally with sample cURL requests.
6. Provide a 2–4 minute demo video showing: upload (if implemented), listing topic images, and simulating the AI response detection flow that shows images.

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
