# QueryMind

![n8n](https://img.shields.io/badge/Built%20with-n8n-orange?logo=n8n&logoColor=white)
![Ollama](https://img.shields.io/badge/LLM-Ollama%20llama3.1%3A8b-000000?logo=ollama&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-Live%20Database-4479A1?logo=mysql&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Chat%20Memory-336791?logo=postgresql&logoColor=white)
![Workflow](https://img.shields.io/badge/Workflow-SQL%20AI%20Agent-0F766E)

A conversational SQL AI agent built in n8n that lets anyone query a live MySQL database in plain English — no SQL required. The agent generates the query, runs it, and answers back in a sentence.

## What This Workflow Does

QueryMind sits in front of a MySQL database and answers natural-language questions about it through a chat interface. A user asks something like "how much damage did car ABC123 cause?" and the agent writes the SQL, executes it against the database, and replies conversationally.

Every question and answer is logged to an audit table, and conversation history is persisted across sessions via PostgreSQL so the agent has long-term memory of past chats.

## End-to-End Flow

1. Chat Trigger receives the user's plain-English question through n8n's built-in Chat UI.
2. The AI Agent (Tools Agent) interprets the question against a fixed schema and relationship map for the target database.
3. The agent calls the `run_sql` MySQL tool, generating and executing a `SELECT`-only query.
4. Results come back as raw rows.
5. The agent turns the rows into a short, plain-English answer — bullet points for lists, a clear statement when nothing matches.
6. The question and answer are logged to a `query_logs` table in MySQL.
7. Chat Memory (PostgreSQL) stores the exchange so context carries across sessions.
8. Respond to Webhook returns the final answer to the Chat UI.

## Main Integrations

- n8n Chat Trigger for the conversational front end
- Ollama (`llama3.1:8b`, local) as the reasoning and SQL-generation model
- MySQL as the queried database and as the audit-log store
- PostgreSQL as long-term chat memory

## Repository Contents

- [sql_agent.json](sql_agent.json): the n8n workflow export
- [README.md](README.md): project documentation
- [QueryMind_Report.pdf](QueryMind_Report.pdf): full build report — problems faced, models tested, lessons learned
- [screenshots/chat_interface.png](screenshots/chat_interface.png): the chat UI in action
- [screenshots/n8n_wf.png](screenshots/n8n_wf.png): the workflow canvas in n8n

## Prerequisites

Before importing and running the workflow, make sure you have:

- An active n8n instance (self-hosted via Docker)
- Ollama running locally with `llama3.1:8b` pulled, reachable via `host.docker.internal:11434`
- A MySQL 8 instance with a dedicated read-only `n8n` user
- A PostgreSQL instance for persistent chat memory
- If running in Docker: use `host.docker.internal` for every local service — `localhost` and `127.0.0.1` resolve to the container, not the host

## Setup

1. Import `sql_agent.json` into n8n.
2. Connect the Ollama Chat Model node to your local Ollama instance.
3. Point the `run_sql` MySQL Tool node at your database using the read-only `n8n` credential (see below).
4. Point the Chat Memory node at your PostgreSQL instance.
5. Update the system prompt's `DATABASE` and `SCHEMA` sections to match the database you're querying — the agent does not discover schema at runtime, it needs it hardcoded.
6. Activate the workflow and open the Chat UI.

## More Details

Configuration notes, the MySQL/PostgreSQL setup commands, the local-vs-cloud model comparison, error handling, security notes, and troubleshooting are all covered in depth in the [build report](QueryMind_Report.pdf) — it's worth reading before you deploy this against a new database.
