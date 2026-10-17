# ➿ harness-loop

> **Autonomous multi-agent software engineering template.**  
> A plug-and-play, zero-token-communication loop pairing high-frequency local/free worker harnesses (OpenCode, Aider) with high-tier lead architect harnesses (Antigravity Pro, Claude Code).

---

## ⚡ Highlights

- **Zero-Token Coordination**: No conversational history, transcripts, or scratchpads are exchanged between harnesses. Agents communicate strictly through minimal filesystem artifacts (`state.json`, `task.md`, and `feedback.log`).
- **Standard Library Only**: Built with Python 3 standard library (`subprocess`, `shlex`, `json`, `argparse`, `tempfile`, `pathlib`). **Zero third-party `pip` dependencies required** to run the orchestration loop.
- **Deterministic Test Execution**: Worker harnesses are strictly prohibited from invoking test suites. The Python runner executes test commands directly via `subprocess` and captures strictly the **tail 40 lines** of failure output to conserve token context.
- **Plug-and-Play Setup**: One-command interactive CLI wizard (`python runner.py --setup`) configured with pre-tuned presets for leading CLI coding tools.
- **Automatic Escalation**: If the free/local worker fails to satisfy tests after a configurable retry budget (`max_free_retries`), the loop automatically escalates to the Lead Architect to diagnose root causes or modify architecture.

---

## 📂 Repository Structure

```
harness-loop/
├── .agent_loop/
│   ├── .gitignore             # Ignores runtime state (state.json, task.md, feedback.log, config.json)
│   ├── .gitkeep               # Preserves loop directory in version control
│   ├── config.example.json    # Example configuration with multi-engine presets
│   └── rules.md               # Master system instructions for both harnesses
├── src/
│   └── .gitkeep               # Project source code directory
├── tests/
│   └── .gitkeep               # Project test suites directory
├── runner.py                  # Agnostic loop engine + interactive setup wizard
├── README.md                  # Documentation on setup, presets, and architecture
├── .gitignore                 # Standard Python / polyglot gitignore
└── LICENSE                    # MIT License
```

---

## 🚀 Quickstart

### 1. Clone into your new project
```bash
git clone https://github.com/sthaarwin/Harness-Loop.git my-feature-project
cd my-feature-project
```

### 2. Run the interactive setup wizard
```bash
python runner.py --setup
```

The wizard will prompt you to:
1. Select an agent preset (or define custom commands).
2. Choose your model IDs (or accept tuned defaults).
3. Specify your project's test command (auto-detects `pytest`, `cargo test`, `npm test`).
4. Set the retry budget before escalating to the Lead Architect.
5. (Optional) Define your initial task objective.

### 3. Start the autonomous loop
```bash
python runner.py
```

The loop runs autonomously:
- Lead formulates the specification in `.agent_loop/task.md` and scopes `target_files` in `.agent_loop/state.json`.
- Worker edits `target_files`.
- Runner executes your test suite deterministically.
- On test pass, Lead reviews and signs off.
- On test failure, runner captures the tail 40 lines of errors and re-triggers Worker (or escalates to Lead if retries expire).

---

## 🔄 The Zero-Token Communication Bus

```
                       ┌─────────────────────────┐
                       │       User / Goal       │
                       └────────────┬────────────┘
                                    │
                                    ▼
                       ┌─────────────────────────┐
                       │      LEAD ARCHITECT     │
                       │ (Antigravity Pro/Claude)│
                       └────────────┬────────────┘
                                    │ Writes task.md & target_files
                                    ▼
       ┌───────────────────► .agent_loop/state.json ◄──────────────────┐
       │                     .agent_loop/task.md                       │
       │                     .agent_loop/feedback.log                  │
       │                                │                              │
       │                                ▼                              │
       │                       ┌───────────────────┐                   │
       │                       │  WORKER HARNESS   │                   │
       │                       │ (OpenCode / Aider)│                   │
       │                       └────────┬──────────┘                   │
       │                                │ Edits target_files           │
       │                                │ (NO TESTS ALLOWED)           │
       │                                ▼                              │
       │                       ┌───────────────────┐                   │
       │                       │  PYTHON RUNNER    │                   │
       │                       │ (subprocess.run)  │                   │
       │                       └────────┬──────────┘                   │
       │                                │                              │
       │                        [ Tests Pass? ]                        │
       │                         /           \                         │
       │                  No    /             \    Yes                 │
       │                       ▼               ▼                       │
       │            Extract Tail 40 lines    Set status: "ready_for_review"
       │            Iteration += 1           Set active_turn: "lead"   │
       │                       │                       │               │
       │            [ Iter >= max_retries? ]           ▼               │
       │             /                  \        ┌─────────────┐       │
       │       No   /                    \  Yes  │ Lead Review │       │
       └───────────┘                      └─────►│  & Sign-off │───────┘
                                                 └──────┬──────┘
                                                        │ If approved
                                                        ▼
                                                 ┌─────────────┐
                                                 │   DONE ✅   │
                                                 └─────────────┘
```

---

## 🎛️ Presets & Supported Engines

Select from built-in presets during setup or configure custom CLI harnesses in `.agent_loop/config.json`:

| Preset | Worker Harness (Local / Free) | Lead Architect (High-Tier) | Best Used For |
|---|---|---|---|
| **1** | **OpenCode** (`deepseek-coder:6.7b`) | **Antigravity Pro** (`gemini-1.5-pro`) | Maximum cost efficiency with Google ecosystem |
| **2** | **Aider** (`ollama/qwen2.5-coder:7b`) | **Claude Code** (`claude-3-7-sonnet`) | Local Ollama editing + Anthropic reasoning |
| **3** | **OpenCode** (`deepseek-coder:6.7b`) | **Claude Code** (`claude-3-7-sonnet`) | Free local worker + Anthropic architect |
| **4** | **Custom Harness** | **Custom Harness** | Any CLI agent or internal company wrapper |

### Command Template Placeholders

When defining custom harness commands in `.agent_loop/config.json`, use these template placeholders:

- `{prompt}`: Formatted role instructions and file pointers.
- `{model}`: Configured model identifier (e.g. `deepseek-coder:6.7b`, `claude-3-7-sonnet`).
- `{dir}`: Absolute path to the repository root.
- `{session}`: Unique loop session identifier (e.g. `loop-1773738000`).

Example custom configuration:
```json
{
  "worker": {
    "name": "CustomWorker",
    "command": "my-agent run --dir {dir} --model {model} --prompt {prompt}",
    "model": "local-coder"
  },
  "lead": {
    "name": "CustomLead",
    "command": "lead-cli -p {prompt}",
    "model": "cloud-reasoner"
  }
}
```

---

## 💡 Token-Saving Philosophy

1. **Strict Separation of Concerns**:
   - The Lead never writes boilerplate code; it specifies architecture and reviews results.
   - The Worker never performs broad strategic planning; it executes against targeted files.
2. **Deterministic Test Execution (Zero Worker Test Tokens)**:
   - Worker harnesses are forbidden from running `pytest`, `npm test`, or `cargo test`.
   - Running test suites inside an LLM's tool loop wastes tokens on parsing compilation logs, environment checks, and multi-line stacks. `runner.py` runs tests natively in milliseconds.
3. **Tail 40 Error Clipping**:
   - Full compiler and test traces can easily exceed 5,000 tokens of noisy traceback.
   - `runner.py` slices strictly the **tail 40 lines** of combined stdout/stderr into `feedback.log`, delivering only actionable assertions and failure locations to the Worker.
4. **Clean Session Boundary**:
   - Each turn is spawned as an isolated process. There is no cumulative conversational context drift.

---

## 🛠️ CLI Reference

```bash
# Start autonomous continuous loop
python runner.py

# Launch interactive setup wizard
python runner.py --setup

# Execute a single turn (Lead or Worker + tests) and exit
python runner.py --step

# Check current loop state, active turn, target files, and test status
python runner.py --status

# Reset state.json and clear feedback.log
python runner.py --reset

# Directly update the task specification in .agent_loop/task.md
python runner.py --task "Add token bucket rate limiter to src/limiter.py"

# Run project test command directly and update feedback.log without LLMs
python runner.py --test-only

# Limit loop cycles (default: 50)
python runner.py --max-cycles 20
```

---

## 📋 Role Contracts Summary

### 🏛️ Lead Architect (`rules.md`)
- **Spec Formulation**: Defines requirements in `.agent_loop/task.md`.
- **Target File Scoping**: Declares `target_files` in `.agent_loop/state.json`.
- **Escalation Triage**: When `escalate_to_lead: true`, inspects `feedback.log` to fix flawed tests or specs.
- **Review & Approval**: Reviews passed code and marks `active_turn: "done"`.

### 🔨 Worker (`rules.md`)
- **Implementation Only**: Edits ONLY files listed in `target_files`.
- **Feedback Remediation**: Reads `feedback.log` to fix errors.
- **Forbidden Actions**:
  - ❌ Never run test commands manually.
  - ❌ Never modify `state.json`.

---

## 📄 License

Distributed under the [MIT License](LICENSE).
