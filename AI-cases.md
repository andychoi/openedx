Creating training content using OpenAI is one of the most powerful use cases of GenAI in education and LMS platforms like Open edX. Whether you want to generate quizzes, course outlines, video scripts, lesson summaries, or adaptive exercises, OpenAI can streamline the authoring process dramatically.

Below is a step-by-step guide to setting up a training content authoring workflow with OpenAI, either standalone or integrated into Open edX via Tutor.

⸻

🧠 What You Can Auto-Generate with OpenAI

Content Type	Example
✅ Course outline	Generate modules/units from a topic
✅ Lesson content	Explanatory text for concepts
✅ Quiz questions	Multiple-choice, true/false
✅ Video scripts	For instructors or animation
✅ Summary & notes	Recap of concepts per unit
✅ Adaptive hints	Step-by-step assistance for assessments



⸻

🧭 Step-by-Step Guide

1️⃣ Define Authoring Use Cases

Start by deciding what you want to generate. For example:
	•	Given a topic: generate a course outline
	•	Given a lesson: generate quizzes
	•	Given a transcript: generate summaries and notes

Each use case needs a prompt template.

⸻

2️⃣ Set Up a Prompt Template Library

Create reusable prompts using either:
	•	Python string templates
	•	Prompt engineering tools like LangChain
	•	JSON-based prompt templates

Example (Course Outline):

f"""
You are a course designer. Generate a structured course outline on the topic:
"{topic}"

The outline should have:
- 5 modules
- Each module has 3 units
- Use a clear hierarchical structure

Return as JSON with: module title, unit titles
"""



⸻

3️⃣ Build Authoring Script or Admin UI (Options)

🔁 Option A: Python Script (CLI or Jupyter)

Create a Python script using openai SDK:

import openai

openai.api_key = "your-api-key"

def generate_outline(topic):
    prompt = f"""Generate a course outline for: {topic}..."""
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
    )
    return response['choices'][0]['message']['content']

🖥 Option B: Django Admin UI for Authors

Create a model:

class CourseAuthoringRequest(models.Model):
    topic = models.CharField(max_length=255)
    prompt_type = models.CharField(choices=[...])
    output = models.TextField(blank=True)

And a form or admin action that sends the request to OpenAI and saves the result.

⸻

4️⃣ Integrate with Open edX Studio (Optional)

To enable AI authoring inside Studio (CMS):
	•	Extend cms/templates
	•	Add a rich text editor or AI sidebar
	•	Use Django/XBlock backend to send content to OpenAI
	•	Store the result as course unit content or draft

📦 You can do this as a Tutor plugin that:
	•	Adds AI buttons in Studio
	•	Calls a Django API (like /ai/author/outline)
	•	Injects content into the lesson draft area

⸻

5️⃣ Manage API Keys Securely
	•	Use Django settings to store keys
	•	For Azure OpenAI:

AZURE_OPENAI_API_KEY = ...
AZURE_OPENAI_ENDPOINT = ...

	•	Build a proxy backend like /aiassistant/generate/

⸻

6️⃣ Example Use Case: Generate a Quiz

Prompt:

Create 3 multiple choice questions with 4 options each on the topic: "Photosynthesis".
Mark the correct answer and return in JSON format like:
[{"question": ..., "options": [...], "answer": ...}]



⸻

🧠 Advanced: Use Embeddings for Personalized Content

If you want AI to reference your own content, use embeddings:
	•	Embed your course materials using OpenAI Embeddings API
	•	Store in vector DB like FAISS
	•	Use retrieval-augmented generation (RAG) to give AI access to existing lesson content
	•	Example: “Based on our materials, create a summary”

⸻

🛠️ Tools & Frameworks

Tool	Usage
openai SDK	Direct API calls
Django	UI and backend integration
LangChain	Prompt management, memory, retrieval
Streamlit	Quick UI for authoring
FAISS / Qdrant	Vector search for content-aware AI
Tutor Plugin	Open edX integration



⸻

✅ Summary

Step	Description
1. Define goals	What content you want to generate
2. Prompt design	Clear templates with structure
3. Backend API	Django view or script for AI calls
4. Authoring UI	Admin page, CMS plugin, or form
5. Store output	Save to LMS content or logs
6. Secure API	Use proxy/backend, not frontend calls
7. (Optional) RAG	Add content-awareness for AI with vector search



⸻

Would you like a working Django admin example to generate course outlines or quizzes via OpenAI, or a Tutor plugin that adds authoring tools into Studio directly?

Awesome! Let’s build a Tutor plugin that brings AI-powered content authoring into Open edX Studio (CMS) using OpenAI. This plugin will add a simple UI in Studio where content authors can:
	•	Enter a topic or lesson description
	•	Choose a generation type (outline, quiz, summary)
	•	Click a button to generate content
	•	Automatically insert the result into the current unit draft

⸻

📦 Plugin Overview: studioaiauthoring

Feature	Description
Location	Visible inside CMS (Studio)
UI	Sidebar or form inside the unit editor
Backend	Django API that proxies Azure OpenAI
Integration	Uses Tutor’s plugin system



⸻

🧰 Step-by-Step Implementation

1️⃣ Scaffold the Plugin

tutor plugins new studioaiauthoring

This creates:

studioaiauthoring/
├── __init__.py
├── templates/
│   └── studioaiauthoring/
│       └── authoring_ui.html
└── patches/



⸻

2️⃣ Add AI Authoring UI in Studio

Create: templates/studioaiauthoring/authoring_ui.html

<div id="ai-authoring-box" style="padding: 1em; background: #f0f0f0;">
  <h3>AI Content Generator</h3>
  <label>Topic:</label><br>
  <input id="ai-topic" type="text" style="width: 100%;" /><br><br>

  <label>Generate:</label>
  <select id="ai-type">
    <option value="outline">Course Outline</option>
    <option value="summary">Summary</option>
    <option value="quiz">Quiz Questions</option>
  </select><br><br>

  <button onclick="generateAIContent()">Generate</button>
  <pre id="ai-output" style="white-space: pre-wrap; background: #fff; padding: 1em;"></pre>
</div>

<script>
function generateAIContent() {
  const topic = document.getElementById('ai-topic').value;
  const type = document.getElementById('ai-type').value;
  fetch('/aiassistant/generate/', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({ topic: topic, type: type })
  })
  .then(res => res.json())
  .then(data => {
    document.getElementById('ai-output').textContent = data.result || data.error;
  });
}
</script>



⸻

3️⃣ Patch the Studio Unit Edit Page

In __init__.py:

from tutor import hooks

# Inject into Studio Unit page
hooks.Filters.CMS_TEMPLATE_OVERRIDES.add_items([
    ("studioaiauthoring/authoring_ui.html", "unit.html"),
])

This renders the UI on the unit editing screen.

⸻

4️⃣ Add Backend Proxy to OpenAI (Django View)

In your LMS or CMS plugin (shared code or API proxy), expose:

@csrf_exempt
def ai_generate(request):
    data = json.loads(request.body)
    topic = data.get("topic", "")
    type_ = data.get("type", "")

    prompt_map = {
        "outline": f"Create a structured course outline on: {topic}",
        "summary": f"Write a brief summary for the topic: {topic}",
        "quiz": f"Generate 3 multiple choice questions on: {topic}"
    }
    prompt = prompt_map.get(type_, "Describe this topic: " + topic)

    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return JsonResponse({ "result": response.choices[0].message.content })

Mount this at: /aiassistant/generate/ via Django URLs.

⸻

5️⃣ Enable the Plugin

tutor plugins enable studioaiauthoring
tutor config save
tutor local restart cms



⸻

🧠 Result
	•	When a content author opens a unit in Studio, they’ll see the AI Content Generator
	•	They type in a topic, select what to generate (outline, quiz, summary)
	•	The content is generated via OpenAI (Azure OpenAI-compatible too)
	•	The output can be copied into the unit or auto-inserted (optional next step)

⸻

🚀 Optional Enhancements
	•	✅ Auto-insert result into HTML field of the unit
	•	🗃 Save previous generations per user
	•	📚 Train OpenAI with existing course materials using embeddings (RAG)
	•	🔐 Add staff-only visibility control

⸻

Would you like a ready-to-use plugin ZIP or a GitHub repo with this implementation? Or should we expand it to include automatic insertion of content into the unit editor?