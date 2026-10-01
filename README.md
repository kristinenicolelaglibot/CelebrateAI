# 🎂 CelebrateAI
### Plan less. Celebrate more.

A web app that takes a few details about the birthday person and generates a full, personalized party plan: theme, menu with costs, activities, schedule, budget breakdown, shopping checklist, and a ready-to-send invitation. It turns "Planning a party means juggling ideas, prices, and timelines across a dozen tabs and notes" into a single form and one click.

![image alt](https://github.com/kristinenicolelaglibot/CelebrateAI/blob/main/CelebrateAI.png)

## Example

Here's what CelebrateAI generates from a real set of inputs:

**Input:**
![image alt](https://github.com/kristinenicolelaglibot/CelebrateAI/blob/main/celebx.png)

**Generated plan:**
![image alt](https://github.com/kristinenicolelaglibot/CelebrateAI/blob/main/celeby.png)
![image alt](https://github.com/kristinenicolelaglibot/CelebrateAI/blob/main/celebz.png)
![image alt](https://github.com/kristinenicolelaglibot/CelebrateAI/blob/main/celeba.png)
![image alt](https://github.com/kristinenicolelaglibot/CelebrateAI/blob/main/celebb.png)

## Technologies Used

| Tool | Purpose |
|---|---|
| Python + FastAPI | Backend API that receives party details and returns the plan |
| Google Gemini (`gemini-3.1-flash-lite`) | Generates the party plan as structured JSON |
| `google-genai` SDK | Calls the Gemini API with a schema-constrained response |
| Pydantic | Defines and validates the request and the AI response schema |
| HTML / CSS / JavaScript | Frontend form and plan renderer (no framework, no build step) |
| Render *(deployment)* | Hosts the FastAPI backend |
| Vercel *(deployment)* | Hosts the static frontend |

## Features

- **AI-generated party planning**: enter name, age, interests, budget, guest count, location, and food preference, and get a complete plan back.
- **Interest-based theming**: the theme is built around the person's interests (for example, books + travel becomes a bookstore garden party), not a generic template.
- **Structured JSON output**: the AI is forced to return a fixed schema (theme, food, activities, schedule, budget, shopping list, invitation) so each section renders cleanly instead of showing a wall of text.
- **Budget breakdown**: costs per menu item plus a category budget (food, decorations, cake, activities, contingency) with an automatic total.
- **Interactive shopping checklist**: tick items off as you buy them.
- **Ready-to-send invitation**: a short, warm message you can copy straight into a chat or email.
- **Multi-currency support**: PHP, USD, EUR, or GBP.

## The Process: How I Built It

1. **Defined the inputs and outputs first.** Before writing code, I listed what a person planning a party actually needs (theme, food, games, schedule, budget, shopping list, invitation) and what information the AI needs to produce them.
2. **Designed the JSON contract.** I wrote the response shape as Pydantic models (`Plan`, `Theme`, `FoodItem`, `Activity`, `ScheduleItem`, `Budget`) before touching the prompt, so the frontend and backend agree on exactly what the data looks like.
3. **Built the FastAPI backend.** Two endpoints: `GET /health` to confirm the server and API key are configured, and `POST /api/plan` to generate a plan.
4. **Connected Gemini with structured output.** The request passes the Pydantic schema as `response_schema`, so the model returns machine-readable JSON that is parsed and sent to the frontend.
5. **Built the frontend.** A single-page form that calls the API and renders each section of the plan as its own block.
6. **Designed the UI around the content.** Warm cream background, raspberry accent, serif headings (Fraunces) paired with a clean sans-serif body (Work Sans), and a color-coded tag per section so the plan reads like a document rather than a stack of identical cards.
7. **Tested locally end-to-end**, then fixed the setup and API issues listed below.
8. **Deployed** the backend to Render and the frontend to Vercel, and pointed the frontend's `API_BASE` at the live backend URL.

## What I Learned

- How to get **reliable structured output** from an LLM by giving it a schema instead of asking for free-form text, and why that makes the result usable by a real frontend.
- Debugging real-world setup problems: a `pydantic-core` build failure caused by very new Python versions without pre-built packages, fixed by using a supported Python version (3.12) and loosening pinned dependencies.
- **AI model lifecycles**: model names get retired (`gemini-2.0-flash` was shut down in June 2026), so the model is now configurable through a `GEMINI_MODEL` environment variable instead of being hard-coded.
- The difference between **PowerShell and Command Prompt** syntax for environment variables (`$env:KEY="value"` vs `set KEY=value`), and why a silently ignored variable produces confusing errors.
- **Secret handling**: API keys live in environment variables, never in code, screenshots, or commits, and a leaked key should be rotated immediately.
- How a **frontend/backend split** works in practice, including CORS and pointing a static site at a hosted API.

## How It Could Be Improved

- **Save plans with Supabase**: store events and checklist tasks so users can return to a plan and track progress.
- **Regenerate individual sections**: a button per block (theme, food, activities) instead of regenerating the whole plan.
- **Budget optimizer**: a "reduce my budget" action that asks the AI for cheaper alternatives and recalculates.
- **n8n automation**: scheduled reminders as the date approaches, automated invitation sending, and shopping-list nudges.
- **Guest list and RSVP tracking** with automatic headcount-based food adjustments.
- **Calendar integration**: add the party and its schedule to Google Calendar.
- **Safer rendering**: escape AI-generated text before inserting it into the page, and add rate limiting to the API.
- **Export to PDF** so the plan can be printed or shared.

## How to Run This Project

**Requirements:** Python 3.11 or 3.12 and a free Gemini API key from [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey).

1. **Clone the repo**
   ```bash
   git clone https://github.com/YOUR_USERNAME/celebrate-ai.git
   cd celebrate-ai/backend
   ```
2. **Create a virtual environment and install dependencies**
   ```bash
   py -3.12 -m venv venv          # Mac/Linux: python3 -m venv venv
   venv\Scripts\Activate.ps1      # Mac/Linux: source venv/bin/activate
   pip install -r requirements.txt
   ```
3. **Set your API key**
   ```powershell
   $env:GEMINI_API_KEY="your_key_here"     # PowerShell
   # Mac/Linux: export GEMINI_API_KEY=your_key_here
   ```
   Optional: choose a different model with `GEMINI_MODEL` (default is `gemini-3.1-flash-lite`).
4. **Start the backend**
   ```bash
   uvicorn main:app --reload --port 8000
   ```
   Check `http://localhost:8000/health`. It should show `"gemini_configured": true`.
5. **Open the frontend**: double-click `frontend/index.html`.
6. **Test the API directly** (optional) at `http://localhost:8000/docs` with this sample payload:
   ```json
   {
     "name": "Nicole",
     "age": 23,
     "interests": "Barbie, Tinker Bell, Disney",
     "budget": 30000,
     "currency": "PHP",
     "guest_count": 25,
     "location": "Manila",
     "food_preference": "Filipino",
     "theme_preference": "Surprise me"
   }
   ```
7. **Deploying:** host `backend/` on Render (set `GEMINI_API_KEY` as an environment variable), then change `API_BASE` in `frontend/index.html` to your Render URL and host `frontend/` on Vercel or Netlify.

## Project Structure

```
celebrate-ai/
├── backend/
│   ├── main.py            # FastAPI app, Pydantic schema, Gemini call
│   └── requirements.txt
├── frontend/
│   └── index.html         # Form + plan renderer
└── README.md
```

## License

This project is open for learning and reference purposes.
