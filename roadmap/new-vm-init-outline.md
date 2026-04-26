# Project: The Basement Brain
## Mission: Bridging Claude Code to Local Ollama Inference

### 1. Overview
An exploration into self-hosting a professional AI coding agent (Claude Code) on local hardware to achieve zero-cost, privacy-focused software engineering. This project documents the "bypass" of cloud-subscription requirements by redirecting agentic workflows to a local Xeon-powered Ubuntu server.

### 2. Hardware Specs ("The Iron")
* **Host:** VMware Workstation VM
* **OS:** Ubuntu 26.04 (Noble Numbat)
* **CPU:** Intel Xeon (Multi-core allocation)
* **RAM:** 64GB DDR4
* **Storage:** 128GB LVM (XFS logical volume)

### 3. The Software Stack
* **Model:** Qwen 3.6 35B (via Ollama)
* **Agent:** Claude Code CLI (Anthropic)
* **Orchestration:** Custom `.bashrc` environment redirects
* **Model Optimization:** 4-bit quantization for CPU-bound inference (2-3 words/sec)

### 4. Technical Exploits & Key Learnings
* **Environment Hijacking:** Used `ANTHROPIC_BASE_URL` to redirect CLI traffic from `anthropic.com` to `localhost:11434`.
* **Subscription Bypass:** Leveraged an explicitly empty `ANTHROPIC_API_KEY` and the special `ANTHROPIC_AUTH_TOKEN="ollama"` flag to skip Pro/Max licensing checks.
* **Memory Context Management:** Handled the "23GB Load" latency on CPU by pre-warming the model via `ollama run` before initiating agentic tasks.
* **Agent Persistence:** Successfully generated a 42KB `claude.md` architectural map of a local Booking Platform repo, proving the agent could "see" and "analyze" local codebases without cloud access.

### 5. Challenges Overcome
* **Networking:** Resolved Ubuntu Netplan YAML formatting errors and Broadcom adapter driver conflicts.
* **Python Versioning:** Navigated PEP 668 and Python 3.14 (Ubuntu 26.04) compatibility issues by moving to a direct environment variable override strategy rather than a proxy middleman.
* **Model Recognition:** Fixed "Model Not Found" errors by aliasing local model names to match Claude's internal expectations.

### 6. Deployment Command
```bash
# The 'God Mode' Alias used for this setup
alias ai='claude --model qwen3.6:35b --permission-mode bypassPermissions --dangerously-skip-permissions'