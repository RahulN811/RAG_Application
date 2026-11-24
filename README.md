# PDF Question Answering with RAG

A small Retrieval-Augmented Generation (RAG) learning project. The Jupyter notebook loads a PDF, splits its text into chunks, embeds the chunks with a Sentence Transformers model, retrieves relevant chunks with FAISS, and asks a Groq-hosted language model to answer from the retrieved context. It also includes an LLM-based evaluation workflow over a question set.

> The current runnable implementation is in `Document_Loader.ipynb`. `main.py` is currently a placeholder and only prints `Hello from rag!`.

## How it works

1. Read PDF pages and keep page/source metadata.
2. Split page text into overlapping chunks (currently 500 characters with 50 characters overlap).
3. Embed chunks with `sentence-transformers/all-MiniLM-L6-v2`.
4. Add vectors to a FAISS `IndexFlatL2` index.
5. Embed a question and retrieve the nearest chunks (default `top_k=2`).
6. Send the retrieved text as context to Groq's `openai/gpt-oss-120b` model. The prompt asks it to answer only from that context and say when the answer is missing.
7. Optionally evaluate generated answers with structured judgments for correctness, faithfulness, answer relevance, and context relevance.

## Requirements

- Python 3.11 or newer
- A Groq API key for answer generation and LLM evaluation
- The project dependencies listed in `requirements.txt` (or `pyproject.toml`)

## Setup

From the project directory, create and activate a virtual environment, then install dependencies:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Create a `.env` file in the project root with your key:

```dotenv
GROQ_API_KEY=your_groq_api_key
```

Keep `.env` private. It should be ignored by Git and must not be committed. If a key was ever committed or exposed, revoke it with Groq and create a replacement.

## Run the notebook

Start Jupyter from the project directory:

```powershell
python -m pip install ipykernel
python -m jupyter notebook
```

Open `Document_Loader.ipynb` and run its cells in order. The notebook currently refers to `RAG_Practice_Guide.pdf` and `eval_questions.json` using absolute paths such as `D:\RAG\...`. If your project is in a different directory, change these references to paths relative to the notebook/project directory before running. For example:

```python
from pathlib import Path
project_dir = Path.cwd()
pdf_path = project_dir / "RAG_Practice_Guide.pdf"
eval_path = project_dir / "eval_questions.json"
```

The embedding model may download from Hugging Face on first use. The notebook's evaluation section sends questions, reference answers, generated answers, and retrieved context to the configured Groq model.

## Evaluation data

`eval_questions.json` contains question records with `id`, `question`, and `ground_truth` fields. The notebook uses these to generate answers and assess them. Its structured evaluation records:

- Correctness: `correct`, `partially_correct`, or `incorrect`
- Faithfulness: `faithful`, `partially_faithful`, or `unfaithful`
- Answer relevance and context relevance scores from 0 to 1
- A short reasoning field

The notebook also calculates exact and weighted correctness percentages. These are model-based evaluation results and should be reviewed rather than treated as definitive scores.

## Project files

- `Document_Loader.ipynb` — PDF loading, chunking, embedding, FAISS retrieval, generation, and evaluation experiments
- `RAG_Practice_Guide.pdf` — sample knowledge source
- `eval_questions.json` — evaluation questions and reference answers
- `main.py` — current placeholder entry point
- `requirements.txt` / `pyproject.toml` — Python dependencies and project metadata

## Current limitations

- The workflow is notebook-based; `main.py` does not yet run the RAG pipeline.
- PDF and evaluation file paths in the notebook are currently machine-specific and need to be made relative or configured for another environment.
- The FAISS index is built in memory each time the notebook setup cells are run; it is not persisted.
- The current sample uses L2 distance and fixed chunk/retrieval settings. Retrieval quality and generated answers depend on the source PDF, model, and settings.
- The project dependencies include Chroma, but the current notebook retrieval implementation uses FAISS.

## Security

Never place API keys in source files, notebooks, evaluation data, or commits. Load secrets through environment variables and keep `.env` out of version control.
