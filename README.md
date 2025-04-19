# openedx

https://docs.tutor.edly.io/install.html#download
```
git clone https://github.com/overhangio/tutor
cd tutor
pip install -e .
```

Let’s break this down clearly to help you assess whether Overhang.io’s Open edX distribution (called Tutor) is the right foundation for deeply customizing and AI-enabling Open edX.

⸻

🔍 What is Overhang.io’s Tutor?

Tutor is an officially supported Docker-based deployment and management system for the Open edX platform. It is:
	•	Modular and plugin-based: You can easily enable/disable features using plugins.
	•	DevOps-friendly: Local development, staging, and production deployments are streamlined via Docker Compose or Kubernetes.
	•	Actively maintained: Tutor is endorsed by the Open edX community and kept up to date with the latest releases.
	•	Customizable: You can override templates, Python code, and configurations.

But yes, fundamentally it is a Docker-based implementation and orchestration layer for Open edX.

⸻

✅ When is Tutor the right choice?

Tutor is ideal if:
	1.	You need to set up and manage Open edX efficiently, without battling raw Ansible scripts.
	2.	You plan to make frontend/backend customizations, override templates, add plugins (including your own), or tweak system behavior.
	3.	You want to scale the platform (e.g., deploy to Kubernetes or AWS ECS).
	4.	You want to leverage its plugin system to add new functionality (e.g., LTI, analytics, SSO, AI services).

It simplifies the heavy lifting but gives you full access to the underlying platform.

⸻

❗ When it might not be enough (on its own)

If your goal is deep AI integration, such as:
	•	Personalized learning pathways with GenAI
	•	Intelligent tutor bots
	•	Adaptive assessments
	•	Auto-grading with NLP/ML
	•	Embedded LLM-based Q&A or feedback mechanisms

Then Tutor will be only the foundation. You’ll need to extend Open edX itself via:
	•	Custom XBlocks (for AI-enabled course components)
	•	LMS/Studio backend changes
	•	Custom plugins or Django apps
	•	Integrations via APIs or middleware services

And all of these are absolutely doable with Tutor, but Tutor itself doesn’t provide AI capabilities—it just gives you the environment to build them.

⸻

🚀 Summary

Aspect	Tutor (Overhang.io)
Purpose	Simplified deployment & customization
Based on	Docker / Docker Compose
AI features included	❌ None out of the box
Plugin system	✅ Yes, powerful and extendable
Ideal for	Custom Open edX with good DevOps practice
Good base for AI-enablement?	✅ Yes, but you build the AI layer



⸻

Great! Here’s a sample architecture diagram and explanation for building an AI-enabled Open edX platform using Tutor. This will help you visualize the big picture.

⸻

🧱 Architecture: AI-Enabled Open edX with Tutor

                     ┌────────────────────────────────────────┐
                     │            Learners / Instructors      │
                     │        (Web, Mobile, LTI, etc.)        │
                     └────────────────┬───────────────────────┘
                                      │
                         ┌────────────▼────────────┐
                         │       NGINX (Tutor)     │
                         └────────────┬────────────┘
                                      │
     ┌────────────────────────────────▼────────────────────────────────┐
     │                    Tutor-managed Open edX Services              │
     │                                                                │
     │  ┌────────────┬──────────────┬────────────┬────────────────┐  │
     │  │ LMS (edxapp)| CMS (Studio)│ Identity   │ Discovery      │  │
     │  │ Django app │ Django app  │ Provider   │ Course Search  │  │
     │  └────┬───────┴──────────────┴────────────┴──────┬─────────┘  │
     │       │                                           │            │
     │  ┌────▼──────────┐                      ┌─────────▼──────────┐│
     │  │  Plugin Layer │◄──────Custom AI─────►│   AI Services API  ││
     │  │ (Tutor + XBlocks│       Calls        │ (FastAPI / Django) ││
     │  └───────────────┘                      └─────────▲──────────┘│
     └───────────────────────────────────────────────────│───────────┘
                                                         │
             ┌────────────────────────────┬──────────────┘
             │                            │
 ┌───────────▼────────────┐   ┌───────────▼────────────┐
 │   GenAI Services       │   │    ML/NLP Model APIs   │
 │ (OpenAI, Claude, etc.) │   │ (Custom models via     │
 │                        │   │ HuggingFace / LangChain│
 └────────────────────────┘   └────────────────────────┘



⸻

🧩 Key Components Explained

1. Tutor-Based Open edX Core
	•	Runs the LMS, Studio, NGINX, MySQL/PostgreSQL, MongoDB, Redis via Docker.
	•	Extendable using Tutor plugins.
	•	You can override Django templates, settings, and frontend (MFEs).

2. Plugin Layer (Your AI Extensions)
	•	Build custom Tutor plugins or extend XBlocks to integrate:
	•	AI tutors
	•	Adaptive assessments
	•	Auto-grading
	•	Chatbot assistants
	•	These interact with your backend AI APIs.

3. AI Services API (FastAPI/Django REST)
	•	A microservice layer you build.
	•	Handles:
	•	Prompt formatting
	•	Calls to GenAI or ML models
	•	Caching and user-specific context
	•	Secure API endpoints

4. GenAI/ML Backend
	•	Integrate with:
	•	OpenAI / Claude / Gemini APIs
	•	HuggingFace Transformers for LLM/NLP
	•	LangChain for RAG (Retrieval-Augmented Generation)
	•	Custom-trained models for domain-specific tasks

5. User Interface (LMS/Studio)
	•	Frontend (React MFE or legacy templates) modified to:
	•	Inject AI-generated hints, feedback, or summaries
	•	Enable intelligent course navigation
	•	Display insights or learning analytics

⸻

🧠 AI Use Case Ideas for Open edX

Use Case	Integration Point	Tech Involved
Personalized tutoring	LMS → XBlock plugin	OpenAI, LangChain
Auto-graded essays/code	LMS → XBlock or submissions	HuggingFace, Python scripts
AI chatbot for students	LMS/Studio plugin + backend	RAG, LLM, FastAPI
Adaptive content sequencing	Backend → LMS logic injection	AI Rules engine
Instructor Q&A summarization	Studio → plugin/API layer	GPT/Claude
Learner engagement feedback	Backend → LMS dashboards	Analytics + NLP



⸻

🚀 Next Steps

If you’re serious about this route, here are some steps to begin:
	1.	Start with Tutor setup: Get a local deployment running.
	2.	Plan your AI APIs: Decide what AI use cases to prioritize.
	3.	Create a plugin skeleton using tutor plugins new your-plugin-name.
	4.	Develop and integrate AI services in Python (e.g., FastAPI for backend).
	5.	Test XBlocks or override templates to hook into LMS behavior.
	6.	Deploy to staging with Tutor’s Kubernetes or Docker Compose setup.

⸻


Great question! The Overhang.io Tutor project is a modular, Docker-based deployment system for Open edX, designed to simplify the often-complex installation and maintenance of Open edX platforms. Here’s a breakdown of how Tutor builds and orchestrates Open edX:

⸻

🏗️ How Tutor Builds from the Open edX Platform

1. Docker-Based Architecture

Tutor uses Docker and Docker Compose to containerize Open edX. It abstracts all the complexity into manageable services.
	•	Key containers include:
	•	lms, cms: Core Django apps of Open edX (LMS and Studio)
	•	mysql, mongodb, redis: Backend services
	•	nginx: Web server
	•	caddy: Reverse proxy with automatic HTTPS (used in Tutor’s newer stack)
	•	worker: Celery for asynchronous tasks

This means:
	•	No direct interaction with native OS packages.
	•	All dependencies are pinned and isolated in Docker images.

⸻

2. Open edX Source Code

Tutor uses the official Open edX repositories (e.g., edx-platform, cs_comments_service, xqueue, etc.).
	•	It clones or mounts these repositories into containers.
	•	Tutor supports multiple Open edX releases like Olive, Palm, Quince, etc.
	•	The version/tag is determined by the Tutor plugin or environment configuration (OPENEDX_RELEASE).

⸻

3. Image Building

Tutor builds its own Docker images for Open edX components using:
	•	tutor/docker-images
	•	These Dockerfiles define the base OS, install system dependencies, and clone/configure Open edX.

You can build with:

tutor images build all



⸻

4. Pluggable System (Plugins)

Tutor includes and supports many Open edX services via plugins:
	•	forum, discovery, ecommerce, mfe, etc.
	•	Plugins are Python packages that register custom Docker images, configs, and commands.

Example: the ecommerce plugin provides its own Docker build context and container orchestration.

You can install or enable plugins like:

tutor plugins enable ecommerce discovery mfe



⸻

5. Configuration System

Tutor uses:
	•	A YAML-based config system stored in $(tutor config printroot)/config.yml
	•	Python templates (Jinja2) to generate configs (env folder)
	•	All settings are customizable via CLI or .yml files.

Example:

tutor config save --interactive

will guide you through key settings like domain name, SMTP, OAuth, etc.

⸻

6. Provisioning and Initialization

After building and configuring, Tutor sets up and starts Open edX with:

tutor local quickstart

This does:
	•	Create databases (MySQL, Mongo)
	•	Run Django migrations
	•	Collect static files
	•	Start all services via Docker Compose

⸻

7. Custom Code and Development

Tutor supports:
	•	Mounting custom code via tutor dev mount
	•	Development workflows via tutor dev run and tutor dev start
	•	CI/CD setups for custom plugins, themes, or Docker builds

⸻

🧩 Summary Diagram (Conceptual)

            +----------------+
            |  tutor config  |
            +----------------+
                     |
       +-------------+-------------+
       |                           |
+---------------+         +-----------------+
| tutor plugins |         | openedx releases|
+---------------+         +-----------------+
       |                           |
       +-------------+-------------+
                     |
            +----------------+
            | docker-compose |
            +----------------+
                     |
         +--------------------------+
         |   LMS / CMS / Other svc  |
         +--------------------------+
         | DB, Redis, Elasticsearch |
         +--------------------------+



⸻

💬 Key Advantages of Tutor
	•	🧩 Modular: Easy to extend with plugins.
	•	🐳 Containerized: Full isolation from host OS.
	•	🧼 Simplified DevOps: One command (tutor local quickstart) to get a full Open edX stack.
	•	🚀 Quick Deployments: Easy to update, maintain, and version.

⸻

Let’s walk through two powerful Tutor customizations:
	1.	✅ Creating a Custom Tutor Plugin
	2.	📂 Mounting Your Own Fork of edx-platform into Tutor

⸻

1️⃣ Creating a Custom Tutor Plugin

Tutor plugins are Python packages that register custom Dockerfiles, configuration values, hooks, and commands.

🔧 Step-by-Step: Create a Plugin

✅ a. Scaffold a plugin:

tutor plugins new myplugin

This generates:

plugins/
└── myplugin/
    ├── __init__.py
    ├── templates/
    └── patches/

✅ b. Define the plugin in __init__.py:

from tutor import hooks

hooks.Filters.ENV_PATCHES.add_items([
    ("lms-env", "myplugin/lms-env"),
])

✅ c. Add a config template in templates/myplugin/lms-env:

"MY_CUSTOM_SETTING": "{{ MYPLUGIN_MY_CUSTOM_SETTING }}"

✅ d. Add a default setting in __init__.py:

hooks.Filters.CONFIG_DEFAULTS.add_items([
    ("MYPLUGIN_MY_CUSTOM_SETTING", "default-value"),
])

✅ e. Enable your plugin:

tutor plugins enable myplugin

Now tutor config save will include your new setting!

⸻

2️⃣ Mounting Your Own Fork of edx-platform

This is useful for:
	•	Patching LMS/CMS views
	•	Developing custom XBlocks
	•	Experimenting with upstream features

📦 Step-by-Step: Mount Your Own Code

✅ a. Clone Open edX fork:

git clone https://github.com/your-org/edx-platform.git
cd edx-platform

✅ b. Enable dev mode in Tutor:

tutor dev run lms bash

✅ c. Mount your local fork into the container:

tutor dev mount lms ./edx-platform
tutor dev mount cms ./edx-platform

This replaces /openedx/edx-platform inside LMS and CMS containers.

✅ d. Restart services in dev mode:

tutor dev start

Now you’re running your own code inside the Open edX LMS/CMS!

⚠️ Note:

You may need to re-run:

tutor dev run lms pip install -r requirements/edx/development.txt
tutor dev run lms python manage.py lms migrate



⸻

🧪 Optional: Hot-Reload with Code Watcher

Tutor supports live code reloading (limited for backend code):

tutor dev start --mount



⸻

🔁 Summary: When to Use What?

Goal	Use Custom Plugin	Mount edx-platform
Add a config value	✅ Yes	❌ No
Patch LMS templates	✅ Yes	✅ Yes
Change LMS backend logic	❌ Not directly	✅ Yes
Build reusable extensions	✅ Yes	❌ No
Run a forked Open edX	❌ No	✅ Yes



⸻


