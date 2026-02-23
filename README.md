# 🤖 Do-It-For-Me (DIFM) Agent

**Tagline:** Transform YouTube Tutorial Videos into Automated Actions with AI & Headless Browser.

---

## 💡 Problem Statement

Users often watch tutorial videos on YouTube (e.g., how to fill out a visa form, how to register for a trading platform, how to receive airdrops). However, constantly switching tabs between the video and the target website to follow along (pause/play) is time-consuming and prone to errors.

## 🎯 Solution

DIFM Agent is a background AI assistant. By simply providing a YouTube tutorial link, the agent will "watch" the video, understand the necessary steps, and directly control a headless browser to complete the entire task on the target website on behalf of the user.

## ⚙️ Core Workflow

The project utilizes a **Two-Step Agent Architecture (Plan - Execute)** to overcome the "semantic gap" between natural language and HTML source code:

### Stage 1: Extraction & Planning (The Planner)

1. Users input the YouTube URL.
2. The agent uses a stealth web scraping library to bypass YouTube's anti-bot measures, retrieving the entire automatic transcript and description.
3. LLM (Language Model) reads the raw text, removes unnecessary words, and outputs a standardized **Action Plan in JSON format**.

### Stage 2: Automated Execution (The Executor)

1. The JSON plan is sent to the **Headless Browser Engine** (Playwright/Puppeteer).
2. The agent opens the target website and sequentially executes the actions: **Navigate**, **Click**, **Type** based on the learned content from the video.
3. The result is returned to the user.

## 🛠️ Technology Stack

- **AI/LLM:** OpenAI GPT-4o / Gemini / Claude (Process natural language and convert it into JSON structure)
- **Headless Automation:** Playwright or Puppeteer (Combining the stealth plugin to avoid detection)
- **Backend:** Node.js / Python (Manage data flow and API)
- **DOM Parsing / Vision (Optional Enhancements):** Utilize AI tools to read and understand web interfaces to accurately identify buttons without hardcoding IDs

## 🚀 Demo MVP Scenario (Proof of Concept)

To demonstrate feasibility within the Hackathon context, we will demo the following scenario:

- **Input:** A YouTube video (Unlisted) lasting 45 seconds, titled "How to Register for a Web3 Event"
- **Target Web:** A Dummy Event Registration web page (HTML/CSS) created by the team
- **Action:** The agent will read the YouTube link, automatically open the Dummy web page, fill in Name/Email, and click the "Register" button with no human intervention
- **X-Ray Screen:** An "X-Ray" screen will show the running log to the judges, allowing them to see the AI's reasoning and browser control process

## 🚧 Risks & Mitigation Strategies

**Risk:** YouTube transcripts can be "garbage" and lack grammatical structure.

**Solution:** Use advanced Prompt Engineering to force the LLM to extract only action verbs and target objects, ensuring the output format is strict JSON.

---

## 📋 Project Structure

_Coming soon..._

## 🚀 Getting Started

_Coming soon..._

## 📝 License

_Coming soon..._
