# Mac AI Performance Healthcheck

An AI-powered macOS performance diagnostic tool that triages system issues, generates targeted capture scripts, and produces a comprehensive HTML optimization report — all automatically.

## What It Does

This tool runs a **5-phase pipeline**:

1. **Triage** — Collects a fast local snapshot of CPU, memory, startup items, disk usage, memory pressure, thermal state, and LaunchAgents.
2. **AI Pass 1 (Planner)** — Analyzes the triage data and generates a custom Bash script targeting the specific processes causing issues.
3. **Execution** — Runs the AI-generated capture script with enforced output limits.
4. **AI Pass 2 (Analyst)** — Analyzes the deep capture output and generates a styled HTML report with actionable fix commands and per-item impact estimates.
5. **Launch** — Opens the report in your default browser.

## Sample Output

The generated HTML report includes:

- Executive summary of performance bottlenecks
- Quick wins with copy-paste terminal commands
- Top resource bottlenecks table (CPU%, MEM%, severity)
- Startup item audit with disable/remove recommendations
- Memory, disk, Spotlight, and thermal analysis
- Process-specific deep dives
- Prioritized optimization roadmap

## Requirements

- **macOS** (uses macOS-specific system commands)
- **Python 3.9+**
- **An API key for one of the supported providers:**
  - **Anthropic** (Claude Opus 4.8 — the default), or
  - **OpenAI** (GPT-4o)

## Provider Selection

The tool works with either **Anthropic (Claude)** or **OpenAI (GPT)**. It picks a
provider automatically based on which API key is set, preferring Anthropic when
both are present. You can always force a choice with `--provider` or the
`PROVIDER` env var.

| Situation | Provider used |
|-----------|---------------|
| `--provider` / `PROVIDER` is set | That provider (explicit) |
| Only `ANTHROPIC_API_KEY` is set | Anthropic |
| Only `OPENAI_API_KEY` is set | OpenAI |
| Both keys set, no flag | Anthropic (default) |

## Installation

```bash
# Clone the repository
git clone https://github.com/alkari/mac-performance-triage.git
cd mac-performance-triage

# Install dependencies (bundles both provider SDKs)
pip install -r requirements.txt

# Set the API key for whichever provider you want to use
export ANTHROPIC_API_KEY="your_api_key_here"   # Claude (default)
# or
export OPENAI_API_KEY="your_api_key_here"      # GPT
```

## Usage

```bash
# Run with defaults (auto-detect provider; Claude/claude-opus-4-8 if available)
python3 mac-ai-healthcheck.py

# Force the OpenAI provider
python3 mac-ai-healthcheck.py --provider openai

# Pick a specific model (uses the matching provider's SDK)
python3 mac-ai-healthcheck.py --model claude-sonnet-4-6
python3 mac-ai-healthcheck.py --provider openai --model gpt-4o-mini

# Save reports to a specific directory
python3 mac-ai-healthcheck.py --out-dir ~/Desktop/reports
```

### Command-Line Options

| Flag | Description | Default |
|------|-------------|---------|
| `--provider` | AI provider: `anthropic` or `openai` | auto-detect (prefers Anthropic) |
| `--model` | Model to use | provider default (`claude-opus-4-8` / `gpt-4o`) |
| `--out-dir` | Directory for output files | `.` (current directory) |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | Your Anthropic API key (required to use the Anthropic provider) |
| `OPENAI_API_KEY` | Your OpenAI API key (required to use the OpenAI provider) |
| `PROVIDER` | Force a provider (`anthropic` or `openai`) without using `--provider` |
| `ANTHROPIC_MODEL` | Override the default Anthropic model |
| `OPENAI_MODEL` | Override the default OpenAI model |

## Output Files

Each run produces three timestamped files:

| File | Description |
|------|-------------|
| `dynamic_capture_<timestamp>.sh` | AI-generated targeted capture script |
| `capture_output_<timestamp>.txt` | Raw output from the capture script |
| `final_analysis_<timestamp>.html` | Full HTML optimization report |

## How It Works

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Local Triage   │────▶│  AI Plans Script │────▶│  Execute Script │
│  (ps, vm_stat,  │     │  (targeted bash) │     │  (bounded I/O)  │
│   disk, thermal)│     └──────────────────┘     └────────┬────────┘
└─────────────────┘                                       │
                                                          ▼
                         ┌──────────────────┐     ┌─────────────────┐
                         │  Open in Browser │◀────│  AI Generates   │
                         │                  │     │  HTML Report    │
                         └──────────────────┘     └─────────────────┘
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `ANTHROPIC_API_KEY not set` | Run `export ANTHROPIC_API_KEY="sk-ant-..."` (or use `--provider openai` with `OPENAI_API_KEY`) |
| `OPENAI_API_KEY not set` | Run `export OPENAI_API_KEY="sk-..."` (or use the Anthropic provider) |
| `anthropic` / `openai` module not found | Run `pip install -r requirements.txt` |
| Script timeout | Some system commands may be slow; retry or check Activity Monitor |
| Large capture output warning | Normal for busy systems; output is auto-truncated |

## Security & Privacy

- All diagnostics run **locally** on your Mac.
- Process names and system stats are sent to your chosen AI provider's API (Anthropic or OpenAI) for analysis.
- No data is stored remotely beyond that provider's standard API data handling.
- Review the generated `.sh` script before running if you prefer manual control.

## License

[MIT](LICENSE)

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.
