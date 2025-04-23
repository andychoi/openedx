Excellent — integrating an external Identity Provider (IdP) like Azure AD B2C into a Tutor-based Open edX system adds a secure and scalable authentication layer. Below is your updated architecture and a detailed integration flow that includes Azure AD B2C as the IdP.

⸻

🧱 Updated Architecture: Tutor-based Open edX + Azure AD B2C + AI Services

                     ┌────────────────────────────────────────┐
                     │        Learners / Instructors          │
                     │         (Web, Mobile, LTI, etc.)       │
                     └────────────────────┬───────────────────┘
                                          │
                                 ┌────────▼─────────┐
                                 │   Azure AD B2C   │◄───────────────┐
                                 │ (External IdP)   │                │
                                 └────────┬─────────┘                │
                                          │                          │
                     ┌────────────────────▼────────────────────┐     │
                     │         NGINX / OIDC Middleware          │     │
                     └────────────────┬─────────────────────────┘     │
                                      │                               │
 ┌────────────────────────────────────▼────────────────────────────────────────┐
 │                     Tutor-based Open edX Services (Docker)                  │
 │                                                                             │
 │ LMS (edxapp)       CMS (Studio)      Discovery     Identity Provider (IDP) │
 │  Django app        Django app        Search        (OIDC enabled)          │
 │       │                                                        ▲           │
 │       └──────┬──────────────────────────────────────────────────┘           │
 │              │                                                            │
 │      ┌───────▼─────────┐        ┌──────────────────────┐                 │
 │      │ Custom Plugins  │◄─────►│   AI Services API      │                 │
 │      │ (Tutor, XBlock) │       │ (FastAPI / LangChain) │                 │
 │      └─────────────────┘       └────────────────────────┘                 │
 └───────────────────────────────────────────────────────────────────────────┘

                          │
      ┌───────────────────▼──────────────────────────────┐
      │ GenAI APIs (e.g., OpenAI, Claude, Gemini, etc.)  │
      └──────────────────────────────────────────────────┘



⸻

🔐 Authentication & SSO Integration Flow (Azure AD B2C)

🌐 1. User Authentication via Azure AD B2C
	•	Learner/instructor clicks “Sign in” in Open edX LMS.
	•	LMS redirects to Azure AD B2C’s login page.
	•	User authenticates with email, password, MFA, or any configured method.
	•	B2C returns an ID Token + Access Token via OIDC protocol.

🔄 2. Tutor OIDC Plugin Handles Token Exchange
	•	Open edX receives the tokens via OIDC client (enabled by plugin).
	•	It extracts user info (sub, email, name, etc.).
	•	If the user doesn’t exist, Open edX creates a new user.
	•	Subsequent logins match users by the OIDC subject or email.

⸻

🧩 Required Components for OIDC Integration

✅ 1. Tutor OIDC Plugin

Install the official Tutor plugin:

tutor plugins enable oauth

🔧 2. Configure OIDC Settings in Tutor

Add these settings to your Tutor configuration:

tutor config save --set OAUTH_OIDC_ISSUER=https://<your-b2c-tenant>.b2clogin.com/<tenant-id>/v2.0/
tutor config save --set OAUTH_OIDC_CLIENT_ID=<client-id>
tutor config save --set OAUTH_OIDC_CLIENT_SECRET=<client-secret>
tutor config save --set OAUTH_OIDC_LOGOUT_URL=https://<your-b2c-tenant>.b2clogin.com/logout

Use your Azure B2C custom policy or user flow endpoint as the issuer.

You can also define user mapping functions (e.g., map_claims_to_user) if needed.

⸻

🔁 End-to-End Login Flow
	1.	Open edX (via LMS or Studio) uses OIDC to redirect to Azure AD B2C.
	2.	Azure B2C authenticates and returns tokens.
	3.	Tutor’s LMS plugin validates token → extracts claims.
	4.	If user doesn’t exist → new user is provisioned.
	5.	User is logged into Open edX and redirected to the LMS or Studio.
	6.	Custom LMS features (e.g., AI tutor chat, dashboards) now know the user and can personalize responses using AI.

⸻

🧠 Optional Enhancements

🎓 Role Mapping
	•	Use Azure AD B2C group claims to assign roles like staff, instructor, or admin automatically.
	•	Customize OIDC_CLAIM_MAPPING in Tutor.

🔗 Linking Existing Accounts
	•	If your users already have Open edX accounts, consider implementing account linking logic on first OIDC login.

📊 AI Personalization Context
	•	Pass user metadata (email, course list, progress, goals) to your AI API for:
	•	Tailored study plans
	•	Contextual feedback
	•	Adaptive learning paths

⸻

✅ Summary

Component	Tool / Tech
Platform Base	Tutor (Docker-based Open edX)
External IdP	Azure AD B2C (OIDC)
Integration Plugin	Tutor OAuth plugin (oauth)
AI Extension Layer	Custom APIs (FastAPI + LLMs)
Deployment	Docker Compose / Kubernetes



⸻

Here’s a working YAML-based configuration example to integrate Azure AD B2C with Tutor via OpenID Connect (OIDC). This configuration should be placed inside your Tutor environment using either config.yml overrides or environment variables via .env or plugin-specific mechanisms.

⸻

✅ Step-by-Step YAML Configuration for Azure AD B2C

🗂️ 1. Enable the OAuth plugin

tutor plugins enable oauth

🛠️ 2. Update Tutor config.yml or set with CLI

You can either add these to $(tutor config printroot)/config.yml, or run them directly with tutor config save --set.

Here’s the YAML snippet for your config.yml:

OAUTH_OIDC_ISSUER: "https://<your-tenant-name>.b2clogin.com/<your-tenant-id>/v2.0/"
OAUTH_OIDC_CLIENT_ID: "<your-client-id>"
OAUTH_OIDC_CLIENT_SECRET: "<your-client-secret>"
OAUTH_OIDC_LOGOUT_URL: "https://<your-tenant-name>.b2clogin.com/<your-tenant-id>/oauth2/v2.0/logout"

# Optional mappings for user identity
OAUTH_OIDC_USERNAME_CLAIM: "email"
OAUTH_OIDC_EMAIL_CLAIM: "email"
OAUTH_OIDC_FIRST_NAME_CLAIM: "given_name"
OAUTH_OIDC_LAST_NAME_CLAIM: "family_name"

# Friendly name displayed on login page
OAUTH_OIDC_NAME: "Login with Azure AD B2C"

Replace the placeholders with values from your Azure AD B2C App Registration:

Placeholder	Description
<your-tenant-name>	Your B2C tenant name (e.g., contoso)
<your-tenant-id>	Directory (tenant) ID
<your-client-id>	Application (client) ID
<your-client-secret>	Client secret value from app registration

You can also set these via CLI:

tutor config save \
  --set OAUTH_OIDC_ISSUER="https://<tenant>.b2clogin.com/<tenant-id>/v2.0/" \
  --set OAUTH_OIDC_CLIENT_ID="<client-id>" \
  --set OAUTH_OIDC_CLIENT_SECRET="<client-secret>" \
  --set OAUTH_OIDC_LOGOUT_URL="https://<tenant>.b2clogin.com/<tenant-id>/oauth2/v2.0/logout"



⸻

🔐 Azure B2C Notes
	•	Use user flows or custom policies (recommended) to define the login experience.
	•	If using a user flow like B2C_1_signupsignin1, your OAUTH_OIDC_ISSUER may look like:

OAUTH_OIDC_ISSUER: "https://<tenant>.b2clogin.com/<tenant>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/"

	•	Make sure your Azure app registration includes:
	•	Redirect URIs (e.g., https://yourdomain.com/auth/complete/azuread-oauth2/)
	•	ID token and Access token options enabled under Token Configuration
	•	Claims like email, given_name, family_name under User Attributes

⸻

🔄 Apply Config and Restart

After updating your config:

tutor config save
tutor local launch



⸻

Here’s an example FastAPI AI backend structure that integrates with your Tutor-based Open edX system, reads user identity via a token (e.g. from Azure AD B2C or LMS), and provides personalized tutoring using an LLM like OpenAI or Claude.

⸻

📦 Project Structure

ai-backend/
├── main.py
├── auth.py
├── routers/
│   └── tutor.py
├── services/
│   └── llm.py
├── models/
│   └── user_context.py
├── requirements.txt
└── .env



⸻

🧠 Overview of Core Components

1. main.py – App entry point

from fastapi import FastAPI
from routers import tutor

app = FastAPI(title="AI Tutor API")

app.include_router(tutor.router, prefix="/tutor", tags=["AI Tutor"])



⸻

2. auth.py – Identity parsing from JWT/OIDC token

from fastapi import Depends, HTTPException, Header
from jose import jwt, JWTError
from pydantic import BaseModel
import os

# Azure B2C config from env or Tutor settings
OIDC_ISSUER = os.getenv("OIDC_ISSUER")
OIDC_CLIENT_ID = os.getenv("OIDC_CLIENT_ID")

class User(BaseModel):
    username: str
    email: str
    name: str = ""
    roles: list[str] = []

def get_current_user(authorization: str = Header(...)) -> User:
    try:
        token = authorization.replace("Bearer ", "")
        payload = jwt.decode(token, options={"verify_signature": False})  # For demo; verify in prod
        return User(
            username=payload.get("sub"),
            email=payload.get("email"),
            name=payload.get("name", ""),
            roles=payload.get("roles", []),
        )
    except JWTError as e:
        raise HTTPException(status_code=401, detail="Invalid token")



⸻

3. routers/tutor.py – AI Tutor endpoint

from fastapi import APIRouter, Depends, Request
from auth import get_current_user, User
from services.llm import get_personalized_help

router = APIRouter()

@router.post("/assist")
async def tutor_assist(request: Request, user: User = Depends(get_current_user)):
    body = await request.json()
    question = body.get("question")

    if not question:
        return {"error": "Missing question"}

    # Use AI model with user context
    response = get_personalized_help(user=user, question=question)
    return {"reply": response}



⸻

4. services/llm.py – AI logic using OpenAI (can use Claude, Gemini, etc.)

import openai
from auth import User

openai.api_key = os.getenv("OPENAI_API_KEY")

def get_personalized_help(user: User, question: str) -> str:
    prompt = f"""
    You are a helpful AI tutor. The student is:
    Name: {user.name}
    Email: {user.email}
    Roles: {', '.join(user.roles)}
    
    The student asks: {question}
    Give a clear, contextual answer.
    """
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.7,
    )
    return response.choices[0].message["content"]



⸻

5. models/user_context.py (optional reusable user info)

from pydantic import BaseModel

class UserContext(BaseModel):
    email: str
    name: str
    enrolled_courses: list[str] = []
    current_course: str = ""
    roles: list[str] = []



⸻

6. .env Example

OIDC_ISSUER=https://your-b2c.b2clogin.com/tenant-id/v2.0/
OIDC_CLIENT_ID=abc123
OPENAI_API_KEY=sk-xxx



⸻

🔐 Token Passing from Open edX
	•	In Tutor, pass the JWT to your backend via:
	•	LMS XBlock calling backend with Authorization: Bearer <token>
	•	Custom plugin making API calls from the frontend (React MFE)
	•	Tokens can be fetched via LMS login, or passed securely via session/cookie if using a proxy.

⸻

🧪 Test Example (cURL)

curl -X POST https://your-backend.example.com/tutor/assist \
  -H "Authorization: Bearer <user_token>" \
  -H "Content-Type: application/json" \
  -d '{"question": "Can you explain how to solve quadratic equations?"}'



⸻

🚀 What’s Next?
	•	You can enhance context with course progress, learning goals, or even past answers.
	•	Add support for retrieval-augmented generation (RAG) to include course content from MongoDB.
	•	Implement rate limiting, logging, and feedback collection per student.

⸻

Dockerfile + Docker Compose for deploying this backend alongside Tutor

Awesome! Here’s a complete setup to run your AI Tutor FastAPI backend alongside Tutor using Docker Compose.

⸻

🐳 Step-by-Step: Docker + Docker Compose Setup

📁 Final Folder Structure

ai-backend/
├── app/
│   ├── main.py
│   ├── auth.py
│   ├── routers/
│   │   └── tutor.py
│   ├── services/
│   │   └── llm.py
│   ├── models/
│   │   └── user_context.py
│   └── requirements.txt
├── .env
├── Dockerfile
└── docker-compose.yml



⸻

🐍 app/requirements.txt

fastapi
uvicorn[standard]
python-jose
pydantic
openai
python-dotenv



⸻

🐋 Dockerfile

FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install dependencies
COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app files
COPY app .

# Expose port
EXPOSE 8000

# Run FastAPI with uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]



⸻

⚙️ .env (in project root)

OPENAI_API_KEY=sk-xxx
OIDC_ISSUER=https://your-b2c.b2clogin.com/tenant-id/v2.0/
OIDC_CLIENT_ID=abc123



⸻

🧱 docker-compose.yml

Here’s how to run it alongside Tutor:

version: "3.8"

services:
  ai-backend:
    build: .
    container_name: ai-tutor-api
    env_file: .env
    ports:
      - "8001:8000"  # Accessible at http://localhost:8001
    networks:
      - tutor_default
    restart: unless-stopped

networks:
  tutor_default:
    external: true

Note: tutor_default is the default network Tutor uses when you run tutor local start. You can verify with docker network ls.

⸻

🔗 Integrate with Tutor LMS/XBlock/Frontend

From your LMS/XBlock frontend, send requests to:

http://ai-backend:8000/tutor/assist

OR from local browser:

http://localhost:8001/tutor/assist

With headers:

Authorization: Bearer <OIDC token>
Content-Type: application/json



⸻

✅ Start the system

Run from the ai-backend folder:

docker-compose up --build -d



⸻

🛡️ Add Security Tips
	•	In production, use HTTPS + reverse proxy (NGINX).
	•	Enable token validation with Azure B2C public keys (currently skipped for demo).
	•	Add logging, rate limiting, and token expiry checks.

⸻

A sample XBlock or MFE React component that calls this AI tutor backend from within the LMS

Here’s a minimal working example of both:

⸻

📦 Option 1: Minimal XBlock (Python Backend) to call the AI Tutor API

This example creates a custom XBlock in Open edX that sends a learner’s question to the FastAPI AI Tutor backend and shows the AI’s reply.

📁 File: ai_tutor/ai_tutor.py

from xblock.core import XBlock
from xblock.fields import String, Scope
from xblock.fragment import Fragment
import requests

class AITutorXBlock(XBlock):
    question = String(default="", scope=Scope.user_state, help="User's question")
    answer = String(default="", scope=Scope.user_state, help="AI Tutor reply")

    def student_view(self, context=None):
        html = f"""
        <div>
            <input id="ai-question" type="text" placeholder="Ask a question..." />
            <button onclick="submitQuestion()">Ask</button>
            <p><strong>Answer:</strong> <span id="ai-answer">{self.answer}</span></p>
        </div>
        <script>
        function submitQuestion() {{
            const question = document.getElementById('ai-question').value;
            fetch('/ai_tutor/ask', {{
                method: 'POST',
                headers: {{ 'Content-Type': 'application/json' }},
                body: JSON.stringify({{ question }})
            }})
            .then(res => res.json())
            .then(data => {{
                document.getElementById('ai-answer').innerText = data.reply;
            }});
        }}
        </script>
        """
        return Fragment(html)

    @XBlock.json_handler
    def ask(self, data, suffix=''):
        token = self.runtime.service(self, 'user').get_current_user().opt_attrs.get('access_token', '')
        question = data.get("question", "")
        response = requests.post(
            "http://ai-backend:8000/tutor/assist",
            headers={"Authorization": f"Bearer {token}", "Content-Type": "application/json"},
            json={"question": question}
        )
        if response.ok:
            self.answer = response.json()["reply"]
        else:
            self.answer = "Error contacting AI tutor."
        return {"reply": self.answer}



⸻

⚛️ Option 2: Minimal React MFE Component for Tutor

This works if you’re extending Tutor with a Micro Frontend (MFE).

📝 AITutor.jsx

import React, { useState } from "react";

export const AITutor = () => {
  const [question, setQuestion] = useState("");
  const [answer, setAnswer] = useState("");

  const handleAsk = async () => {
    const token = await getUserJwt();  // You need to implement this securely
    const res = await fetch("http://localhost:8001/tutor/assist", {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${token}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ question })
    });
    const data = await res.json();
    setAnswer(data.reply);
  };

  return (
    <div className="ai-tutor-box">
      <h3>Ask the AI Tutor</h3>
      <input
        type="text"
        value={question}
        onChange={(e) => setQuestion(e.target.value)}
        placeholder="Ask a question about your course..."
      />
      <button onClick={handleAsk}>Ask</button>
      {answer && <p><strong>Answer:</strong> {answer}</p>}
    </div>
  );
};



⸻

🛠️ How to Securely Get the JWT Token

In LMS:
	•	Use getUserJwt() from Open edX auth utilities (if inside MFE).
	•	Or pass the token from Django template into JS context if custom HTML.

In XBlock:
	•	Use self.runtime.service(self, 'user') as shown to fetch the token.

⸻

Perfect — here are the complete, step-by-step instructions to install and use your custom XBlock (e.g. AITutorXBlock) in a Tutor-based Open edX instance.

⸻

✅ Step 1: Create the XBlock Package

Create your XBlock as a Python package.

📁 Folder Structure

ai-tutor-xblock/
├── ai_tutor/
│   ├── __init__.py
│   └── ai_tutor.py  # Your XBlock class from earlier
├── setup.py
└── README.md

📄 setup.py

from setuptools import setup

setup(
    name='ai-tutor-xblock',
    version='0.1',
    description='AI Tutor XBlock for Open edX',
    packages=['ai_tutor'],
    install_requires=[
        'XBlock',
        'requests'
    ],
    entry_points={
        'xblock.v1': [
            'ai_tutor = ai_tutor.ai_tutor:AITutorXBlock',
        ]
    },
)

✅ You can test it locally with:

pip install -e .



⸻

✅ Step 2: Build a Tutor Plugin to Install the XBlock

Create a Tutor plugin to automatically install this XBlock inside the LMS container.

🔧 Generate a plugin scaffold

tutor plugins new ai-xblock
cd "$(tutor plugins printroot)/ai-xblock"

✏️ Edit ai-xblock/plugin.py

Update patch_lms_envs to include the XBlock:

from tutor import hooks

hooks.Filters.ENV_PATCHES.add_items([
    ("lms-env", {
        "ADDITIONAL_XBLOCKS": ["ai_tutor.ai_tutor.AITutorXBlock"]
    }),
])

📁 Create a folder for requirements

mkdir patches

Add this line to a file named patches/requirements.txt:

/mnt/hosted-xblocks/ai-tutor-xblock

🛠️ Mount the XBlock source code into the LMS container

Update patches/docker-compose.override.yml:

services:
  lms:
    volumes:
      - ./mnt/hosted-xblocks:/mnt/hosted-xblocks

Put your ai-tutor-xblock package inside mnt/hosted-xblocks.

⸻

✅ Step 3: Enable and Apply the Plugin
	1.	Move the source code into place:

mkdir -p "$(tutor plugins printroot)/ai-xblock/mnt/hosted-xblocks"
cp -r ~/ai-tutor-xblock "$(tutor plugins printroot)/ai-xblock/mnt/hosted-xblocks/"


	2.	Enable and rebuild:

tutor plugins enable ai-xblock
tutor config save
tutor images build openedx
tutor local start -d



⸻

✅ Step 4: Enable the XBlock in Studio
	1.	Go to Studio (/admin or /admin/xblock_django/xblockconfiguration/).
	2.	Click “Add XBlock Configuration”.
	3.	Add ai_tutor to the list of enabled XBlocks.

Or in Studio:
	•	Create a course
	•	Go to “Advanced Settings”
	•	Add "ai_tutor" to advanced_modules.

⸻

✅ Step 5: Use the XBlock in a Course
	1.	In Studio, open a unit.
	2.	Click “Add New Component” > “Advanced”.
	3.	Select “AI Tutor” (or your custom name).
	4.	Use the component to test your integration.

⸻

🧠 Notes
	•	You can live-reload XBlock changes by restarting only LMS:

tutor local run lms bash
# Inside container:
pkill -HUP gunicorn


	•	If you want the XBlock HTML or JS to be more advanced, consider:
	•	Loading templates via self.resource_string()
	•	Using RequireJS or React via fragment assets

⸻

Would you like me to send a zip of this plugin and XBlock package or help turn it into a GitHub repo template?