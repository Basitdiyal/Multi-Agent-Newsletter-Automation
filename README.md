# 📰 Multi-Agent Newsletter Automation

An end-to-end **AI-powered newsletter creation workflow** built with **n8n** and **multi-agent orchestration**.
This system automatically researches, plans, writes, edits, and prepares a complete HTML newsletter draft — ready for review and sending via Gmail — every week.

---


![Uploading News Letter Automation.png…]()

## 🧭 Overview

The **Multi-Agent Newsletter Automation** workflow combines search intelligence and generative AI to produce professional weekly newsletters with minimal human effort.

It uses three main AI agents — **Planner**, **Writer**, and **Editor** — coordinated by **n8n** to handle each stage of the process:

1. Collect relevant industry articles from the past week.
2. Identify key topics and define the edition title.
3. Perform deep research on each topic.
4. Write well-structured newsletter sections.
5. Edit, compile, and format the final newsletter into clean HTML.
6. Create a **Gmail draft** for final review and sending.

---

## 🧩 Core Components

| Step                             | Node Type                    | Description                                                                                                        |
| -------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **1. Schedule Trigger**          | Schedule                     | Starts the workflow weekly (e.g., every Sunday at midnight). Keeps automation consistent.                          |
| **2. Initial Research (Tavily)** | Tavily → Search              | Searches for trending or relevant news in your chosen niche (e.g., “AI adoption for small businesses”).            |
| **3. Planning Agent**            | AI Agent                     | Proposes an edition title and exactly three topics based on recent articles.                                       |
| **4. Split Out (Topics)**        | Item Lists → Split Out Items | Breaks the topic array into three separate items for individual processing.                                        |
| **5. Topic Research**            | Tavily → Search              | Conducts deeper research on each topic for more context and raw content.                                           |
| **6. Section Writer Agent**      | AI Agent                     | Writes a standalone, well-formatted section (350–500 words) for each topic in Markdown.                            |
| **7. Aggregate Sections**        | Aggregator                   | Merges all three sections into one structured list.                                                                |
| **8. Editor Agent**              | AI Agent                     | Converts sections into responsive HTML, adds headers, sources, and a clean layout, and generates an email subject. |
| **9. Create Draft (Gmail)**      | Gmail → Create Draft         | Creates a Gmail draft with the generated subject and HTML body.                                                    |

---

## ⚙️ Step-by-Step Setup

### **1. Schedule Trigger**

* **Node:** Schedule Trigger
* **Frequency:** Every 1 week (Sunday, 00:00)
* Keep “Active” turned off while building the workflow to avoid premature runs.

---

### **2. Initial Research (Tavily)**

* **Node:** Tavily → Search
* **Query Example:** `AI adoption for small businesses`
* **Options:**

  * `topic`: news
  * `time_range`: week
  * `max_results`: 3
  * `include_raw_content`: false
* **Rename Node:** `Initial Research`
* **Tip:** After a successful test, **Pin Data** to save results for later testing.

---

### **3. Planning Agent (Title + Topics)**

* **Node:** AI Agent
* **Rename:** `Planning Agent`
* **Purpose:** Generates the newsletter title and 3 concise topics.

**System Prompt:**

```text
You are an expert newsletter planner. You receive 3–5 article digests (title, URL, date, summary).
Propose a catchy edition title (≤ 80 chars) and exactly 3 concise topics for small-business readers.
Constraints: unique topics; no duplicates; no clickbait; be informative.
```

**JSON Schema Output:**

```json
{
  "type": "object",
  "properties": {
    "title": { "type": "string", "minLength": 10 },
    "topics": {
      "type": "array",
      "minItems": 3,
      "maxItems": 3,
      "items": { "type": "string" }
    }
  },
  "required": ["title", "topics"]
}
```

---

### **4. Split Out Topics**

* **Node:** Item Lists → Split Out Items
* **Source:** `{{$node["Planning Agent"].json["topics"]}}`
* This creates 3 separate executions — one per topic.

---

### **5. Topic Research (Tavily)**

* **Node:** Tavily → Search
* **Rename:** `Topic Research`
* **Query:** `={{ $json }}` (each topic name)
* **Options:**

  * `time_range`: "month"
  * `max_results`: 5
  * `include_raw_content`: true
* **Purpose:** Gather detailed content for each topic.

---

### **6. Section Writer**

* **Node:** AI Agent
* **Rename:** `Section Writer`
* **Purpose:** Writes a standalone section for each topic in Markdown format.

**System Prompt:**

```text
You are a professional newsletter section writer.
Write ONE section (350–500 words) for small-business leaders using the provided research.
Start with an H2 heading matching the topic, cite sources inline [1], [2], and add a short “Sources” list at the end.
Tone: expert, clear, and engaging.
Output plain Markdown.
```

---

### **7. Aggregate Sections**

* **Node:** Aggregator
* **Rename:** `Sections`
* **Purpose:** Combine all section outputs into a single item:

  ```json
  { "sections": [sec1, sec2, sec3] }
  ```

---

### **8. Editor Agent (Final HTML + Subject)**

* **Node:** AI Agent
* **Rename:** `Editor`
* **Purpose:** Converts Markdown sections into styled HTML and generates the newsletter’s email subject.

**System Prompt:**

```text
You are the newsletter editor and layout stylist.
Input: title + 3 Markdown sections with [n] footnotes.

Output:
1. subject — engaging email subject line (≤ 80 chars)
2. content — responsive HTML body (no DOCTYPE or <html> tags)

HTML Rules:
- Include header with title and date {{ $now.format("DDD") }}
- Add short intro paragraph
- Convert sections to HTML
- Add “Key Sources” list with deduplicated links
- Include clean inline CSS for readability
- Use semantic headings and accessible markup
```

**JSON Schema Output:**

```json
{
  "type": "object",
  "properties": {
    "subject": { "type": "string" },
    "content": { "type": "string" }
  },
  "required": ["subject", "content"]
}
```

---

### **9. Gmail Draft Creation**

* **Node:** Gmail → Create Draft
* **Subject:** `={{ $json.output.subject }}`
* **Email Type:** HTML
* **Message:** `={{ $json.output.content }}`
* Creates a formatted Gmail draft for manual review and sending.

---

## 🧠 Technologies Used

| Component                | Purpose                                           |
| ------------------------ | ------------------------------------------------- |
| **n8n**                  | Workflow automation and multi-agent orchestration |
| **Tavily Search API**    | Topic and article data gathering                  |
| **Gemini 2.5 Pro (LLM)** | Generative AI for planning, writing, and editing  |
| **Gmail API**            | Automated draft creation for newsletters          |

---

## 🗓️ Schedule and Maintenance

* Runs **every Sunday at 00:00** (configurable).
* You can manually trigger the workflow during testing.
* Always **Pin Data** at key nodes to prevent excessive API usage.

---

## ⚙️ Customization

* Change **topic/niche** in the Initial Research node.
* Modify **schedule frequency** or timezone in the Trigger node.
* Update **model providers** (e.g., OpenAI, Gemini, Claude, Mistral).
* Edit **styling instructions** in the Editor prompt to match your brand.

---

## 🧾 Output Example

| Output           | Description                                                                        |
| ---------------- | ---------------------------------------------------------------------------------- |
| **Subject:**     | “How Small Businesses Are Winning With AI (Week 42)”                               |
| **Content:**     | Responsive HTML newsletter including 3 topic sections, footnotes, and key sources. |
| **Destination:** | Gmail Draft (ready for review).                                                    |

---

## 📬 Benefits

✅ Fully automated weekly newsletter
✅ Multi-agent orchestration for better accuracy and style
✅ Human-editable drafts via Gmail
✅ Structured JSON schema for reliable outputs
✅ Reduces research and writing time by 90%

---

## 🧩 Use Cases

* Weekly tech or startup newsletters
* Company internal digests
* Industry intelligence summaries
* Marketing campaign content automation

