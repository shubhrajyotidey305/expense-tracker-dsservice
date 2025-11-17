Expense Tracker DS Service
--------------------------

A small Flask service that classifies incoming text as a potential bank SMS and, when matched, uses OpenAI via LangChain to extract structured expense details (amount, merchant, currency). Intended to power downstream expense tracking workflows.

Prerequisites
-------------
- Python 3.11+ recommended
- OpenAI API key with access to `gpt-5-nano`
- `virtualenv` or another Python environment manager

Setup
-----
1) Create and activate an environment
```bash
python -m venv .venv
source .venv/bin/activate
```
2) Install dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```
3) Provide your OpenAI credentials in `.env`
```bash
echo "OPENAI_API_KEY=sk-..." > .env
```

Running the service
-------------------
From the repo root:
```bash
PYTHONPATH=src FLASK_ENV=development python -m flask --app app run --host 0.0.0.0 --port 8000
```
The Flask app is defined in `src/app/__init__.py` and also supports direct invocation for local debugging:
```bash
PYTHONPATH=src python -m app
```

API
---
- `GET /` – simple health check returning `Hello world`.
- `POST /v1/ds/message` – body: `{"message": "<text>"}`; returns extracted keys when the message looks like a bank SMS, otherwise `null`.

Example
-------
```bash
curl -X POST http://localhost:8000/v1/ds/message \
  -H "Content-Type: application/json" \
  -d '{"message": "INR 850.75 spent on your card at ABC Stores."}'
```
Sample response:
```json
{
  "amount": "850.75",
  "merchant": "ABC Stores",
  "currency": "INR"
}
```

How it works
------------
- `MessageService` uses `MessageUtil` keyword matching to gate messages that resemble bank/card SMS.
- `LLMService` loads the OpenAI key from `.env`, prompts `gpt-5-nano` with LangChain, and requests structured output into the `Expense` Pydantic model.
- When a message fails the bank-SMS heuristic, the endpoint returns `null` to avoid unnecessary LLM calls.

Notes
-----
- Missing or invalid `OPENAI_API_KEY` will result in errors when calling `/v1/ds/message`.
- Adjust model name in `src/app/service/llmService.py` if you prefer a different deployment.
