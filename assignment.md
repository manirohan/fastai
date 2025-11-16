cat > INTERN_ASSIGNMENT_IMAGE_SERVICE.md << 'EOF'
# Intern Assignment: Contextual Image Service for AI Quick Revision Teacher

## 📋 Project Overview

Smart Revision Squad is an educational platform for Class 6-12 students featuring AI-powered voice tutoring. Currently, our **AI Quick Revision Teacher** (`AITutoringInterface`) provides real-time voice conversations using Google Gemini 2.5 Flash Live API, but lacks visual support.

**Your Mission**: Add a contextual image service that displays relevant educational images/diagrams during the conversation based on the transcript and topic context.

---

## 🎯 What You're Building

A smart image suggestion system that:
1. **Monitors** the real-time conversation transcript between student and AI teacher
2. **Identifies** when visual aids would enhance learning (e.g., "photosynthesis process", "Pythagorean theorem", "water cycle")
3. **Fetches & displays** contextually relevant images/diagrams from a curated source
4. **Updates dynamically** as the conversation progresses through different concepts

**Example Flow**:
- Student asks: "Can you explain how photosynthesis works?"
- AI responds with explanation (voice)
- **Your system**: Detects "photosynthesis" → Fetches diagram → Displays alongside transcript
- Student asks follow-up: "What about the light-dependent reactions?"
- **Your system**: Updates image to show detailed chloroplast diagram

---

## 🏗️ Current Architecture (What You Need to Know)

### Relevant Files to Study

1. **`src/pages/AITutoringInterface.tsx`** (1000+ lines)
   - Main component for AI voice tutoring
   - Handles Gemini Live API WebSocket connection
   - Manages conversation state in `messages` array
   - Each message: `{ id, type: 'ai' | 'user', content: string, timestamp }`
   - Real-time transcript updates via `handleMessage()` callback

2. **`src/services/geminiService.ts`**
   - Creates WebSocket connection to Gemini Live API
   - Handles bidirectional audio streaming
   - Exposes callbacks: `onMessage`, `onStatusChange`, `onError`
   - System prompt customization support

3. **`supabase/migrations/20251028132701_*.sql`**
   - `topic_learning_prompts` table: Stores custom AI prompts per topic
   - Structure: `topic_id`, `prompt`, `created_at`, `updated_at`
   - Could be extended for image metadata storage

4. **Existing Image Implementation** (Text-based tutor, different from voice tutor)
   - `src/tutor/pages/TutorInterface.tsx`: Text-based chat with image upload
   - `src/tutor/hooks/useTutorChat.ts`: Handles image to base64 conversion
   - **Note**: This is separate from the voice-based `AITutoringInterface` you'll modify

### Key State Variables in AITutoringInterface

```typescript
const [messages, setMessages] = useState<Message[]>([]); // Conversation history
const [currentTopicIndex, setCurrentTopicIndex] = useState(0); // Active topic
const [topics, setTopics] = useState<Topic[]>([]); // All topics in chapter
const [isConnected, setIsConnected] = useState(false); // WebSocket status
const [isSpeaking, setIsSpeaking] = useState(false); // AI is speaking
```

### Message Flow

```
Gemini API → handleMessage() → setMessages() → UI Update
                    ↓
            Your Image Service
                    ↓
            Display Contextual Images
```

---

## 📝 Requirements

### Core Functionality (Must Have)

#### 1. Keyword Extraction from Transcript
- Monitor the `messages` array for new AI/user messages
- Extract educational keywords/concepts from message content
- Use NLP techniques or LLM API to identify visual-worthy topics
- Examples to detect:
  - Scientific concepts: "mitochondria", "water cycle", "cell structure"
  - Mathematical concepts: "quadratic equation", "trigonometry", "graph"
  - Historical events: "French Revolution", "Mughal Empire"
  - Geographical features: "tectonic plates", "river system"

**Suggested Approach**:
- Use OpenAI/Gemini API with prompt: "Extract visual educational keywords from: {transcript}"
- Or simpler: Keyword matching against pre-defined concept database per subject

#### 2. Image Source Integration
Choose ONE of the following:

**Option A: Educational Image APIs**
- [Unsplash API](https://unsplash.com/developers) (High-quality, free tier)
- [Pexels API](https://www.pexels.com/api/) (Free, educational images)
- Search query: Combine extracted keywords + "diagram" or "illustration"

**Option B: Pre-curated Image Database**
- Create Supabase table: `topic_images`
  - Columns: `topic_id`, `keyword`, `image_url`, `caption`, `priority`
- Seed with 20-30 common educational images per subject
- Match keywords to pre-stored image URLs

**Option C: Google Custom Search API**
- Use Custom Search JSON API with site restriction to educational domains
- Filter for images from Khan Academy, Wikipedia, educational sites

#### 3. UI Integration

**Modify `AITutoringInterface.tsx` to add image panel:**

```tsx
// Suggested layout change
<div className="flex flex-col md:flex-row h-screen bg-background">
  {/* Existing sidebar with topics */}
  
  {/* Main chat area */}
  <div className="flex-1 flex flex-col">
    {/* Messages */}
  </div>
  
  {/* NEW: Image panel (collapsible on mobile) */}
  <div className="md:w-80 border-l border-border bg-card">
    <ImageContextPanel 
      currentKeywords={extractedKeywords}
      topicName={getCurrentTopic()?.name}
      subjectName={subjectName}
    />
  </div>
</div>
```

**Image display requirements**:
- Show 1-3 relevant images based on current conversation context
- Include image caption/source attribution
- Smooth transitions when images update (fade in/out)
- Loading state while fetching images
- Fallback UI when no images available
- Collapsible on mobile (icon button to show/hide)

#### 4. Smart Context Awareness

- **Timing**: Don't fetch images for every single message
  - Wait for AI response completion (`response.audio_transcript.done` event)
  - Debounce image updates (minimum 3-5 seconds between updates)
  
- **Relevance**: Track conversation flow
  - Prioritize most recent 2-3 messages for keyword extraction
  - Consider topic name from `topics[currentTopicIndex]`
  - Combine subject + chapter + topic for better search context

- **Caching**: Don't re-fetch same images
  - Store fetched images in component state or localStorage
  - Key by concept/keyword combination

### Bonus Features (Nice to Have)

#### 1. Image Storage in Supabase
- Create migration for `contextual_images` table
- Store: `keyword`, `subject`, `chapter_id`, `topic_id`, `image_url`, `source`, `relevance_score`
- Admin interface to seed images (can be simple CSV upload)

#### 2. Multi-Image Carousel
- Show 3-5 related images
- Allow student to swipe through or select
- Track which images are most viewed (analytics)

#### 3. Image Quality Scoring
- Use LLM to verify image relevance before displaying
- Prompt: "Rate this image's relevance to {concept} from 1-10"
- Only show images with score > 7

#### 4. Adaptive Display
- If student asks "show me a diagram", prioritize diagram-style images
- If "example", show real-world photos
- Context-aware image type selection

#### 5. Student Interaction
- "Show more images like this"
- "This image helped" / "This image didn't help" feedback
- Save preferred images to user's flashcards

---

## 🛠️ Technical Guidance

### New Files to Create

1. **`src/services/imageContextService.ts`**
```typescript
interface ImageResult {
  id: string;
  url: string;
  thumbnail: string;
  caption: string;
  source: string;
  relevanceScore?: number;
}

export class ImageContextService {
  async fetchImages(keywords: string[], context: {
    subject: string;
    topic?: string;
  }): Promise<ImageResult[]> {
    // Your implementation
  }
  
  extractKeywords(transcript: string): Promise<string[]> {
    // Use LLM or keyword extraction
  }
}
```

2. **`src/components/ImageContextPanel.tsx`**
```typescript
interface Props {
  currentKeywords: string[];
  topicName?: string;
  subjectName?: string;
}

export const ImageContextPanel: React.FC<Props> = ({ ... }) => {
  // Fetch and display images
  // Handle loading/error states
  // Responsive collapse on mobile
};
```

3. **`src/hooks/useImageContext.ts`** (optional, but recommended)
```typescript
export const useImageContext = (messages: Message[], topic: Topic) => {
  const [images, setImages] = useState<ImageResult[]>([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    // Debounced keyword extraction + image fetching
  }, [messages, topic]);
  
  return { images, loading };
};
```

4. **`supabase/migrations/YYYYMMDDHHMMSS_add_contextual_images.sql`** (if using Supabase storage)

### Integration Points

#### Where to hook into AITutoringInterface

```typescript
// In AITutoringInterface.tsx

// Add state for images
const [contextualImages, setContextualImages] = useState<ImageResult[]>([]);

// Modify handleMessage to trigger image updates
const handleMessage = (event: any) => {
  // ... existing code ...
  
  // After AI finishes speaking
  if (event.type === 'response.audio_transcript.done') {
    const text = event.text || '';
    if (text) {
      // Trigger image fetch with debounce
      debouncedFetchImages(text, getCurrentTopic(), subjectName);
    }
  }
};

// New debounced image fetcher
const debouncedFetchImages = debounce(async (
  transcript: string, 
  topic: Topic, 
  subject: string
) => {
  const keywords = await imageService.extractKeywords(transcript);
  const images = await imageService.fetchImages(keywords, { 
    subject, 
    topic: topic?.name 
  });
  setContextualImages(images);
}, 3000); // 3 second debounce
```

### API Integration Examples

#### Option 1: Unsplash
```typescript
const UNSPLASH_ACCESS_KEY = 'your_key_here';

async function searchUnsplash(query: string) {
  const response = await fetch(
    `https://api.unsplash.com/search/photos?query=${encodeURIComponent(query + ' educational diagram')}&per_page=3`,
    {
      headers: {
        'Authorization': `Client-ID ${UNSPLASH_ACCESS_KEY}`
      }
    }
  );
  return response.json();
}
```

#### Option 2: Keyword Extraction with Gemini
```typescript
async function extractKeywords(transcript: string): Promise<string[]> {
  const response = await fetch(
    'https://generativelanguage.googleapis.com/v1/models/gemini-pro:generateContent',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-goog-api-key': GEMINI_API_KEY
      },
      body: JSON.stringify({
        contents: [{
          parts: [{
            text: `Extract 2-3 key educational concepts from this conversation that would benefit from visual diagrams or images. Return only the keywords, comma-separated:\n\n${transcript}`
          }]
        }]
      })
    }
  );
  const data = await response.json();
  const keywords = data.candidates[0].content.parts[0].text
    .split(',')
    .map(k => k.trim());
  return keywords;
}
```

### Testing Strategy

1. **Manual Testing Scenarios**:
   - Start conversation about "Photosynthesis" → Verify plant diagram appears
   - Switch topic to "Pythagoras theorem" → Verify triangle diagram appears
   - Ask unrelated question → Verify images don't change unnecessarily
   - Test on mobile → Verify image panel collapses properly

2. **Edge Cases**:
   - No internet connection for image API
   - API rate limits exceeded
   - Invalid/inappropriate images returned
   - Very long transcripts (performance)
   - Rapid topic switching

3. **Performance**:
   - Image loading shouldn't block conversation
   - Memory leak check (cleanup on unmount)
   - Caching working properly

---

## 📦 Deliverables

### 1. Code Submission
- **Format**: Git branch or GitHub PR against `main` branch
- **Include**:
  - Modified `AITutoringInterface.tsx`
  - New `src/services/imageContextService.ts`
  - New `src/components/ImageContextPanel.tsx`
  - Any new database migrations (if applicable)
  - Updated `package.json` (if new dependencies added)

### 2. Demo Video (5-7 minutes)
- **Show**:
  - Live demo of conversation with image updates
  - Explanation of keyword extraction logic
  - Mobile responsiveness
  - Edge case handling (no images found, loading states)
- **Upload to**: YouTube (unlisted) or Loom

### 3. Documentation
- **`IMPLEMENTATION.md`** (1-2 pages):
  - Architecture decisions (which API chosen and why)
  - How keyword extraction works
  - Challenges faced and solutions
  - Future improvement ideas
  - Setup instructions (API keys needed, etc.)

### 4. Bonus: Seeded Image Database
- If using pre-curated approach, provide:
  - SQL migration with sample data (20-30 images)
  - CSV file with image metadata
  - Screenshots of images displayed in UI

---

## 📊 Evaluation Criteria

| Criteria | Weight | What We're Looking For |
|----------|--------|------------------------|
| **Functionality** | 35% | - Images display correctly<br>- Keyword extraction works<br>- Smooth UI integration<br>- No breaking existing features |
| **Code Quality** | 25% | - Clean, readable TypeScript<br>- Proper error handling<br>- Type safety<br>- Reusable components |
| **UX/UI Design** | 20% | - Responsive layout<br>- Smooth transitions<br>- Loading/error states<br>- Doesn't distract from conversation |
| **Performance** | 10% | - Debouncing implemented<br>- Image caching works<br>- No memory leaks<br>- Fast loading |
| **Documentation** | 10% | - Clear setup instructions<br>- Thoughtful architecture explanation<br>- Good demo video |

---

## 🚀 Getting Started

### Step 1: Setup Development Environment
```bash
# Clone the repository
git clone https://github.com/yavarizhar/smart-rev-squad.git
cd smart-rev-squad

# Install dependencies
npm install

# Start development server
npm run dev
# Visit http://localhost:5173
```

### Step 2: Study the Codebase (2-3 hours)
- Read `.github/copilot-instructions.md` (comprehensive architecture guide)
- Study `AITutoringInterface.tsx` - understand message flow
- Test the existing voice tutor feature (need phone OTP to login)
- Look at `src/tutor/` folder for inspiration (text-based tutor with images)

### Step 3: Plan Your Approach (1-2 hours)
- Choose image API (Unsplash vs pre-curated vs Google)
- Decide on keyword extraction method (LLM vs simple matching)
- Sketch UI layout for image panel
- Write pseudocode for main logic flow

### Step 4: Implement Core Features (8-12 hours)
- Create `imageContextService.ts`
- Build `ImageContextPanel.tsx` component
- Integrate into `AITutoringInterface.tsx`
- Test with sample conversations

### Step 5: Polish & Document (3-4 hours)
- Add loading/error states
- Make mobile responsive
- Record demo video
- Write documentation

**Total Estimated Time**: 15-20 hours (3-4 days of focused work)

---

## 💡 Tips for Success

1. **Start Simple**: Get basic image display working before adding smart features
2. **Don't Overthink Keyword Extraction**: A simple LLM prompt works well for MVP
3. **Use Existing Patterns**: Follow the code style in `AITutoringInterface.tsx`
4. **Test Incrementally**: Verify each feature works before moving to next
5. **Ask for Help**: If stuck on Supabase/Firebase auth, focus on the image service logic first
6. **Mobile-First**: Test on mobile early, responsive issues are easier to fix gradually
7. **Fallback Gracefully**: If image API fails, conversation should continue normally

---

## ❓ FAQs

**Q: Do I need to modify the Gemini Live API integration?**  
A: No! The WebSocket connection should remain unchanged. You only add a side effect when messages update.

**Q: Should images update while AI is speaking?**  
A: No, wait for `response.audio_transcript.done` event to avoid distracting students during explanation.

**Q: What if I can't find good images for a concept?**  
A: Show a "No visual aids found" message or hide the image panel. Don't show irrelevant images.

**Q: Can I use a different tech stack for image service?**  
A: Stick to TypeScript + React for frontend. For backend, you can add a simple Node/Python API if needed, but try to keep it client-side for simplicity.

**Q: How do I handle API keys securely?**  
A: For this assignment, hardcoded keys in code are fine (comment in docs that they should be env vars in production). In real project, use Vite env vars: `import.meta.env.VITE_UNSPLASH_KEY`

---

## 📧 Submission

**Send to**: [your-email@company.com]

**Subject**: "Intern Assignment - Image Service - [Your Name]"

**Include**:
1. GitHub repository link or PR link
2. Demo video link (YouTube/Loom)
3. Brief cover note (2-3 sentences about your approach)

**Deadline**: 5 days from assignment receipt

---

## 🎓 What You'll Learn

- React state management with real-time data
- WebSocket integration patterns
- Third-party API integration (image search)
- Responsive UI design for complex layouts
- Performance optimization (debouncing, caching)
- Supabase database design (if bonus features)
- Working with production-quality codebase

---

Good luck! We're excited to see your creative solution to enhancing our AI teacher with visual learning aids. 🚀

If you have questions about the assignment requirements (not implementation help), feel free to reach out.
EOF
