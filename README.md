# 📚 Semantic Book Recommender: Technical Deep Dive
[![MIT License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python](https://img.shields.io/badge/Made%20with-Python-1f425f.svg)](https://www.python.org/)

An end-to-end semantic book recommendation system leveraging vector embeddings (`sentence-transformers`), approximate nearest neighbor search (FAISS), and generative AI (Google Gemini/Vertex AI) for personalized analysis and content generation, wrapped in an interactive Gradio web UI.

[<img src="https://raw.githubusercontent.com/ivancastroprojects/semantic-book-recommender/main/assets/gifrecommenderbooks.gif" alt="Application Demo GIF" width="800"/>](https://drive.google.com/file/d/138ioFt1HbPh65Qj84gDxkFafTE9iEdC-/view?usp=sharing)

*Haz clic en el GIF para ver el video demo completo.*

**(For a comprehensive architecture overview, data flow diagrams, and UML, please refer to the [**TECHNICAL_DOCUMENTATION.md**](TECHNICAL_DOCUMENTATION.md))**

## ⭐ Core Features & Technical Highlights

- **Deep Semantic Search**: Moves beyond keyword matching by converting natural language queries and book descriptions into high-dimensional vectors using `sentence-transformers`. Employs **FAISS (CPU)** for efficient similarity search (cosine distance) in the vector space, capturing nuanced meaning.
- **Advanced Filtering & Ranking**: Allows users to refine results by genre and author. Provides sorting options based on semantic relevance (vector distance), user rating, popularity (number of ratings), and publication date.
- **Interactive UI (Gradio)**: Offers a reactive web interface built with Gradio, featuring a gallery view for results, detailed book information upon selection, and integrated AI capabilities.
- **Persistent User State**: Enables users to mark books as **Favorites** (persisted in `favorites.json`) and review their **Search History** (persisted in `search_history.json`), maintaining context across sessions.
- **AI-Powered Analysis (Google Gemini)**: Delivers concise, AI-generated book summaries and personalized explanations (`Why you might like this book?`) connecting the recommendation to the user's *specific query* and favorite history, leveraging **Google Gemini**'s reasoning capabilities via LangChain orchestration.
- **Creative Visual Generation (Google Gemini & Vertex AI)**: Brings book concepts to life by generating **conceptual images** and experimental **short videos** based on plot or themes, utilizing the multimodal capabilities of Google's models.
- **Favorite-Influenced Re-ranking**: Includes an optional feature (`ENABLE_FAVORITE_BOOSTING`) to bias search results towards books similar to the user's favorites, calculated via weighted averaging of query and average favorite vectors.
- **Efficient Image Handling (`Pillow`)**: Downloads and caches book cover images locally, optimizing UI load times and reducing external dependencies during runtime.
- **Flexible Configuration (`.env`)**: Centralizes API keys, project IDs, model names, and file paths via environment variables (`python-dotenv`), promoting security and deployment ease without code modification.

## 🏛️ Architecture & Design Philosophy

This application adheres to a **modular design**, promoting separation of concerns, testability, and maintainability. Key architectural components include:

- **Semantic Core (`modules/vector_db.py`)**: Encapsulates all logic related to embedding generation (`sentence-transformers`) and vector storage/search (`FAISS`). Automatically builds the FAISS index on first run if not found, ensuring ease of setup. FAISS was chosen for its speed on CPU and memory efficiency for medium-sized datasets.
- **AI Orchestration (`modules/llm_clients.py`, LangChain)**: Provides a unified interface for interacting with **Google Gemini** (via `google-generativeai`) and **Vertex AI** (via `google-cloud-aiplatform`). LangChain assists in structuring complex prompts for multi-step reasoning, particularly for the personalized AI analysis.
- **Decoupled Data Management (`modules/data_manager.py`)**: Handles loading, preprocessing (`pandas`), and access to the core book dataset and user history. Isolates data handling logic from the UI and vector database modules.
- **Component-Based UI (`modules/ui/*.py`, Gradio)**: Each application tab (Search, History, Favorites) is built as a distinct Gradio module (`search_tab.py`, `history_tab.py`, etc.). This promotes independent development and testing of UI sections.
- **Simple Persistence**: User favorites and history are stored in simple JSON/CSV files for easy inspection and portability. The FAISS index is persisted as binary files.
- **Centralized Configuration (`modules/config.py`)**: An `AppConfig` class loads settings from `.env` and defines constants, providing a single source of truth for parameters.

### Project Structure Overview

```
gradio-dashboard.py         # Main application entry point
modules/
├── config.py               # Configuration class (reads .env)
├── data_manager.py         # Handles book data, history
├── image_processor.py      # Image downloading and caching
├── llm_clients.py          # Interface to Gemini/Vertex AI
├── user_preferences.py     # Manages favorites
├── vector_db.py            # FAISS index creation and search
├── ui/                     # Gradio UI modules
│   ├── search_tab.py
│   ├── history_tab.py
│   ├── favorites_tab.py
│   └── components.py       # (Potentially shared UI logic or base class)
└── utils/                  # Utility functions
    ├── text_utils.py
    ├── google_books_client.py
    └── example_queries.py
data/
├── datasets/               # Input CSV dataset(s)
├── vector_db/              # Generated FAISS index files (ignored by Git)
├── images/cached/          # Cached cover images (ignored by Git)
└── user_data/              # User favorites & history (ignored by Git)
assets/                     # Static assets (images, CSS)
requirements.txt            # Python dependencies
TECHNICAL_DOCUMENTATION.md  # In-depth technical docs & UML
README.md                   # This file
.gitignore                  # Specifies intentionally untracked files
.env.example                # Example environment variables file
```

## 🛠️ Core Technologies

- **UI Framework**: Gradio
- **Data Handling**: Python 3.9+, Pandas
- **Vector Embeddings**: Sentence-Transformers (via HuggingFace)
- **Vector Database**: FAISS (CPU)
- **AI Models & Orchestration**: Google Gemini API, Google Cloud Vertex AI SDK, LangChain
- **Image Processing**: Pillow
- **External APIs**: Google Books API
- **Dependency Management**: Pip, python-dotenv

## 🚀 Setup & Running Locally

**Prerequisites:**

- Python 3.9+
- Git LFS (Recommended, although large files are now ignored, good practice for potential future assets)
- (Optional but Recommended) A virtual environment (`venv`, `conda`)

**Steps:**

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/ivancastroprojects/semantic-book-recommender.git # Use your repo URL
    cd semantic-book-recommender
    ```

2.  **Set up Virtual Environment (Recommended):**
    ```bash
    python -m venv venv
    # Linux/macOS: source venv/bin/activate
    # Windows: .\venv\Scripts\activate
    ```

3.  **Install Dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure Environment Variables:**
    *   Copy or rename `.env.example` to `.env`.
    *   Edit `.env` and add your actual API keys and configuration:
        ```dotenv
        # --- REQUIRED for AI features ---
        GOOGLE_API_KEY="YOUR_GEMINI_API_KEY"

        # --- REQUIRED for Vertex AI Image Generation (if used) ---
        VERTEX_PROJECT_ID="YOUR_GCP_PROJECT_ID"
        VERTEX_LOCATION="us-central1" # Or your preferred region

        # --- OPTIONAL: Override default models ---
        # VERTEX_IMAGEN_MODEL_ID="imagegeneration@006"
        # VERTEX_LLAMA3_MODEL_ID="llama3-70b-instruct" # Example
        ```

5.  **Dataset:**
    *   Ensure the `books_with_emotions.csv` dataset file is placed in the `data/datasets/` directory.
    *   *(Add download instructions here if the dataset isn't included)*.

6.  **Run the Application:**
```bash
    python gradio_dashboard.py
    ```
    *   **Important Note:** The **first time** you run the application, it will generate the FAISS vector index from the dataset. This process involves downloading the embedding model and processing all book descriptions. **It can take several minutes** (5-20+ min depending on dataset size and CPU speed). Subsequent launches will be much faster as they load the pre-built index.
    *   Access the application via the local URL provided in the console (usually `http://127.0.0.1:7860`).

## ☁️ Deployment (Hugging Face Spaces)

Deploying to [Hugging Face Spaces](https://huggingface.co/spaces) is streamlined for Gradio applications:

1.  **Push Code to GitHub:** Ensure your latest code is on GitHub, and that `.env`, `data/vector_db/`, `data/images/cached/`, and `data/user_data/` are listed in your `.gitignore`.
2.  **Create a Hugging Face Account.**
3.  **Create a New Space:**
    *   Select "**Gradio**" as the SDK.
    *   Choose "**Clone from GitHub**" and provide your repository URL.
    *   **Configure Secrets**: In the Space settings ("Settings" -> "Secrets"), add your `GOOGLE_API_KEY`, `VERTEX_PROJECT_ID`, and `VERTEX_LOCATION` as secrets. These are crucial for the deployed app's AI features.
    *   **Hardware**: Start with the free tier (`CPU basic`). If index creation is too slow or the app needs more power, consider upgrading.
4.  **Build Process:** Hugging Face will clone the repo, install dependencies, and run `gradio_dashboard.py`. The initial build will include the **FAISS index generation**, which can take time.
5.  **Access:** Once built, your application will be live at the public Space URL.

## ⚙️ Performance & Optimization Considerations

- **Index Creation Time:** The initial FAISS index build is the main startup bottleneck. For very large datasets, consider pre-building the index locally and finding a way to upload/load it in the deployment environment (might require paid HF tiers or alternative hosting).
- **Embedding Model:** `all-MiniLM-L6-v2` provides a good balance of speed and quality. Larger, more powerful embedding models exist but would increase index size and potentially slow down search and initial build time.
- **FAISS Index Type:** Currently uses the default `IndexFlatL2` (or `IndexFlatIP` for cosine). For significantly larger datasets (> millions of vectors), exploring approximate indices like `IndexIVFFlat` could improve search speed at the cost of some accuracy.
- **API Latency:** Calls to external APIs (Gemini, Vertex AI, Google Books) introduce latency. The UI provides some feedback (e.g., "Generating..."), but extensive use might feel slow depending on API responsiveness.
- **Gradio Performance:** For very high traffic, standard Gradio deployment might require scaling up hardware resources or exploring alternative deployment strategies (e.g., behind a more robust load balancer).

## 🔮 Future Enhancements & Contribution

This project serves as a robust foundation. Potential areas for future work include:

- **Quantitative Evaluation**: Implement metrics (e.g., NDCG, Recall@K) to formally evaluate recommendation quality.
- **Advanced Re-ranking**: Explore Learning-to-Rank (LTR) models incorporating more user signals.
- **Data Enrichment**: Integrate APIs like OpenLibrary or Wikidata for richer metadata.
- **Scalability**: Test and optimize for larger datasets (alternative vector DBs like Milvus/ChromaDB, more efficient FAISS indices).
- **UI/UX**: Implement result pagination, improved filtering UI, user accounts.
- **Testing**: Develop comprehensive unit and integration tests.

**Contributions are welcome!** Please follow standard GitHub practices (Issues, Forks, Pull Requests) for proposing changes.

## 📄 License

Distributed under the MIT License. (Create a LICENSE file if one doesn't exist).

## 📧 Contact

Iván Castro Martínez - [LinkedIn](https://www.linkedin.com/in/iv%C3%A1n-castro-mart%C3%ADnez-293b9414a/) - [GitHub](https://github.com/ivancastroprojects)

---