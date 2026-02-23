# 📋 Do-It-For-Me (DIFM) Agent - Technical Specification

## 📑 Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Component Specifications](#component-specifications)
4. [API Specifications](#api-specifications)
5. [Data Models](#data-models)
6. [Workflow Details](#workflow-details)
7. [Implementation Details](#implementation-details)
8. [Error Handling](#error-handling)
9. [Security Considerations](#security-considerations)
10. [Testing Requirements](#testing-requirements)
11. [Deployment](#deployment)

---

## 1. Overview

### 1.1 Purpose
DIFM Agent is an AI-powered automation system that extracts actionable steps from YouTube tutorial videos and executes them on target websites using headless browser automation.

### 1.2 Key Features
- **YouTube Transcript Extraction**: Retrieves video transcripts and descriptions using stealth scraping
- **AI-Powered Planning**: Converts natural language instructions into structured JSON action plans
- **Automated Execution**: Executes actions on target websites via headless browser
- **Real-time Monitoring**: Provides X-Ray view for debugging and transparency

### 1.3 Target Use Cases
- Form filling (visa applications, registrations)
- Account registration (trading platforms, services)
- Airdrop claiming processes
- Multi-step web workflows

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────┐
│   User Input    │
│  (YouTube URL)  │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────┐
│      Stage 1: Planner Agent         │
│  ┌───────────────────────────────┐  │
│  │  YouTube Scraper              │  │
│  │  (Stealth Mode)               │  │
│  └───────────┬───────────────────┘  │
│              │                       │
│  ┌───────────▼───────────────────┐  │
│  │  Transcript Parser             │  │
│  └───────────┬───────────────────┘  │
│              │                       │
│  ┌───────────▼───────────────────┐  │
│  │  LLM Processor                │  │
│  │  (GPT-4o/Gemini/Claude)       │  │
│  └───────────┬───────────────────┘  │
│              │                       │
│  ┌───────────▼───────────────────┐  │
│  │  Action Plan Generator        │  │
│  │  (JSON Output)                │  │
│  └───────────┬───────────────────┘  │
└──────────────┼───────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      Stage 2: Executor Agent        │
│  ┌───────────────────────────────┐  │
│  │  Action Plan Parser           │  │
│  └───────────┬───────────────────┘  │
│              │                       │
│  ┌───────────▼───────────────────┐  │
│  │  Browser Engine               │  │
│  │  (Playwright/Puppeteer)       │  │
│  └───────────┬───────────────────┘  │
│              │                       │
│  ┌───────────▼───────────────────┐  │
│  │  Action Executor               │  │
│  │  (Navigate/Click/Type)         │  │
│  └───────────┬───────────────────┘  │
│              │                       │
│  ┌───────────▼───────────────────┐  │
│  │  Result Collector              │  │
│  └───────────┬───────────────────┘  │
└──────────────┼───────────────────────┘
               │
               ▼
┌─────────────────┐
│  User Output    │
│  (Results/Logs) │
└─────────────────┘
```

### 2.2 Technology Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Backend Runtime | Node.js | 18+ | Main application runtime |
| Alternative Runtime | Python | 3.10+ | Alternative implementation |
| LLM Provider | OpenAI GPT-4o | Latest | Natural language processing |
| Alternative LLM | Google Gemini / Anthropic Claude | Latest | Fallback LLM options |
| Browser Automation | Playwright | Latest | Headless browser control |
| Alternative Browser | Puppeteer | Latest | Alternative automation |
| Stealth Plugin | playwright-stealth / puppeteer-extra-plugin-stealth | Latest | Anti-detection |
| Web Framework | Express.js / FastAPI | Latest | API server |
| Logging | Winston / Loguru | Latest | Structured logging |

---

## 3. Component Specifications

### 3.1 YouTube Scraper Module

**Purpose**: Extract transcript and metadata from YouTube videos

**Input**:
- YouTube video URL (string)
- Optional: Language preference

**Output**:
```typescript
interface YouTubeData {
  videoId: string;
  title: string;
  description: string;
  transcript: string;
  language: string;
  duration: number;
  timestamp: string;
}
```

**Requirements**:
- Must use stealth mode to bypass YouTube's anti-bot measures
- Support multiple languages
- Handle videos with/without transcripts
- Retry mechanism for failed requests
- Rate limiting to avoid IP bans

**Implementation Notes**:
- Use `yt-dlp` or `youtube-transcript-api` (Python)
- Use `@distube/ytdl-core` or custom scraper (Node.js)
- Implement user-agent rotation
- Add random delays between requests

### 3.2 LLM Processor Module

**Purpose**: Convert natural language transcript into structured action plan

**Input**:
- Raw transcript text (string)
- Video description (string)
- Optional: Target website URL

**Output**: Action Plan JSON (see Data Models section)

**LLM Prompt Structure**:
```
You are an expert at extracting actionable steps from tutorial videos.

Video Transcript:
{transcript}

Video Description:
{description}

Target Website: {target_url}

Instructions:
1. Extract only actionable steps (verbs + objects)
2. Ignore explanations, background information, and filler words
3. Identify the target website URL if not provided
4. Output a structured JSON action plan
5. Each action must have: type, target, value (if applicable), waitTime

Output Format:
{
  "targetUrl": "string",
  "actions": [
    {
      "type": "navigate|click|type|wait|select|scroll",
      "target": "string (selector or description)",
      "value": "string (for type/select actions)",
      "waitTime": number (milliseconds),
      "description": "string"
    }
  ]
}
```

**Requirements**:
- Strict JSON output validation
- Error handling for malformed responses
- Retry logic with exponential backoff
- Token usage optimization
- Support for multiple LLM providers

### 3.3 Action Plan Validator

**Purpose**: Validate and sanitize action plans before execution

**Validation Rules**:
- All required fields present
- Action types are valid enum values
- Target selectors are non-empty
- Wait times are positive numbers
- Target URL is valid format
- No circular dependencies

**Output**: Validated Action Plan or Error Report

### 3.4 Browser Engine Module

**Purpose**: Execute actions on target website

**Configuration**:
```typescript
interface BrowserConfig {
  headless: boolean;
  stealth: boolean;
  viewport: { width: number; height: number };
  userAgent: string;
  timeout: number;
  retries: number;
}
```

**Supported Actions**:
1. **Navigate**: Load target URL
2. **Click**: Click element by selector/text
3. **Type**: Input text into field
4. **Wait**: Wait for time or element
5. **Select**: Select dropdown option
6. **Scroll**: Scroll to element or position
7. **Screenshot**: Capture page state (for debugging)

**Requirements**:
- Element detection with multiple strategies (CSS selector, XPath, text content)
- Wait for elements to be visible/interactive
- Handle dynamic content loading
- Error recovery mechanisms
- Screenshot capture at each step (optional)

### 3.5 X-Ray Monitor Module

**Purpose**: Provide real-time visibility into agent's reasoning and actions

**Output Format**:
```typescript
interface XRayLog {
  timestamp: string;
  stage: "planning" | "execution";
  step: number;
  action: string;
  status: "success" | "error" | "warning";
  details: Record<string, any>;
  screenshot?: string; // Base64 encoded
}
```

**Features**:
- Real-time log streaming
- Screenshot capture at key steps
- Error highlighting
- Performance metrics
- Action timeline visualization

---

## 4. API Specifications

### 4.1 REST API Endpoints

#### POST `/api/v1/execute`
Execute a YouTube tutorial automation task.

**Request Body**:
```json
{
  "youtubeUrl": "https://www.youtube.com/watch?v=...",
  "targetUrl": "https://example.com/form", // Optional
  "options": {
    "headless": true,
    "screenshot": true,
    "timeout": 30000
  }
}
```

**Response**:
```json
{
  "taskId": "uuid",
  "status": "processing",
  "message": "Task queued successfully"
}
```

#### GET `/api/v1/task/:taskId`
Get task status and results.

**Response**:
```json
{
  "taskId": "uuid",
  "status": "completed|failed|processing",
  "result": {
    "success": true,
    "actionsExecuted": 5,
    "errors": [],
    "screenshots": ["base64..."],
    "finalUrl": "https://example.com/success"
  },
  "logs": [...]
}
```

#### GET `/api/v1/task/:taskId/xray`
Get X-Ray view logs (streaming).

**Response**: Server-Sent Events (SSE) stream

#### POST `/api/v1/plan`
Generate action plan without execution (for testing).

**Request Body**:
```json
{
  "youtubeUrl": "https://www.youtube.com/watch?v=...",
  "targetUrl": "https://example.com"
}
```

**Response**:
```json
{
  "plan": {
    "targetUrl": "...",
    "actions": [...]
  },
  "confidence": 0.95
}
```

### 4.2 WebSocket API (Optional)

For real-time updates:
- `ws://host/task/:taskId` - Subscribe to task updates

---

## 5. Data Models

### 5.1 Action Plan Schema

```typescript
interface ActionPlan {
  version: string; // "1.0"
  targetUrl: string;
  metadata: {
    sourceVideo: string;
    extractedAt: string;
    llmModel: string;
  };
  actions: Action[];
}

interface Action {
  id: string; // UUID
  type: ActionType;
  target: string; // CSS selector, XPath, or text description
  value?: string; // For type/select actions
  waitTime?: number; // Milliseconds
  description: string;
  retries?: number;
  timeout?: number;
  conditions?: Condition[]; // Optional pre-conditions
}

enum ActionType {
  NAVIGATE = "navigate",
  CLICK = "click",
  TYPE = "type",
  SELECT = "select",
  WAIT = "wait",
  SCROLL = "scroll",
  SCREENSHOT = "screenshot"
}

interface Condition {
  type: "elementExists" | "textContains" | "urlMatches";
  value: string;
}
```

### 5.2 Execution Result Schema

```typescript
interface ExecutionResult {
  taskId: string;
  status: "success" | "failed" | "partial";
  startTime: string;
  endTime: string;
  duration: number;
  actionsExecuted: number;
  actionsTotal: number;
  errors: ExecutionError[];
  screenshots: string[]; // Base64 encoded
  finalState: {
    url: string;
    title: string;
    screenshot: string;
  };
}

interface ExecutionError {
  actionId: string;
  actionType: ActionType;
  errorType: "timeout" | "notFound" | "blocked" | "unknown";
  message: string;
  timestamp: string;
}
```

---

## 6. Workflow Details

### 6.1 Complete Execution Flow

```
1. User submits YouTube URL
   ↓
2. Validate URL format
   ↓
3. Extract transcript (YouTube Scraper)
   ├─ Success → Continue
   └─ Failure → Return error
   ↓
4. Process transcript with LLM (Planner)
   ├─ Success → Generate Action Plan
   └─ Failure → Retry with different prompt
   ↓
5. Validate Action Plan
   ├─ Valid → Continue
   └─ Invalid → Return error with details
   ↓
6. Initialize Browser Engine
   ↓
7. Execute Actions Sequentially
   ├─ Navigate to target URL
   ├─ For each action:
   │   ├─ Wait for element (if needed)
   │   ├─ Execute action
   │   ├─ Capture screenshot (if enabled)
   │   ├─ Log to X-Ray
   │   └─ Handle errors
   └─ Continue to next action
   ↓
8. Collect final state
   ↓
9. Return results to user
```

### 6.2 Error Recovery Strategies

1. **Element Not Found**:
   - Retry with different selector strategies
   - Use AI vision to locate element
   - Fallback to text-based search

2. **Timeout**:
   - Increase wait time
   - Check for loading indicators
   - Retry action

3. **Blocked/Anti-bot**:
   - Rotate user agent
   - Add random delays
   - Use stealth mode enhancements

4. **LLM Failure**:
   - Retry with simplified prompt
   - Fallback to alternative LLM provider
   - Return partial plan with warnings

---

## 7. Implementation Details

### 7.1 Project Structure

```
do-it-for-me/
├── src/
│   ├── planner/
│   │   ├── youtube-scraper.ts
│   │   ├── transcript-parser.ts
│   │   ├── llm-processor.ts
│   │   └── plan-generator.ts
│   ├── executor/
│   │   ├── browser-engine.ts
│   │   ├── action-executor.ts
│   │   ├── element-finder.ts
│   │   └── result-collector.ts
│   ├── api/
│   │   ├── routes/
│   │   ├── controllers/
│   │   └── middleware/
│   ├── models/
│   │   ├── action-plan.ts
│   │   └── execution-result.ts
│   ├── utils/
│   │   ├── logger.ts
│   │   ├── validator.ts
│   │   └── errors.ts
│   └── xray/
│       ├── monitor.ts
│       └── logger.ts
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── config/
│   ├── default.json
│   └── production.json
├── docs/
│   └── api.md
├── package.json
├── tsconfig.json
└── README.md
```

### 7.2 Key Dependencies

**Node.js Version**:
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "playwright": "^1.40.0",
    "playwright-stealth": "^1.0.6",
    "openai": "^4.20.0",
    "@google/generative-ai": "^0.2.0",
    "winston": "^3.11.0",
    "zod": "^3.22.0",
    "uuid": "^9.0.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.3.0",
    "jest": "^29.7.0",
    "@types/jest": "^29.5.0"
  }
}
```

### 7.3 Environment Variables

```bash
# LLM Configuration
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...
ANTHROPIC_API_KEY=...

# Server Configuration
PORT=3000
NODE_ENV=development|production

# Browser Configuration
BROWSER_HEADLESS=true
BROWSER_TIMEOUT=30000
BROWSER_STEALTH=true

# Logging
LOG_LEVEL=info|debug|error
XRAY_ENABLED=true
```

---

## 8. Error Handling

### 8.1 Error Types

| Error Code | Description | HTTP Status |
|------------|-------------|-------------|
| `INVALID_URL` | Invalid YouTube URL format | 400 |
| `TRANSCRIPT_NOT_FOUND` | Video has no transcript | 404 |
| `SCRAPING_FAILED` | Failed to scrape YouTube | 500 |
| `LLM_ERROR` | LLM processing failed | 500 |
| `INVALID_PLAN` | Generated plan is invalid | 500 |
| `BROWSER_ERROR` | Browser initialization failed | 500 |
| `ELEMENT_NOT_FOUND` | Target element not found | 500 |
| `EXECUTION_TIMEOUT` | Action execution timeout | 504 |

### 8.2 Error Response Format

```json
{
  "error": {
    "code": "ELEMENT_NOT_FOUND",
    "message": "Could not find element with selector: #submit-button",
    "details": {
      "actionId": "uuid",
      "actionType": "click",
      "suggestions": [
        "Try using text-based search",
        "Check if element is in iframe"
      ]
    },
    "timestamp": "2026-02-23T14:30:00Z"
  }
}
```

---

## 9. Security Considerations

### 9.1 Input Validation
- Sanitize all user inputs
- Validate URL formats
- Rate limiting on API endpoints
- Request size limits

### 9.2 API Security
- Authentication tokens (JWT)
- API key validation
- CORS configuration
- HTTPS enforcement

### 9.3 Browser Security
- Sandboxed browser instances
- Resource limits (memory, CPU)
- Timeout enforcement
- No access to local filesystem

### 9.4 Data Privacy
- No storage of sensitive user data
- Transcript data purged after processing
- Screenshots encrypted if stored
- GDPR compliance considerations

---

## 10. Testing Requirements

### 10.1 Unit Tests
- YouTube scraper module
- LLM processor with mock responses
- Action plan validator
- Element finder strategies

### 10.2 Integration Tests
- End-to-end workflow with mock YouTube
- Browser automation with test pages
- API endpoint testing

### 10.3 E2E Tests
- Complete flow with real YouTube video (unlisted)
- Execution on dummy registration form
- Error scenario testing

### 10.4 Test Coverage Goals
- Code coverage: > 80%
- Critical paths: 100%
- Error handling: > 90%

---

## 11. Deployment

### 11.1 Development Setup

```bash
# Install dependencies
npm install

# Set environment variables
cp .env.example .env

# Run development server
npm run dev

# Run tests
npm test
```

### 11.2 Production Deployment

**Docker Configuration**:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**Docker Compose**:
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - ./logs:/app/logs
```

### 11.3 Monitoring & Logging
- Application logs: Winston
- Error tracking: Sentry (optional)
- Performance monitoring: Prometheus (optional)
- Health check endpoint: `/health`

---

## 12. MVP Demo Scenario

### 12.1 Test Video
- **Title**: "How to Register for a Web3 Event"
- **Duration**: 45 seconds
- **Visibility**: Unlisted
- **Content**: Simple form filling tutorial

### 12.2 Target Website
- **URL**: `https://demo.difm-agent.com/register`
- **Form Fields**:
  - Name (text input)
  - Email (text input)
  - Register button

### 12.3 Expected Flow
1. Extract transcript from YouTube video
2. LLM identifies: Navigate → Type Name → Type Email → Click Register
3. Execute actions on demo website
4. Capture screenshots at each step
5. Display X-Ray log showing reasoning

### 12.4 Success Criteria
- ✅ Successfully extracts transcript
- ✅ Generates valid action plan
- ✅ Executes all actions correctly
- ✅ Form submission successful
- ✅ X-Ray log shows clear reasoning

---

## 13. Future Enhancements

### 13.1 Phase 2 Features
- Multi-page workflows
- Form validation handling
- CAPTCHA solving integration
- Video frame analysis (computer vision)
- Multi-language support

### 13.2 Phase 3 Features
- User dashboard
- Task scheduling
- Task templates
- Browser extension
- Mobile app

---

## 14. Appendix

### 14.1 Action Plan Example

```json
{
  "version": "1.0",
  "targetUrl": "https://demo.difm-agent.com/register",
  "metadata": {
    "sourceVideo": "https://youtube.com/watch?v=abc123",
    "extractedAt": "2026-02-23T14:30:00Z",
    "llmModel": "gpt-4o"
  },
  "actions": [
    {
      "id": "action-1",
      "type": "navigate",
      "target": "https://demo.difm-agent.com/register",
      "description": "Navigate to registration page",
      "waitTime": 2000
    },
    {
      "id": "action-2",
      "type": "type",
      "target": "#name-input",
      "value": "John Doe",
      "description": "Enter name in the name field",
      "waitTime": 1000
    },
    {
      "id": "action-3",
      "type": "type",
      "target": "#email-input",
      "value": "john@example.com",
      "description": "Enter email address",
      "waitTime": 1000
    },
    {
      "id": "action-4",
      "type": "click",
      "target": "#register-button",
      "description": "Click the register button",
      "waitTime": 3000
    }
  ]
}
```

### 14.2 Glossary

- **Action Plan**: Structured JSON representation of steps to execute
- **Planner**: Stage 1 agent that extracts and plans actions
- **Executor**: Stage 2 agent that executes actions on browser
- **X-Ray**: Real-time monitoring view showing agent reasoning
- **Stealth Mode**: Browser configuration to avoid bot detection
- **Headless Browser**: Browser without GUI, controlled programmatically

---

**Document Version**: 1.0  
**Last Updated**: February 23, 2026  
**Author**: DIFM Development Team
