# ISCAD Chat Assistant

The ISCAD editor contains a chat assistant. You describe in plain language what you want to model or change, and a language model edits the model script, rebuilds the model, reads the error messages and corrects the script, much as a software developer works on source code. The assistant can also inspect the built geometry (dimensions, volume, named values) and look at the 3D view.

The assistant works with two kinds of language models:

-   **Claude** by Anthropic, through the Anthropic API (requires an API key; usage is billed per token).
-   **Any server with an OpenAI compatible interface**, in particular local models running on your own computer (Ollama, LM Studio, llama.cpp server, vLLM) or hosted services.

No training of the model is required. The assistant receives the ISCAD language documentation as part of its instructions and learns from the error messages of the rebuilds.

## Usage

Every model window has an **Assistant** tab next to the Notepad (right of the script editor). Each model window has its own conversation.

-   Type your request into the input field and press **Enter** (Shift+Enter starts a new line).
-   While the assistant works, its progress is shown in the conversation: text, tool calls (e.g. `edit_script ✓`, `rebuild ✗` with the error message) and images of the 3D view.
-   The script editor is read-only during this time. **Stop** interrupts the assistant.
-   All changes of one request form **a single undo step**: Ctrl+Z in the script editor reverts everything the assistant changed for the last request.
-   **New chat** starts a new conversation (the script is not changed).
-   Below the conversation, the token usage and the estimated cost of the last request and of the whole conversation are displayed (the cost only for models with known prices).

You can continue to work on the script yourself between requests. The assistant is told when the script was changed and reads it again.

Examples of requests:

```text
Create a flange named `flange`: disc 160 mm diameter, 20 mm thick, bore 60 mm,
6 holes of 14 mm on a bolt circle of 120 mm.

Make the plate 150 mm long and add a second hole 25 mm from the right end.

The model does not build. Please fix it.

Replace the numbers by parameters with ?= so that the model can be used as a submodel.

What is the volume of `body` in cm³ and where is its center of gravity?
```

Hints:

-   Give names for new features ("named `plate`") and state positions and directions explicitly (e.g. "bottom face at z=0", "along +X"). Dimensions are in the units of the script (usually millimeters).
-   Ask the assistant to check its result ("check the dimensions", "show me the part from the top"). It then inspects the built feature or renders the 3D view.
-   For larger changes, describe one step at a time. Ambiguous requests (which face, which direction) are answered with a question.

### What the assistant can do

The assistant works with the following tools. It decides itself, which ones it uses:

| Tool | Function |
|---|---|
| `read_script` | read the script (or a range of lines) from the editor |
| `edit_script` | replace a unique text passage of the script |
| `write_script` | replace the complete script (only for new models or complete rewrites) |
| `rebuild` | rebuild the model (or the script up to a line), returns errors with line and column. Postprocessing actions which write files are not executed. |
| `list_commands`, `command_help` | signature and documentation of the feature commands |
| `list_library_models`, `read_library_model` | models of the ISCAD model library, used as examples |
| `list_symbols` | all scalars, vectors, datums and features of the built model with their values |
| `inspect_feature` | topology, volume, surface area, bounding box, center of gravity, subfeatures and named values of a built feature |
| `view_model` | renders the 3D view from a given direction (optionally with a highlighted feature) and returns the image. Your own view is restored afterwards. Only available for models which accept images. |

## Settings

The button **Settings...** below the conversation opens the configuration. The settings are stored in the user settings and apply to all model windows; changes take effect for the next request of the current conversation.

### Anthropic (Claude)

| Setting | Description |
|---|---|
| Model | e.g. `claude-opus-5-5` (default, most capable), `claude-sonnet-5-5` (about half the price), `claude-haiku-5-5` (cheapest, weaker in multi-step editing) |
| Effort | how thoroughly the model works: low, medium, high (default), xhigh, max. Higher effort uses more tokens. |
| API key | the key from the Anthropic Console |

To use Claude, you need an account in the [Anthropic Console](https://platform.claude.com) and an API key. API usage is billed per token, a Claude Pro or Max subscription does not include it. As a rough estimate, a typical request (several edit and rebuild steps) costs about 0.10–0.30 USD with Sonnet and about twice as much with Opus. The cost display in the assistant shows the actual usage.

The environment variable `ANTHROPIC_API_KEY` takes precedence over the key entered in the dialog. Note that a key entered in the dialog is stored **unencrypted** in the user settings.

### OpenAI compatible server (local models)

| Setting | Description |
|---|---|
| Server | presets for the base URL: Ollama, LM Studio, vLLM, llama.cpp server, OpenAI |
| Base URL | URL of the server including the version path, e.g. `http://localhost:11434/v1` |
| Model | the model name as known to the server. **Fetch models** queries the list of available models. |
| API key | optional (the environment variable `OPENAI_API_KEY` takes precedence) |
| Max. output tokens | limit per response, 0: server default |
| Prices | USD per million input, cached input and output tokens, for the cost display (leave at 0 for local models) |
| The model accepts images | enables the `view_model` tool. Only check this for models with image input, otherwise the requests fail. |

The model must support tool calling (function calling).

### Common settings

| Setting | Description |
|---|---|
| System prompt | how much of the ISCAD documentation is given to the model: the full language and command reference (about 20000 tokens), the language reference without the command list (about 10000 tokens), or none (the model then relies on `command_help`) |
| Tools | all tools, or a basic set (read, edit, write, rebuild, command help, inspect) for small models, which are confused by many tools |

With Claude, use the full reference: it is cached after the first request and costs little. For local models with a small context window, use the shorter reference.

## Local Models

Local models keep the model script and all other data on your computer. They are considerably weaker than large hosted models in multi-step editing, so check their results, and use the evaluation described below to find out, which tasks they handle reliably.

### Model Selection

The model has to fit into the memory of the graphics card together with the context (system prompt, script, conversation). Candidates (as of end of 2026):

| Hardware | Recommendation |
|---|---|
| 6–8 GB GPU, at least 32 GB RAM | Mixture-of-experts model with the experts in system RAM: **Qwen3-Coder 30B-A3B** (Q4_K_M, about 19 GB) or **gpt-oss-20b** (about 13 GB) |
| 6–8 GB GPU, less RAM | **Qwen3 8B** (Q4_K_M, about 5 GB) entirely on the GPU, with a short system prompt (reference "none") and the basic tool set |
| 24–32 GB GPU | **Qwen3.6 35B-A3B** or **Devstral Small 2** |

Mixture-of-experts models have many parameters but only use a small part of them (about 3 billion) per generated token. Therefore, they run at acceptable speed, even if most of their weights reside in system RAM.

Settings in the assistant for small local models: tools **basic**, system prompt **language reference without command list** (or **none** with a context of 8k tokens), images off.

### Ollama

```bash
OLLAMA_FLASH_ATTENTION=1 OLLAMA_KV_CACHE_TYPE=q8_0 OLLAMA_CONTEXT_LENGTH=16384 ollama serve
ollama pull qwen3-coder:30b
```

!!! warning
    Ollama uses a small context window by default and **silently truncates** longer input. The system prompt of the assistant is then cut off, and tool calls fail. Always set `OLLAMA_CONTEXT_LENGTH` (16384 or more). The quantized context cache (`OLLAMA_KV_CACHE_TYPE=q8_0`) halves its memory consumption.

In the assistant settings, select the server preset "Ollama" and use **Fetch models**.

### llama.cpp Server

For mixture-of-experts models on a small GPU, the llama.cpp server is usually faster than Ollama, because it can keep exactly the expert weights in system RAM and everything else on the GPU:

```bash
llama-server -m Qwen3-Coder-30B-A3B-Instruct-Q4_K_M.gguf --jinja \
    -c 16384 -ngl 99 --n-cpu-moe 40 -fa on \
    --cache-type-k q8_0 --cache-type-v q8_0 --port 8080
```

-   `--jinja` is required for tool calling.
-   `--n-cpu-moe` is the number of layers, whose experts stay in system RAM. Decrease it, until the GPU memory is almost full: the more runs on the GPU, the faster.
-   The option names change between llama.cpp versions; check `llama-server --help`.

In the assistant settings, select the server preset "llama.cpp server".

## Data Privacy

With each request, the following data is sent to the configured server: the ISCAD documentation (system prompt), the conversation, the model script (complete or in parts), the results of the tool calls (e.g. error messages, dimensions, names of library models) and, if enabled, images of the 3D view.

When using a hosted service, this data leaves your computer. Check whether this is permitted for the model you are working on. With a local server, all data stays on your computer.

## Evaluation of Models

The program `iscad-assistant-eval` runs a set of modeling tasks with a language model without the graphical editor, builds the resulting scripts and checks the geometry automatically. It is used to compare models, settings and changes of the assistant's instructions.

```bash
# check the task definitions (initial scripts and reference solutions), no model required
iscad-assistant-eval --validate

# Claude (ANTHROPIC_API_KEY must be set)
iscad-assistant-eval --model claude-sonnet-5-5

# local model
iscad-assistant-eval --provider openai --base-url http://localhost:11434/v1 \
    --model qwen3-coder:30b --tools basic --reference basic --timeout 3600 --repeat 2
```

| Option | Description |
|---|---|
| `--tasks <dir>` | task directory (default: `share/insight/iscad-assistant-eval` of the installation, or the environment variable `ISCAD_ASSISTANT_EVAL_TASKS`) |
| `--list` | list the tasks |
| `--validate` | build the initial scripts and the reference solutions and apply the checks to the solutions |
| `--filter <regex>` | run only tasks, whose id or category matches |
| `--provider anthropic\|openai` | type of server |
| `--model`, `--base-url`, `--api-key` | model and server (as in the settings dialog) |
| `--effort <level>` | Anthropic: effort |
| `--max-tokens`, `--price-input`, `--price-output`, `--price-cached` | OpenAI compatible server: output limit and prices |
| `--reference full\|basic\|none` | documentation in the system prompt |
| `--tools full\|basic` | tool set |
| `--repeat <n>` | run every task n times (results vary between runs) |
| `--max-steps <n>` | maximum number of model responses per task (default 40) |
| `--timeout <s>` | maximum time per task (default 900 s; increase for slow local models) |
| `--out <prefix>` | output file names |

The images of the 3D view are not available without the graphical editor; the `view_model` tool is therefore always disabled in the evaluation.

Each run writes two reports: `<prefix>.md` with the pass rate per task category and a table of all tasks (steps, tool calls, tokens, cost, time, failed checks), and `<prefix>.json` with all results including the complete conversations and final scripts. The conversations show, where a model fails, e.g. because of wrong ISCAD syntax or because it loses track of the task.

### Tasks

The supplied tasks cover the categories *create* (new models), *modify* (changes of existing scripts), *fix* (broken scripts), *refactor* and *inspect* (questions about the model). Each task is a JSON file:

```json
{
  "id": "create_tube",
  "category": "create",
  "description": "hollow cylinder",
  "prompt": "Create a tube named `tube` along the Z axis from z=0 to z=200, with an outer diameter of 50 mm and a wall thickness of 5 mm.",
  "script": "",
  "solution": "tube: Cylinder(O, ax 200*EZ, 50, 40);\n",
  "checks": [
    {"type": "builds"},
    {"type": "feature", "feature": "tube", "volume": 141371.669,
     "bbox_size": [50, 50, 200], "bbox_min": [-25, -25, 0]}
  ]
}
```

| Field | Description |
|---|---|
| `id`, `category`, `description` | identification |
| `prompt` | the request to the assistant |
| `script` or `script_file` | the initial script (inline, or a file relative to the task file) |
| `start_builds` | `false` for tasks, whose initial script is broken on purpose |
| `solution`, `solution_answer` | reference solution (script and answer), used by `--validate` |
| `max_steps` | step limit for this task |
| `checks` | list of checks, all must pass |

| Check | Parameters | Passes, if |
|---|---|---|
| `builds` | | the final script builds without error |
| `feature` | `feature` (name or `*` for any feature, `component`: only components), `volume`, `area`, `bbox_size`, `bbox_min`, `bbox_max`, `cog`, `n_solids`, `n_faces` | the feature has all given properties |
| `feature_count` | `min`, `max`, `component` | the number of built features is in range |
| `scalar` | `name`, `value` | the scalar symbol has the value |
| `script_contains`, `script_not_contains` | `text` | the script contains (not) the text |
| `script_matches`, `script_not_matches` | `regex` | the script matches (not) the regular expression |
| `script_unchanged` | | the script was not modified |
| `answer_number` | `value` | the final answer contains the number |
| `answer_contains` | `text` | the final answer contains the text (case insensitive) |

Numeric values are compared with a relative tolerance of 0.5 % and an absolute tolerance of 0.02 (options `rel_tol`, `abs_tol`). Bounding boxes are computed from a tessellation of the geometry and slightly underestimate curved shapes; their tolerance is at least 1 % of the largest dimension (`bbox_rel_tol`). Every check can have a `description`, which is shown in the report.

To add a task, put a new JSON file into the task directory, name the features to be checked in the prompt, add a reference solution and run `--validate`.
