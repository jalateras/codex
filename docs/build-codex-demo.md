# build‑codex‑demo Code Walk‑through

This document explains everything that lives in
`codex-cli/examples/build-codex-demo/`. Compared to the much larger
`prompt‑analyzer` example, this demo is purposely **minimal** — it contains no
ready‑made frontend or backend code. Instead, it is a _template task_ that you
launch with **Codex CLI** and then **guide the AI** to iteratively build the
desired project inside version‑controlled _run_ directories.

---

## 1. Directory layout

```
build-codex-demo/
├── run.sh            # Helper that spins up a fresh run and launches Codex
├── runs/             # Auto‑generated workspaces (git‑ignored)
└── task.yaml         # The prompt that describes what we want to build
```

There is **no `template/` folder** because we start from a clean slate each
time — Codex is expected to scaffold the project from scratch based on the
requirements in _task.yaml_.

---

## 2. The orchestration script (`run.sh`)

The Bash helper is functionally identical to the one found in
`examples/prompt-analyzer/`, so only the key mechanics are repeated here:

1.  Navigate into `runs/` — this keeps generated files out of version control.
2.  Determine the next `run_N` directory and ask for confirmation (or skip with
    `--auto-confirm`).
3.  Optionally copy a `template/` folder if it exists (it doesn’t in this
    example → no‑op).
4.  Read `task.yaml.description`, forward it to the `codex` CLI which spawns a
    chat session.

Effectively `run.sh` is the _“create a fresh workspace then hand over to the
model”_ button.

---

## 3. Understanding `task.yaml`

`task.yaml` is the **sole specification** of the product we want the AI to
build. It contains two top‑level keys:

- `name` – identifier string (`build-codex-demo`).
- `description` – a **rich, multi‑line prompt** enumerating functional and
  non‑functional requirements.

### 3.1 High‑level objective

> “Re‑implement the original OpenAI Codex demo.”

The original demo showed how Codex could turn natural‑language prompts into
fully rendered web pages. The task recreates that experience with modern
OpenAI streaming APIs and a thin local backend.

### 3.2 Frontend requirements

- **Prompt bar** at the bottom – user enters a request, hits _Enter_ or clicks
  a green arrow.
- **Conversation history** must be preserved across turns (user & assistant
  messages).
- **Streaming UX** – tokens arrive progressively, are appended to a _code
  viewer_ **with syntax highlighting** and **line numbers**.
- The _output canvas_ shows the **previous** HTML preview until the new
  response is **fully streamed**, then switches atomically.
- Scroll behaviour constrained to the code window & output canvas only.

### 3.3 Backend requirements

- Simple **Node.js** server.
- Reads `OPENAI_API_KEY` from the environment.
- Exposes a single endpoint that forwards the prompt to `openai.chat.completions.create`
  (stream = true) and **re‑streams** the tokens to the frontend.
- Keep dependencies _minimal_ — think `express` + `openai` SDK and maybe
  `cors`.

### 3.4 System prompt & API example

The YAML includes a full JavaScript snippet that illustrates how to call the
OpenAI SDK:

```js
const response = await openai.responses.create({
  model: "gpt-4.1",
  input: [
    {
      role: "system",
      content: [
        {
          type: "input_text",
          text: "You are a coding agent that specializes in frontend code. Whenever you are prompted, return only the full HTML file.",
        },
      ],
    },
  ],
  text: { format: { type: "text" } },
  stream: true,
});
```

Key instructions embedded in the description:

- The assistant **must return only HTML**, always wrapped in ```html fences —
  the frontend must **strip** those before display.
- Errors should be surfaced gracefully to the user.

### 3.5 UI/UX design cues

The prompt meticulously describes panel sizes, colours, scrolling behaviour and
overall minimal aesthetic so that the generated code matches the original
Codex demo look & feel.

---

## 4. How the pieces fit together during a _run_

1. **Developer executes** `./run.sh` → a pristine `runs/run_N` folder is
   created; Codex chat window pops up.
2. **Codex interprets** the long `description` and begins generating files:
   - `backend/index.js` (or similar)
   - `frontend/index.html`, `style.css`, `main.js`, …
   - `README.md` with run instructions
3. **User iterates**: reviews output, asks Codex to fix bugs, refine UX, add
   features, etc. — all within the isolated run directory so previous attempts
   remain intact.

Because the repository ships with **zero starting code**, the example doubles
as a _tutorial_ on how to leverage Codex CLI for green‑field prototyping.

---

## 5. Extending the demo

Once the baseline functionality works you could ask Codex to:

- Add **authentication** so multiple users can run their own sessions.
- Support **CSS/JS** files streaming & live preview, not just HTML.
- Persist conversation history to **localStorage** or a database.
- Deploy the backend as a **Docker** container or to cloud run‑times like
  Vercel / Fly.io.
- Integrate **unit tests** (Jest, Playwright) to guard against regressions
  during iterative prompting.

---

## 6. Recap

`examples/build-codex-demo` is more of a **launchpad** than a ready‑to‑run app.
It shows how a single, well‑crafted YAML description plus a tiny Bash helper
can bootstrap an entire interactive development session with Codex CLI. All
actual source code will be generated, reviewed and refined _inside_ the
numbered run directories.

Use it as inspiration for your own green‑field ideas — just edit
`task.yaml.description`, run the script and let Codex help you build! 🚀
