<div align="center">

<img src="./assets/banner.svg" alt="AI Travel Planning System using LangGraph" width="100%" />

# AI Travel Planning System using LangGraph

</div>

This project is a multi-agent AI system built with LangGraph. Four agents work one after another to plan a trip: one finds flights, one finds hotels, one writes an itinerary, and the last one combines everything into a final travel plan.

The system remembers conversations using PostgreSQL, and it can be used from the terminal or from a Streamlit web app.

---

## Features

- Flight Agent that fetches flight information
- Hotel Agent that searches for hotels
- Itinerary Agent that creates a travel plan
- Final Agent that combines everything into one response
- Conversation memory stored in PostgreSQL
- Real-time API integration
- Streamlit web interface with live agent progress
- Travel plans saved automatically as Markdown files, with a download button

---

## Tech Stack

| Technology | Purpose |
| --- | --- |
| Python | Core application |
| LangGraph | Multi-agent workflow |
| LangChain | LLM framework |
| Groq (Llama 3.3 70B) | Language model |
| PostgreSQL | Conversation memory |
| Streamlit | Web interface |
| Tavily API | Hotel search |
| AviationStack API | Flight data |
| python-dotenv | Environment variables |

---

## Project Workflow

1. The user enters a travel request
2. Flight Agent fetches flight data
3. Hotel Agent searches for hotels
4. Itinerary Agent creates a day-by-day plan using the flight and hotel results
5. Final Agent combines everything into the final response
6. PostgreSQL saves the conversation state

```mermaid
flowchart LR
    S([Start]) --> A[Flight Agent]
    A --> B[Hotel Agent]
    B --> C[Itinerary Agent]
    C --> D[Final Agent]
    D --> E([End])
```

---

## Agents

| Agent | What it does | Uses |
| --- | --- | --- |
| Flight Agent | Fetches flight details such as airline, departure, arrival and status | AviationStack API |
| Hotel Agent | Searches for the best hotels for the request | Tavily API |
| Itinerary Agent | Creates a travel itinerary from the request, flights and hotels | Llama 3.3 70B (Groq) |
| Final Agent | Writes the final travel response | Llama 3.3 70B (Groq) |

---

## System Architecture

```mermaid
flowchart TB
    U[User] --> UI[Streamlit Web App<br/>or Terminal]
    UI --> G[LangGraph Workflow]

    subgraph Agents
        direction LR
        A[Flight Agent] --> B[Hotel Agent] --> C[Itinerary Agent] --> D[Final Agent]
    end

    G --> A
    A -.-> AV[AviationStack API]
    B -.-> TV[Tavily API]
    C -.-> LLM[Groq<br/>Llama 3.3 70B]
    D -.-> LLM
    G <--> PG[(PostgreSQL<br/>Checkpoint Memory)]
    D --> UI
```

---

## Shared State

All agents read from and write to one shared state, called `TravelState`.

| Field | Description |
| --- | --- |
| `messages` | Conversation messages |
| `user_query` | The user's travel request |
| `flight_results` | Output of the Flight Agent |
| `hotel_results` | Output of the Hotel Agent |
| `itinerary` | Output of the Itinerary Agent |
| `llm_calls` | Counter shown in the web app |

```mermaid
flowchart LR
    Q[user_query] --> F[Flight Agent] --> FR[flight_results]
    Q --> H[Hotel Agent] --> HR[hotel_results]
    FR --> I[Itinerary Agent]
    HR --> I
    Q --> I
    I --> IT[itinerary]
    FR --> Z[Final Agent]
    HR --> Z
    IT --> Z
    Z --> M[messages: final response]
```

---

## Request Flow

What happens when a user submits a request in the web app.

```mermaid
sequenceDiagram
    participant U as User
    participant S as Streamlit
    participant L as LangGraph
    participant F as AviationStack
    participant T as Tavily
    participant G as Groq LLM
    participant P as PostgreSQL

    U->>S: Enter travel request
    S->>L: Start workflow (thread_id)
    L->>F: Get flights
    F-->>L: Flight data
    L->>P: Save state
    L->>T: Search hotels
    T-->>L: Hotel results
    L->>P: Save state
    L->>G: Create itinerary
    G-->>L: Itinerary
    L->>P: Save state
    L->>G: Write final response
    G-->>L: Final plan
    L->>P: Save state
    L-->>S: Stream each agent's result
    S-->>U: Show plan and save as .md file
```

---

## Memory

Memory is handled by the LangGraph PostgreSQL checkpointer. Each user session has a `thread_id` (the User ID field in the sidebar of the web app). The state of the workflow is saved in PostgreSQL after every step, so conversations are stored per user.

```mermaid
flowchart LR
    U[User ID / thread_id] --> W[LangGraph Workflow]
    W --> C[PostgresSaver]
    C --> D[(PostgreSQL)]
    D --> C
```

---

## Project Structure

```
.
├── assets/
│   └── banner.png         # README banner
├── tools/
│   ├── flight_tool.py     # AviationStack flight search
│   └── tavily_tool.py     # Tavily hotel search
├── main.py                # LangGraph agents, graph and terminal runner
├── frontend.py            # Streamlit web app
├── .env                   # API keys and database URL (do not upload)
└── README.md
```

---

# Step 1: Create Python Environment

Open the terminal inside the project folder and run:

```bash
python -m venv langgraph_env
```

Activate the environment.

Windows:

```bash
langgraph_env\Scripts\activate
```

Mac / Linux:

```bash
source langgraph_env/bin/activate
```

---

# Step 2: Install Dependencies

```bash
pip install langgraph langchain langchain-openai langchain-groq langchain-community langchain-tavily "psycopg[binary]" psycopg_pool python-dotenv tavily-python requests streamlit

pip install -U "psycopg[binary,pool]" langgraph-checkpoint-postgres
```

---

# Step 3: Install PostgreSQL

Download and install PostgreSQL: https://www.postgresql.org/download/

While installing, remember these two things:

- PostgreSQL password
- Port number

You will need them for the database connection string.

---

# Step 4: Create the Database

Open PostgreSQL (or pgAdmin) and run:

```sql
CREATE DATABASE langgraph_memory_demo;
```

---

# Step 5: Set Up the `.env` File

Create a `.env` file in the project folder:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
AVIATIONSTACK_API_KEY=your_aviationstack_api_key

DATABASE_URL=postgresql://postgres:your_password@localhost:5432/langgraph_memory_demo
```

Use your own password and port in `DATABASE_URL`.

Important: never upload your real `.env` file to GitHub. Add `.env` to `.gitignore`.

---

# Step 6: Get the API Keys

| Service | Where to get the key |
| --- | --- |
| Groq | https://console.groq.com |
| Tavily | https://tavily.com |
| AviationStack | https://aviationstack.com |

---

# Step 7: Run the Application

Run in the terminal:

```bash
python main.py
```

Run the Streamlit web app:

```bash
streamlit run frontend.py
```

PostgreSQL must be running before you start either one, because the workflow connects to the database when it starts.

---

## Example Prompts

- Plan a complete 7 days Japan trip including flights, hotels and sightseeing under 2 lakhs.
- Paris trip for 5 days
- Dubai weekend trip
- Bali backpacking 10 days

---

## Output

In the web app, each agent's result appears live as it finishes. When the workflow ends, the final plan is saved to a `travel_plans/` folder as a Markdown file, and it can also be downloaded from the page.

---

## Notes

- The flight search currently returns the first 5 flights from AviationStack and does not filter by the route in the user's request.
- Hotel results are the top 5 Tavily search results, shortened to about 300 characters each.
- The agents run in a fixed order. They do not choose the next step on their own.

---

## Future Improvements

- Filter flights by origin, destination and date
- Add a weather agent
- Let the workflow choose which agents to run depending on the request
- Add a requirements file for one-step installation
- Add budget checking for the final plan
