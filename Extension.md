Perfect! Let’s create an AI-enabled Tutor plugin example that integrates an AI assistant into the Open edX LMS using OpenAI’s API.

We’ll simulate an AI chatbot that appears as a sidebar on every LMS page and can answer course-related questions. You could later expand this to integrate with course content, summarize videos, or give intelligent hints.

⸻

🧠 AI Chat Assistant Plugin for Tutor

🎯 What This Plugin Does
	•	Adds an LMS sidebar with a chatbot UI
	•	Sends user queries to the OpenAI API
	•	Displays AI-generated responses
	•	Injected via custom LMS template override

⸻

📦 Step-by-Step Guide

1. Scaffold Your Plugin

tutor plugins new aiassistant
cd "$(tutor plugins printroot)/aiassistant"



⸻

2. Update __init__.py

from tutor import hooks

# Default config values
hooks.Filters.CONFIG_DEFAULTS.add_items([
    ("AIASSISTANT_OPENAI_API_KEY", "your-api-key-here"),
])

# LMS environment variables
hooks.Filters.ENV_PATCHES.add_items([
    ("lms-env", "aiassistant/lms-env.json"),
])

# Mount static assets
hooks.Filters.MOUNT_LMS_STATIC.add_items([
    ("aiassistant/static", "/openedx/edx-platform/themes/aiassistant/static"),
])

# Inject sidebar template
hooks.Filters.LMS_TEMPLATE_OVERRIDES.add_items([
    ("aiassistant/templates/chat_sidebar.html", "footer.html"),
])



⸻

3. Add LMS Environment Patch

Create templates/aiassistant/lms-env.json:

{
  "AIASSISTANT_API_KEY": "{{ AIASSISTANT_OPENAI_API_KEY }}"
}



⸻

4. Inject a Chatbot UI in LMS Footer

Create templates/aiassistant/templates/chat_sidebar.html:

<div id="ai-chatbot" style="position:fixed;bottom:0;right:0;width:300px;height:400px;background:#fff;z-index:9999;border:1px solid #ccc">
  <div style="padding:10px;background:#333;color:#fff">AI Assistant</div>
  <div id="chat-box" style="height:300px;overflow-y:auto;padding:10px;"></div>
  <textarea id="chat-input" style="width:100%;height:60px;"></textarea>
</div>

<script>
  const apiKey = "{{ settings.AIASSISTANT_API_KEY }}";
  const input = document.getElementById('chat-input');
  const chatBox = document.getElementById('chat-box');

  input.addEventListener('keydown', function(e) {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      const message = input.value.trim();
      chatBox.innerHTML += `<div><b>You:</b> ${message}</div>`;
      input.value = '';
      fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${apiKey}`
        },
        body: JSON.stringify({
          model: "gpt-3.5-turbo",
          messages: [{role: "user", content: message}]
        })
      })
      .then(res => res.json())
      .then(data => {
        const reply = data.choices[0].message.content;
        chatBox.innerHTML += `<div><b>AI:</b> ${reply}</div>`;
        chatBox.scrollTop = chatBox.scrollHeight;
      });
    }
  });
</script>



⸻

5. Add Static Folder (optional)

mkdir -p templates/aiassistant/static

(Optional if you want to add your own CSS/JS instead of inline.)

⸻

6. Enable and Test Plugin

tutor plugins enable aiassistant
tutor config save
tutor local restart lms



⸻

✅ Result

You now have a floating AI assistant in your LMS that:
	•	Uses OpenAI’s GPT model
	•	Appears as a chat widget on every page
	•	Can be customized to use content from the course, hints, or knowledge base

⸻

🧠 Bonus Ideas for Expansion
	•	Inject context-aware prompts (e.g., current course, lesson)
	•	Integrate with Open edX course APIs to access specific content
	•	Replace OpenAI with Claude or Azure OpenAI endpoints
	•	Track usage with analytics
	•	Add admin UI for enabling/disabling or viewing chat logs

⸻

Great! Let’s now expand your AI Tutor plugin to offer context-aware help — meaning the AI assistant will respond with information tailored to the current course, unit, or lesson a learner is viewing.

⸻

🎯 Goal: Context-Aware AI Assistant in LMS

The AI chatbot will:
	1.	Detect the current course and page the user is on.
	2.	Include that context in the prompt sent to OpenAI.
	3.	Provide more relevant, helpful responses — e.g., summaries, explanations, or additional guidance.

⸻

🧩 Updated Architecture

LMS Page
 └──> Chat Sidebar JavaScript
       └──> Extracts Course + Unit Info from the DOM or global variables
             └──> Builds a prompt like:
                   "The user is viewing Course: Intro to Python, Unit: Loops.
                    Here's their question: 'Can you explain for loops again?'"
                   └──> Sends to OpenAI for a tailored response.



⸻

🔍 Step-by-Step Implementation

1️⃣ Update LMS Chat Widget (chat_sidebar.html)

Replace the old JS with:

<script>
  const apiKey = "{{ settings.AIASSISTANT_API_KEY }}";
  const input = document.getElementById('chat-input');
  const chatBox = document.getElementById('chat-box');

  function getCourseContext() {
    // Try to access Open edX LMS context
    try {
      const courseName = document.querySelector('[data-course-title]')?.getAttribute('data-course-title')
                          || window.course?.display_name || 'Unknown course';
      const unitTitle = document.querySelector('.xmodule_DisplayNameBlock')?.textContent?.trim() || 'Unknown unit';
      return `The user is currently viewing the course "${courseName}" in the unit "${unitTitle}".`;
    } catch {
      return "Course context could not be determined.";
    }
  }

  input.addEventListener('keydown', function(e) {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      const message = input.value.trim();
      const context = getCourseContext();
      chatBox.innerHTML += `<div><b>You:</b> ${message}</div>`;
      input.value = '';

      const prompt = `${context}\n\nUser's question: "${message}"`;

      fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${apiKey}`
        },
        body: JSON.stringify({
          model: "gpt-3.5-turbo",
          messages: [{ role: "user", content: prompt }]
        })
      })
      .then(res => res.json())
      .then(data => {
        const reply = data.choices[0].message.content;
        chatBox.innerHTML += `<div><b>AI:</b> ${reply}</div>`;
        chatBox.scrollTop = chatBox.scrollHeight;
      });
    }
  });
</script>

✅ What This Does:
	•	Reads course title and unit title from the page.
	•	Appends this info to the user’s query.
	•	Sends a full, context-rich message to the OpenAI API.

⸻

2️⃣ Optional Enhancements

a. 🧠 Include Content Summary (Advanced)

To make it even smarter:
	•	Pull the page content (e.g., learning objectives or video transcript).
	•	Inject this into the prompt (beware of token limits).

const lessonContent = document.querySelector('.xmodule_display')?.innerText?.slice(0, 1000);
const fullPrompt = `${context}\nHere is some content from the page:\n${lessonContent}\n\nUser's question: "${message}"`;



⸻

b. 🧵 Maintain Conversation History (Optional)

To simulate a session:
	•	Store messages in an array and reuse them as messages for OpenAI.
	•	Or save in localStorage for persistence.

⸻

3️⃣ Additional Notes
	•	Works best with well-structured course pages.
	•	Make sure to respect user privacy and never send personally identifiable data.
	•	You can customize the tone by changing the system prompt:

{ "role": "system", "content": "You are a friendly EdTech assistant helping students learn online." }



⸻

🧠 Example Prompt Sent to OpenAI

The user is currently viewing the course "Intro to Python" in the unit "For Loops".
Here is some content from the page:
---
In Python, a for loop is used for iterating over a sequence (like a list, tuple, dictionary, or string)...
---
User's question: "Can you explain for loops again?"



⸻

Perfect! Setting up a Django backend to proxy Azure OpenAI requests is a secure and scalable way to protect API keys, log usage, apply rate-limiting, and extend features (like context enrichment or caching).

⸻

🎯 Goal

Create a Django app (aiassistant) that exposes an endpoint:

POST /aiassistant/chat/

Which:
	•	Accepts user queries from the LMS frontend
	•	Uses Azure OpenAI API to generate responses
	•	Returns the response as JSON
	•	Optionally enriches the prompt with context

⸻

⚙️ 1. Django App Setup

Create an app:

python manage.py startapp aiassistant

Add to INSTALLED_APPS:

# settings.py
INSTALLED_APPS = [
    ...
    'aiassistant',
]



⸻

📁 2. API View (using DRF or plain Django)

File: aiassistant/views.py

import requests
from django.conf import settings
from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse, HttpResponseBadRequest
import json

@csrf_exempt
def chat_with_ai(request):
    if request.method != "POST":
        return HttpResponseBadRequest("Only POST allowed")

    try:
        data = json.loads(request.body)
        user_message = data.get("message", "")
        context = data.get("context", "")

        if not user_message:
            return JsonResponse({"error": "Missing message"}, status=400)

        prompt = f"{context}\n\nUser's question: {user_message}"

        azure_endpoint = settings.AZURE_OPENAI_ENDPOINT
        deployment_name = settings.AZURE_OPENAI_DEPLOYMENT
        api_key = settings.AZURE_OPENAI_API_KEY
        api_version = settings.AZURE_OPENAI_API_VERSION

        headers = {
            "Content-Type": "application/json",
            "api-key": api_key,
        }

        payload = {
            "messages": [{"role": "user", "content": prompt}],
            "temperature": 0.7,
        }

        response = requests.post(
            f"{azure_endpoint}/openai/deployments/{deployment_name}/chat/completions?api-version={api_version}",
            headers=headers,
            json=payload,
        )

        if response.status_code != 200:
            return JsonResponse({"error": "OpenAI error", "details": response.json()}, status=500)

        result = response.json()
        reply = result["choices"][0]["message"]["content"]

        return JsonResponse({"reply": reply})
    except Exception as e:
        return JsonResponse({"error": "Server error", "details": str(e)}, status=500)



⸻

🔐 3. Configuration Settings

# settings.py
AZURE_OPENAI_API_KEY = os.getenv("AZURE_OPENAI_API_KEY")
AZURE_OPENAI_ENDPOINT = os.getenv("AZURE_OPENAI_ENDPOINT")  # Example: https://your-resource.openai.azure.com
AZURE_OPENAI_DEPLOYMENT = os.getenv("AZURE_OPENAI_DEPLOYMENT")  # Your model deployment name
AZURE_OPENAI_API_VERSION = os.getenv("AZURE_OPENAI_API_VERSION", "2023-12-01-preview")



⸻

🌐 4. URL Configuration

# aiassistant/urls.py
from django.urls import path
from .views import chat_with_ai

urlpatterns = [
    path("chat/", chat_with_ai, name="chat_with_ai"),
]

And include it in the project-level urls.py:

# project/urls.py
path("aiassistant/", include("aiassistant.urls")),



⸻

🔄 5. Update LMS Frontend Chat Widget

Instead of calling OpenAI directly, use:

fetch("/aiassistant/chat/", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    message: message,
    context: getCourseContext(),  // or course+unit info
  })
})
.then(res => res.json())
.then(data => {
  chatBox.innerHTML += `<div><b>AI:</b> ${data.reply}</div>`;
});



⸻

✅ Bonus: Add Rate Limiting or Logging

You can:
	•	Log messages to a model (ChatLog)
	•	Add throttling via Django REST Framework’s RateThrottle
	•	Require login and enforce permissions

⸻

💬 Summary
	•	You’ve now added a secure proxy between your LMS and Azure OpenAI.
	•	The LMS sends course-aware queries to /aiassistant/chat/.
	•	Django calls Azure OpenAI and returns the response.

⸻

Perfect! Tracking interactions is essential for analytics, debugging, and even fine-tuning AI behavior later on. Let’s define a simple but extensible ChatLog model to store:
	•	Who asked
	•	What the context was
	•	What question was asked
	•	What the AI replied
	•	When it happened

⸻

🧾 Step 1: Define ChatLog Model

File: aiassistant/models.py

from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()

class ChatLog(models.Model):
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
    context = models.TextField(blank=True)
    user_message = models.TextField()
    ai_reply = models.TextField()
    timestamp = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["-timestamp"]

    def __str__(self):
        return f"Chat from {self.user or 'Anonymous'} at {self.timestamp:%Y-%m-%d %H:%M}"



⸻

⚙️ Step 2: Run Migrations

python manage.py makemigrations aiassistant
python manage.py migrate



⸻

🔄 Step 3: Log Interactions in the View

Update views.py:

from .models import ChatLog

# Inside your view, after getting reply:
ChatLog.objects.create(
    user=request.user if request.user.is_authenticated else None,
    context=context,
    user_message=user_message,
    ai_reply=reply
)

✅ Tip: You may want to wrap this in a try/except block to avoid breaking the response if logging fails.

⸻

🔐 Optional Enhancements
	•	Restrict Logs to Staff in Django admin
	•	Add token usage & latency fields later for performance tracking
	•	Tag chats by course or lesson if you extract structured info from the context

⸻

✅ Bonus: Admin Interface

File: aiassistant/admin.py

from django.contrib import admin
from .models import ChatLog

@admin.register(ChatLog)
class ChatLogAdmin(admin.ModelAdmin):
    list_display = ("user", "timestamp", "user_message", "ai_reply")
    list_filter = ("timestamp", "user")
    search_fields = ("user_message", "ai_reply", "context")

Now you can view and search all chat logs in Django admin.

⸻

📊 Example Output in Admin

User	Timestamp	User Message	AI Reply
alice	2025-04-19 09:05 AM	“Explain for loops”	“In Python, for loops…”
anonymous	2025-04-19 09:10 AM	“Summarize this unit”	“This lesson covers…”



⸻

Would you like to export logs as CSV, filter by course/unit, or even show users their own past conversations in LMS?