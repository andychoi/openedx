Awesome! Combining BM25 (keyword-based search) with vector similarity (embedding-based search) gives you a hybrid search engine that balances precision (exact match) and semantic relevance — best of both worlds. This is ideal for educational platforms like Open edX where search needs to handle exact curriculum keywords and conceptual understanding.

⸻

🎯 Goal: Hybrid Search = BM25 + Embedding (Vector) Search

Component	Description
🔍 BM25	Lexical search (exact term match) using e.g. Whoosh, Elastic, or Lucene
🧠 Vector	Embedding search using FAISS, Qdrant, etc.
⚖️ Scorer	Combine both scores into a unified ranking



⸻

🛠️ Step-by-Step: Build Hybrid Search

1️⃣ Prepare the Document Corpus

Assume each Open edX unit is extracted like:

{
  "id": "unit-id-1",
  "title": "Photosynthesis",
  "content": "Photosynthesis is the process by which plants convert light energy..."
}

Store these in documents.json.

⸻

2️⃣ Set Up BM25 (via Whoosh)

pip install whoosh

✅ Index the documents:

from whoosh.index import create_in
from whoosh.fields import Schema, TEXT, ID
import os, json

schema = Schema(id=ID(stored=True), title=TEXT(stored=True), content=TEXT)
os.makedirs("bm25_index", exist_ok=True)
ix = create_in("bm25_index", schema)

writer = ix.writer()
with open("documents.json") as f:
    docs = json.load(f)

for doc in docs:
    writer.add_document(id=doc["id"], title=doc["title"], content=doc["content"])
writer.commit()



⸻

3️⃣ Set Up Vector Index (FAISS)

pip install faiss-cpu openai langchain

from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.schema import Document

embedding_model = OpenAIEmbeddings()

vector_docs = [Document(page_content=d["content"], metadata={"id": d["id"], "title": d["title"]}) for d in docs]
faiss_index = FAISS.from_documents(vector_docs, embedding_model)
faiss_index.save_local("vector_index")



⸻

4️⃣ Combine Both in a Hybrid Search Function

from whoosh.qparser import QueryParser
from whoosh.index import open_dir
from langchain.vectorstores import FAISS
from sklearn.preprocessing import MinMaxScaler
import numpy as np

def hybrid_search(query, k=5):
    # === BM25 ===
    ix = open_dir("bm25_index")
    bm25_scores = {}
    with ix.searcher() as searcher:
        qp = QueryParser("content", schema=ix.schema)
        q = qp.parse(query)
        results = searcher.search(q, limit=k*2)
        for i, hit in enumerate(results):
            bm25_scores[hit["id"]] = (k - i)  # Simple rank-based score

    # === Vector ===
    faiss_index = FAISS.load_local("vector_index", embedding_model)
    vector_results = faiss_index.similarity_search_with_score(query, k=k*2)
    vector_scores = {doc.metadata["id"]: score for doc, score in vector_results}

    # === Normalize scores ===
    all_ids = set(bm25_scores) | set(vector_scores)
    bm25_vals = np.array([bm25_scores.get(i, 0) for i in all_ids]).reshape(-1, 1)
    vec_vals = np.array([vector_scores.get(i, 0) for i in all_ids]).reshape(-1, 1)

    scaler = MinMaxScaler()
    bm25_norm = scaler.fit_transform(bm25_vals)
    vec_norm = scaler.fit_transform(vec_vals)

    hybrid_scores = {
        doc_id: float(0.5 * b + 0.5 * v)
        for doc_id, b, v in zip(all_ids, bm25_norm, vec_norm)
    }

    top_results = sorted(hybrid_scores.items(), key=lambda x: x[1], reverse=True)[:k]
    return [(doc_id, score) for doc_id, score in top_results]



⸻

✅ Example Usage

results = hybrid_search("How do plants convert sunlight?", k=5)
for doc_id, score in results:
    print(f"{doc_id}: {score:.4f}")



⸻

📈 Summary of Benefits

Feature	BM25	Embedding Search	Hybrid
Keyword Match	✅ Strong	❌ Weak	✅ Preserved
Semantic Relevance	❌ Limited	✅ Strong	✅ Best of both
Precision	High	Medium	High
Recall	Low	High	High



⸻

🔮 Bonus: Tweak Score Weights

You can modify:

0.5 * bm25 + 0.5 * vector  →  0.7 * bm25 + 0.3 * vector

To bias the hybrid engine based on performance benchmarks or use case (e.g., quiz lookup vs conceptual questions).

⸻

🧠 Optional Enhancements
	•	🧾 Add metadata filtering (e.g., course_id, unit_type)
	•	🧪 Run benchmark evaluation (Precision@k, Recall@k, MRR)
	•	🌐 Expose hybrid search via a Django REST endpoint
	•	🎓 Tutor plugin UI to show hybrid results inside LMS/Studio

⸻

Would you like:
	•	A Django API wrapper to serve hybrid search?
	•	A web UI for manual testing?
	•	Pre-indexing script for Open edX content + benchmark tooling?

Let me know what you’d like to build next!

Great! Let’s now wrap the hybrid search engine in a Django REST API so it can serve search results to:
	•	📡 LMS chatbots
	•	🧠 Content authoring plugins
	•	🔍 Search bars inside Open edX (Tutor) Studio or LMS
	•	🛠 Internal tools for curriculum designers

⸻

🧰 Summary of What We’ll Build

Part	Description
🔁 Endpoint	POST /hybridsearch/search/
📥 Input	JSON with query, optional top_k
📤 Output	Ranked results with doc IDs and titles
🧠 Backend	Combines BM25 + FAISS-based vector search



⸻

🏗️ Step-by-Step Implementation

1️⃣ Django App Setup

python manage.py startapp hybridsearch

Add to INSTALLED_APPS:

INSTALLED_APPS = [
    ...
    'hybridsearch',
]



⸻

2️⃣ Define the API View

hybridsearch/views.py

import json
import numpy as np
from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse, HttpResponseBadRequest
from whoosh.index import open_dir
from whoosh.qparser import QueryParser
from sklearn.preprocessing import MinMaxScaler
from langchain.vectorstores import FAISS
from langchain.embeddings.openai import OpenAIEmbeddings

@csrf_exempt
def hybrid_search_view(request):
    if request.method != "POST":
        return HttpResponseBadRequest("Only POST allowed")

    try:
        body = json.loads(request.body)
        query = body.get("query")
        top_k = int(body.get("top_k", 5))
        if not query:
            return JsonResponse({"error": "Query required"}, status=400)

        # === BM25 ===
        ix = open_dir("bm25_index")
        bm25_scores = {}
        with ix.searcher() as searcher:
            qp = QueryParser("content", schema=ix.schema)
            q = qp.parse(query)
            results = searcher.search(q, limit=top_k*2)
            for i, hit in enumerate(results):
                bm25_scores[hit["id"]] = (top_k - i)

        # === Vector ===
        embedding_model = OpenAIEmbeddings()
        faiss_index = FAISS.load_local("vector_index", embedding_model)
        vector_results = faiss_index.similarity_search_with_score(query, k=top_k*2)
        vector_scores = {doc.metadata["id"]: score for doc, score in vector_results}
        vector_titles = {doc.metadata["id"]: doc.metadata["title"] for doc, _ in vector_results}

        # === Merge & Score ===
        all_ids = list(set(bm25_scores) | set(vector_scores))
        bm25_vals = np.array([bm25_scores.get(i, 0) for i in all_ids]).reshape(-1, 1)
        vec_vals = np.array([vector_scores.get(i, 0) for i in all_ids]).reshape(-1, 1)

        scaler = MinMaxScaler()
        bm25_norm = scaler.fit_transform(bm25_vals)
        vec_norm = scaler.fit_transform(vec_vals)

        hybrid_scores = {
            doc_id: float(0.5 * b + 0.5 * v)
            for doc_id, b, v in zip(all_ids, bm25_norm, vec_norm)
        }

        ranked = sorted(hybrid_scores.items(), key=lambda x: x[1], reverse=True)[:top_k]

        return JsonResponse({
            "query": query,
            "results": [
                {
                    "id": doc_id,
                    "score": score,
                    "title": vector_titles.get(doc_id, "")
                }
                for doc_id, score in ranked
            ]
        })

    except Exception as e:
        return JsonResponse({"error": "Internal error", "details": str(e)}, status=500)



⸻

3️⃣ Wire Up the URL

hybridsearch/urls.py

from django.urls import path
from .views import hybrid_search_view

urlpatterns = [
    path("search/", hybrid_search_view, name="hybrid_search")
]

Add to root urls.py:

path("hybridsearch/", include("hybridsearch.urls")),



⸻

4️⃣ Sample Request (Frontend / Postman)

POST /hybridsearch/search/
Content-Type: application/json

{
  "query": "Explain what DNA is",
  "top_k": 5
}

Sample Response:

{
  "query": "Explain what DNA is",
  "results": [
    {
      "id": "unit-id-001",
      "score": 0.928,
      "title": "Introduction to DNA"
    },
    {
      "id": "unit-id-007",
      "score": 0.873,
      "title": "Molecular Structure"
    }
  ]
}



⸻

🔐 Security & Optimization

Feature	Tip
🔐 Authentication	Require request.user.is_authenticated or API token
🧪 Caching	Cache vector results for repeated queries
📄 Logging	Store query logs for future benchmarks
🎯 Filtering	Filter by course ID or tag in metadata if needed
📦 Deployment	Use gunicorn + nginx for production use



⸻

✅ What You Now Have
	•	A fully functional Django API for hybrid search
	•	Backed by BM25 + FAISS
	•	Easily connected to Tutor plugins, LMS widgets, or chatbots

⸻

Would you like next:
	•	📊 Admin panel to review search logs?
	•	🔍 Add filters like course_id, unit_type, etc.?
	•	🌐 A web interface to test hybrid search visually?

Let me know, and I’ll help you build it!

Perfect! Adding filters like course_id, unit_type, or even tags allows you to narrow hybrid search results contextually, which is critical in Open edX–based authoring tools or AI assistants.

Let’s enhance your hybrid search API to support filters like:
	•	course_id — match specific course
	•	unit_type — video, html, discussion, problem, etc.
	•	tags — optional keywords like “biology”, “intro”, etc.

⸻

🎯 Updated Goal

Allow this kind of request:

{
  "query": "How do plants make food?",
  "course_id": "course-v1:Bio+101+2024",
  "unit_type": "html",
  "tags": ["photosynthesis"]
}

And return filtered hybrid search results.

⸻

🧰 Step-by-Step Enhancement

1️⃣ Update Your Vector Document Index

When building your FAISS vector index, add extra metadata per unit:

vector_docs = [
    Document(
        page_content=unit["content"],
        metadata={
            "id": unit["id"],
            "title": unit["title"],
            "course_id": unit["course_id"],
            "unit_type": unit.get("type", "html"),
            "tags": unit.get("tags", [])
        }
    )
    for unit in units
]

Rebuild your FAISS index with enriched metadata:

faiss_index = FAISS.from_documents(vector_docs, embedding_model)
faiss_index.save_local("vector_index")



⸻

2️⃣ Modify the Hybrid Search View for Filtering

Update views.py:

def hybrid_search_view(request):
    ...
    course_id = body.get("course_id")
    unit_type = body.get("unit_type")
    filter_tags = body.get("tags", [])

    ...
    # After computing `ranked` results
    filtered_ranked = []
    for doc_id, score in ranked:
        metadata = faiss_index.docstore._dict[doc_id].metadata
        if course_id and metadata.get("course_id") != course_id:
            continue
        if unit_type and metadata.get("unit_type") != unit_type:
            continue
        if filter_tags:
            doc_tags = metadata.get("tags", [])
            if not any(tag in doc_tags for tag in filter_tags):
                continue
        filtered_ranked.append((doc_id, score, metadata))
        if len(filtered_ranked) >= top_k:
            break

    return JsonResponse({
        "query": query,
        "results": [
            {
                "id": doc_id,
                "score": score,
                "title": meta.get("title", ""),
                "course_id": meta.get("course_id"),
                "unit_type": meta.get("unit_type"),
                "tags": meta.get("tags", [])
            }
            for doc_id, score, meta in filtered_ranked
        ]
    })



⸻

✅ Supported Filter Logic

Filter	Type	Matching Rule
course_id	Exact	metadata["course_id"] == value
unit_type	Exact	e.g., "html", "video"
tags	Partial	At least one tag must match



⸻

🧪 Sample Test Input

{
  "query": "Explain mitosis",
  "course_id": "course-v1:Bio+101+2024",
  "unit_type": "html",
  "tags": ["cell division", "biology"]
}

Sample Output

{
  "results": [
    {
      "id": "unit-id-001",
      "title": "Phases of Mitosis",
      "score": 0.923,
      "course_id": "course-v1:Bio+101+2024",
      "unit_type": "html",
      "tags": ["biology", "mitosis"]
    }
  ]
}



⸻

🧠 Bonus: Advanced Filtering Ideas

Feature	How
Date ranges	Add created_at, updated_at in metadata
Difficulty level	Add "difficulty": "beginner" to metadata
Section-based	Use chapter/section IDs to limit scope



⸻

✅ Summary

You’ve now extended your hybrid search engine to support fine-grained filters — ideal for:
	•	✍️ Training content authoring
	•	📚 LMS search widgets
	•	💬 Context-aware AI assistants

⸻

Would you like next:
	•	📊 Admin logs for search tracking?
	•	🧠 Add a “reranking” LLM step for refining results?
	•	🖼️ Build a Tutor Studio plugin UI to use this filterable search?

Let me know what you’d like to build!


Perfect! Let’s now add a simple but functional web interface for testing your hybrid search API visually. This is ideal for:
	•	✅ Curriculum designers
	•	✅ LMS administrators
	•	✅ AI plugin developers testing search quality

⸻

🎯 Goal

Create a Django view with:
	•	🔍 A search input
	•	📜 Result list with title, score, and ID
	•	🎨 A clean Bootstrap UI (optional)

⸻

🧱 Folder Structure (inside your hybridsearch app)

hybridsearch/
├── templates/
│   └── hybridsearch/
│       └── search_ui.html
├── views.py
├── urls.py



⸻

1️⃣ HTML Template — templates/hybridsearch/search_ui.html

<!DOCTYPE html>
<html>
<head>
    <title>Hybrid Search Tester</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css">
</head>
<body class="p-4">
<div class="container">
    <h2>🔍 Hybrid Search Tester</h2>
    <form id="search-form" class="my-4">
        <div class="mb-3">
            <input type="text" id="query" name="query" class="form-control" placeholder="Enter your query..." required>
        </div>
        <button type="submit" class="btn btn-primary">Search</button>
    </form>

    <div id="results" class="mt-4"></div>
</div>

<script>
document.getElementById('search-form').addEventListener('submit', async function(e) {
    e.preventDefault();
    const query = document.getElementById('query').value;
    const response = await fetch("/hybridsearch/search/", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ query: query })
    });
    const data = await response.json();

    const container = document.getElementById("results");
    container.innerHTML = `<h5>Results for: <em>${query}</em></h5><ul class="list-group mt-3">`;
    data.results.forEach(result => {
        container.innerHTML += `<li class="list-group-item">
            <strong>${result.title}</strong><br>
            <small>ID: ${result.id}</small><br>
            <span class="badge bg-success">Score: ${result.score.toFixed(3)}</span>
        </li>`;
    });
    container.innerHTML += `</ul>`;
});
</script>
</body>
</html>



⸻

2️⃣ Add Django View — views.py

from django.shortcuts import render

def hybrid_search_ui(request):
    return render(request, "hybridsearch/search_ui.html")



⸻

3️⃣ Add Route — urls.py

from django.urls import path
from .views import hybrid_search_view, hybrid_search_ui

urlpatterns = [
    path("search/", hybrid_search_view, name="hybrid_search_api"),
    path("ui/", hybrid_search_ui, name="hybrid_search_ui"),
]

Then include in your root urls.py:

path("hybridsearch/", include("hybridsearch.urls")),



⸻

✅ Test the Interface

Visit:

http://localhost:8000/hybridsearch/ui/

Try sample queries like:
	•	"What is DNA?"
	•	"Explain photosynthesis"
	•	"Molecular biology basics"

You’ll get a list of hybrid-ranked results with:
	•	Title
	•	ID
	•	Score

⸻

🎨 Optional UI Enhancements
	•	✅ Add course/unit tags
	•	📌 Include snippet from unit content
	•	📊 Show side-by-side BM25 vs Vector rank comparison
	•	🔁 Export result set to CSV for benchmarking

⸻

Would you like to:
	•	📊 Log queries and results to a model for analysis?
	•	🧪 Add result evaluation scoring (relevance voting)?
	•	🎓 Restrict access to instructors only?

Let me know how you’d like to evolve it!


Perfect! Adding relevance voting (👍 / 👎) to your hybrid search UI makes it easy to:
	•	Collect real-world feedback for search tuning
	•	Identify which queries return poor results
	•	Use feedback to refine embeddings, prompts, or weights

Let’s add:
	1.	✅ A SearchFeedback model
	2.	🧠 Save feedback via AJAX from the UI
	3.	📊 Optionally view feedback in Django admin

⸻

🧱 1. Define the Feedback Model

models.py

from django.db import models

class SearchFeedback(models.Model):
    query = models.TextField()
    result_id = models.CharField(max_length=255)
    result_title = models.TextField()
    score = models.FloatField()
    is_relevant = models.BooleanField()
    submitted_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{'👍' if self.is_relevant else '👎'} - {self.result_title}"



⸻

🛠 2. Run Migrations

python manage.py makemigrations hybridsearch
python manage.py migrate



⸻

🖥️ 3. Update the UI Template

Add Voting Buttons to search_ui.html

Update this part inside your results loop:

data.results.forEach(result => {
    container.innerHTML += `<li class="list-group-item">
        <strong>${result.title}</strong><br>
        <small>ID: ${result.id}</small><br>
        <span class="badge bg-success">Score: ${result.score.toFixed(3)}</span>
        <div class="mt-2">
            <button class="btn btn-sm btn-outline-success me-2" onclick="submitFeedback('${query}', '${result.id}', '${result.title}', ${result.score}, true)">👍 Relevant</button>
            <button class="btn btn-sm btn-outline-danger" onclick="submitFeedback('${query}', '${result.id}', '${result.title}', ${result.score}, false)">👎 Not Relevant</button>
        </div>
    </li>`;
});

Add AJAX Function

<script>
async function submitFeedback(query, id, title, score, isRelevant) {
    await fetch("/hybridsearch/feedback/", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            query: query,
            result_id: id,
            result_title: title,
            score: score,
            is_relevant: isRelevant
        })
    });
    alert("✅ Feedback submitted. Thank you!");
}
</script>



⸻

🌐 4. Add Feedback View

views.py

from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse
from .models import SearchFeedback
import json

@csrf_exempt
def submit_feedback(request):
    if request.method == "POST":
        data = json.loads(request.body)
        SearchFeedback.objects.create(
            query=data["query"],
            result_id=data["result_id"],
            result_title=data["result_title"],
            score=data["score"],
            is_relevant=data["is_relevant"]
        )
        return JsonResponse({"message": "feedback received"})
    return JsonResponse({"error": "POST only"}, status=405)



⸻

🔗 5. Add URL Route

urls.py

from .views import submit_feedback

urlpatterns += [
    path("feedback/", submit_feedback, name="submit_feedback"),
]



⸻

🗃️ 6. Admin Support (Optional)

admin.py

from django.contrib import admin
from .models import SearchFeedback

@admin.register(SearchFeedback)
class FeedbackAdmin(admin.ModelAdmin):
    list_display = ("query", "result_title", "is_relevant", "submitted_at")
    search_fields = ("query", "result_title")
    list_filter = ("is_relevant", "submitted_at")



⸻

✅ Summary
	•	Your users can now vote on each result’s relevance
	•	Votes are stored in your DB for analysis
	•	Easily accessible via Django admin

⸻

🔮 Next Steps

Would you like to:
	•	📈 Add a feedback dashboard with stats (e.g., most failed queries)?
	•	🧪 Benchmark hybrid search using feedback logs?
	•	🧠 Fine-tune vector weights based on user feedback?

Let me know, and I can help build that next!

Let’s build a feedback dashboard in Django to visualize relevance voting results and help you improve search quality based on real user data.

⸻

🧭 What You’ll Get

A page like /hybridsearch/dashboard/ showing:
	•	📊 Most frequently submitted queries
	•	❌ Queries with the highest % of “not relevant” votes
	•	✅ Overall relevance rates
	•	🕒 Filter by time (e.g., last 7 days)

⸻

✅ Step-by-Step Implementation

1️⃣ Add the Dashboard View

views.py

from django.shortcuts import render
from django.db.models import Count, Avg
from django.utils.timezone import now, timedelta
from .models import SearchFeedback

def feedback_dashboard(request):
    days = int(request.GET.get("days", 7))
    since = now() - timedelta(days=days)

    feedbacks = SearchFeedback.objects.filter(submitted_at__gte=since)

    total = feedbacks.count()
    relevant = feedbacks.filter(is_relevant=True).count()
    not_relevant = feedbacks.filter(is_relevant=False).count()
    relevance_rate = (relevant / total) * 100 if total else 0

    # Group by query
    query_stats = (
        feedbacks
        .values("query")
        .annotate(
            total=Count("id"),
            relevant=Count("id", filter=models.Q(is_relevant=True)),
            not_relevant=Count("id", filter=models.Q(is_relevant=False)),
            relevance=Avg("is_relevant")
        )
        .order_by("-total")[:20]
    )

    return render(request, "hybridsearch/dashboard.html", {
        "total": total,
        "relevant": relevant,
        "not_relevant": not_relevant,
        "relevance_rate": round(relevance_rate, 1),
        "query_stats": query_stats,
        "days": days,
    })



⸻

2️⃣ Add Dashboard Template

templates/hybridsearch/dashboard.html

<!DOCTYPE html>
<html>
<head>
  <title>Hybrid Search Feedback Dashboard</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css">
</head>
<body class="p-4">
<div class="container">
  <h2>📊 Hybrid Search Feedback Dashboard</h2>

  <form method="get" class="mb-3">
    <label for="days">Showing feedback from the last</label>
    <select name="days" id="days" onchange="this.form.submit()">
      <option value="7" {% if days == 7 %}selected{% endif %}>7 days</option>
      <option value="30" {% if days == 30 %}selected{% endif %}>30 days</option>
      <option value="90" {% if days == 90 %}selected{% endif %}>90 days</option>
    </select>
  </form>

  <p>Total Feedback: <strong>{{ total }}</strong> | Relevant: <strong>{{ relevant }}</strong> | Not Relevant: <strong>{{ not_relevant }}</strong> | Relevance Rate: <strong>{{ relevance_rate }}%</strong></p>

  <table class="table table-striped table-bordered mt-4">
    <thead>
      <tr>
        <th>Query</th>
        <th>Total Votes</th>
        <th>👍 Relevant</th>
        <th>👎 Not Relevant</th>
        <th>Relevance %</th>
      </tr>
    </thead>
    <tbody>
      {% for q in query_stats %}
        <tr>
          <td>{{ q.query }}</td>
          <td>{{ q.total }}</td>
          <td>{{ q.relevant }}</td>
          <td>{{ q.not_relevant }}</td>
          <td>{{ q.relevance|floatformat:1 }}%</td>
        </tr>
      {% empty %}
        <tr><td colspan="5">No feedback available for this period.</td></tr>
      {% endfor %}
    </tbody>
  </table>
</div>
</body>
</html>



⸻

3️⃣ Add URL Route

urls.py

from .views import feedback_dashboard

urlpatterns += [
    path("dashboard/", feedback_dashboard, name="feedback_dashboard"),
]

Now you can visit:

http://localhost:8000/hybridsearch/dashboard/



⸻

🧠 Optional Enhancements

Feature	Description
🔍 Search bar	Filter by keyword
📊 Charts	Pie chart of relevance % using Chart.js
⬇ Export	Add CSV export of feedback
👥 User stats	Show top queries per user if auth is enabled



⸻

✅ You Now Have:
	•	A dashboard showing feedback stats per query
	•	Relevance ratings (% of positive votes)
	•	Quick filters by time window

⸻

Would you like to add:
	•	📈 Charts for visualization?
	•	📄 Export to Excel or CSV?
	•	🔒 Staff-only access to the dashboard?

Let me know how you want to expand it!