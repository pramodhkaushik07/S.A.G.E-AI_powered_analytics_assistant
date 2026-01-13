# S.A.G.E. - Smart Assistant for Guided Enablement

S.A.G.E is an AI assistant that unifies conversational reasoning, document retrieval, and automated spreadsheet visualization into a single system. It is designed for users who need rapid access to project knowledge, structured answers grounded in internal documents, and on-demand data analysis without manually writing code or exploring spreadsheets. S.A.G.E reduces workflow switching and provides immediate, context-aware insights.

## Key Capabilities
- Conversational Guidance
Natural, assistant-style chat for explanations, strategy, and general support.

- Retrieval-Augmented Generation (RAG)
Retrieves and summarizes relevant organizational documents.

- Automated Data Visualization
Generates charts automatically from CSV/Excel files.

- Error-Tolerant Code Generation
Self-correcting fallback mechanism for chart creation.

- Seamless Chainlit UI Integration

## Requirements
### Python Version
- Python 3.10+
### Dependencies
Install via pip:
```
pip install langgraph langchain-core langchain-chroma chromadb openai pandas numpy matplotlib seaborn python-dotenv chainlit

```
Environment Variables `(.env)`
```
OPENAI_API_KEY=your_api_key
CHROMA_DIR=path_to_chroma_store
DATA_DIR=path_to_Synthetic_Data_folder

```
## Execution Modes
S.A.G.E determines the mode automatically based on user input:

### 1) Chat Mode

- Pure conversational output

- No document retrieval

- Ideal for explanation, reasoning, and general guidance

### 2) RAG Mode

- Retrieves relevant text-based documents

- Produces grounded answers backed by retrieved content

- Spreadsheet files are excluded to keep responses semantic

### 3) Visualization Mode

Two-stage process: chart_rag → chart_code

A. chart_rag — Dataset Selection

- Retrieves spreadsheets + descriptive documents

- Weighted retrieval: 60% tabular, 40% descriptive

- Extracts schema (column names, datatypes)

- Enhances the user query with column hints

- Selects the best dataset and stores it in LangGraph state

B. chart_code — Chart Generation

- Loads selected CSV or Excel file

- Normalizes column names

- LLM generates executable visualization code + summary

- Code runs automatically to produce a PNG image

- Image displayed in Chainlit UI

## Fallback Mechanism for chart_code

If chart code fails:

1. Capture the Python error

2. Send error back to LLM

3. Regenerate simplified code

4. Retry execution

5. If it fails again → return guided error message

### Purpose:
•	Spreadsheet structures vary
•	Users seldom specify column names correctly
•	LLM corrections require runtime feedback to stabilize visualization

## LangGraph State Design

S.A.G.E is a shared, persistent dictionary that flows through every node in the pipeline.
It does not reset between nodes, and each node reads from and writes to the same state object.
## LangGraph State Fields

| **State Field**   | **Description** |
|-------------------|-----------------|
| `messages`        | Conversation history |
| `last_user_text`  | Most recent user query |
| `query_type`      | `"chat"`, `"rag"`, or `"visualization"` |
| `chart_image`     | Base64 PNG of the generated chart |
| `chart_summary`   | Short explanation of the visualization |
| `chart_context`   | Dataset description selected during `chart_rag` |
| `chart_files`     | List of selected CSV/XLSX file paths |
| `sources`         | Metadata for retrieved documents |

## Pipeline Execution
1. Build the Vector Database

` python vector_db.py `

Loads documents → chunks → embeds → writes vectors into ChromaDB.

2. Launch Chainlit UI

` chainlit run chainlit_app.py --host 0.0.0.0 --port 8000 `

Chainlit imports langgraph_pipeline.py, compiles the LangGraph graph, and executes it for every user message.

3. Optional: Run LangGraph Debug Mode

```
from langgraph_pipeline import run_debug_test
run_debug_test("test query")
```

Useful during development; not meant for production.

## Google Colab Networking (ngrok Required)

- Local PC: No tunneling required

- Google Colab: Must use ngrok, because Colab cannot expose internal ports

ngrok creates a public HTTPS URL that forwards requests to your Chainlit port.

## Running S.A.G.E in Google Colab

S.A.G.E can run inside Google Colab, but few additional setup steps are required because Colab cannot expose localhost ports and does not store files permanently. Your entire S.A.G.E project folder must be placed inside Google Drive (for example: MyDrive/SAGE_Project/) and mounted inside Colab so the vector database and data files remain accessible. The OpenAI API key and ngrok Auth Token must be added through the Colab Secrets panel, not written directly into the notebook. ngrok is required to make the Chainlit interface accessible externally, since Colab cannot open port 8000 by itself. Once Drive is mounted, the secrets are loaded, and ngrok initializes successfully, you can launch Chainlit normally, and the generated public ngrok URL becomes your interface for interacting with S.A.G.E.

## Running Chainlit in Google Colab
Follow the steps below to launch your `Chainlit app (chainlit_app.py)` from Google Colab using ngrok.
1. Mount Google Drive
Your project should be located in:
` /content/drive/MyDrive/SAGE_Project/ `

```
from google.colab import drive
drive.mount('/content/drive')

```
2. Load OpenAI API Key from Colab Secrets

Make sure you saved your secret in Colab using:

#### Colab menu → Edit → Variables → Add New Variable

```
import os
OPENAI_API_KEY = userdata.get("OPENAI_API_KEY")
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY

```
3. Start ngrok (to expose Chainlit publicly)

You must also store your NGROK_AUTH_TOKEN in Colab variables.

```
from pyngrok import ngrok

NGROK_TOKEN = userdata.get("NGROK_AUTH_TOKEN")
!ngrok config add-authtoken $NGROK_TOKEN

ngrok.kill()
public_url = ngrok.connect(8000, "http")
print("Public Chainlit URL:", public_url)
```
4. Run Chainlit

This launches your Chainlit app from Drive.

```
!chainlit run /content/drive/MyDrive/SAGE_Project/chainlit_app.py \
    --host 0.0.0.0 --port 8000

```
You should now see a public URL from ngrok.
Open it in your browser to use the Chainlit interface.

## 
This README provides the full functional overview of S.A.G.E, including purpose, capabilities, architecture, and correct setup procedure.

