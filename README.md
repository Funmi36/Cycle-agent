# Cycle-agent
Cycle-aware productivity assistant built with LangGraph and Groq. She tells it how she feels, in English or Pidgin, and it proposes task changes. Nothing changes until she approves, and red-flag symptoms trigger a safety check first.

1. Plain code (no AI)

This layer does everything that has to be reliable:

Database (SQLite): period logs, tasks, symptoms, proposals, calendar events.
Cycle logic: get_phase() and log_period_start().
Safety: input_guard(), the red-flag rules and the Pidgin glossary.
Tools that only propose: propose_move_task() and propose_cancel_task(). They write a pending proposal and validate it, but change no task.
Approval gate: approve() and reject() are the only code that changes tasks, and approve() also updates the calendar.

2. Agents (AI, with fallbacks)

Each agent is a small piece that can use the model:

Orchestrator decides which specialist handles the message.
Check-in turns messy text into symptom codes.
Planner proposes task changes using get_tasks and propose_*.
Cycle explains her phase using get_phase and log_period_start.

Each agent only gets its own tools, and none can reach approve(). If there’s no key or no internet, each one falls back to plain rules.

3. LangGraph workflow (the order of steps)
guard → orchestrator → checkin → safety → planner → review → approval → apply
                    ↘ cycle                ↘ stop on red flag    (pauses)  (only writer)
guard: scrubs the text and checks for danger words first.
safety: logs symptoms and checks red flags. If one fires, the graph stops and never reaches the Planner.
review: caps the number of proposals.
approval: the graph pauses until she taps Approve or Reject.
apply: the only step that calls approve() or reject().
How the pieces connect
Model (Claude): does the language work.
LangChain: wraps the model and turns your functions into tools.
LangGraph: runs the steps in order, enforces the pause, and remembers where it stopped.



Input guard: is a python fucntion. Cleans a text by removing sensitive data like phon numbers, email, her name. Scans for warning signs in the text. It takes record even when you are offline because it's a function.

The safety node after checkin: 
1. logs symptoms to the database

What is not an agent

The graph has 10 nodes, and the other 6 are plain code:

guard: scrubs text and sets the quick danger flag.
safety: logs symptoms and applies the red-flag rules.
review: caps proposals at 3.
approval: pauses for her tap.
apply: the only step that changes tasks and the calendar.
other: a fixed reply for off-topic messages.





