# 02 — Model Config

Provider setup, local model configuration via Ollama, and hybrid routing with OpenRouter.

---

## Contents

1. Billing — pay-as-you-go vs. subscription credits
2. 2. API provider configuration (Claude, GPT-4o, Gemini)
   3. 3. Local models with Ollama
      4. 4. Hybrid routing via OpenRouter
         5. 5. Model selection strategies
           
            6. ---
           
            7. ## 1. Billing — pay-as-you-go vs. subscription credits
           
            8. API key required — subscription credits do not work.
           
            9. As of April 4, 2026, Claude Pro and Max subscription credits no longer work with third-party tools including OpenClaw. A pay-as-you-go API key is required. The same applies to OpenAI — ChatGPT Plus subscriptions do not carry over to API access.
           
            10. | Provider | Pay-as-you-go portal | Notes |
            11. |----------|---------------------|-------|
            12. | Anthropic | console.anthropic.com | Separate billing from Claude.ai subscription |
            13. | OpenAI | platform.openai.com | Separate billing from ChatGPT Plus |
            14. | Google | aistudio.google.com | Gemini API; free tier available |
           
            15. ---
           
            16. ## 2. API provider configuration
           
            17. ### Claude (Anthropic)
           
            18. 1. Go to [console.anthropic.com](https://console.anthropic.com) and create an API key.
                2. 2. Add billing at console.anthropic.com → Billing.
                   3. 3. Add the key to OpenClaw:
                     
                      4. ```bash
                         openclaw config set model.provider anthropic
                         openclaw config set model.apiKey sk-ant-...
                         ```

                         Or edit `~/.openclaw/openclaw.json` directly:

                         ```json
                         {
                           "model": {
                             "provider": "anthropic",
                             "model": "claude-sonnet-4-5",
                             "apiKey": "sk-ant-..."
                           }
                         }
                         ```

                         Recommended models:

                         | Model | Use case |
                         |-------|----------|
                         | claude-opus-4-5 | Complex reasoning, long documents |
                         | claude-sonnet-4-5 | Balanced — recommended default |
                         | claude-haiku-4-5 | Fast, low cost, simple tasks |

                         ### GPT-4o (OpenAI)

                         1. Go to [platform.openai.com](https://platform.openai.com) and create an API key.
                         2. 2. Add billing under Settings → Billing.
                            3. 3. Configure OpenClaw:
                              
                               4. ```bash
                                  openclaw config set model.provider openai
                                  openclaw config set model.apiKey sk-...
                                  ```

                                  ```json
                                  {
                                    "model": {
                                      "provider": "openai",
                                      "model": "gpt-4o",
                                      "apiKey": "sk-..."
                                    }
                                  }
                                  ```

                                  ### Gemini (Google)

                                  1. Go to [aistudio.google.com](https://aistudio.google.com) and generate an API key.
                                  2. 2. A free tier is available with rate limits; pay-as-you-go billing removes them.
                                     3. 3. Configure OpenClaw:
                                       
                                        4. ```bash
                                           openclaw config set model.provider google
                                           openclaw config set model.apiKey AIza...
                                           ```

                                           ```json
                                           {
                                             "model": {
                                               "provider": "google",
                                               "model": "gemini-2.0-flash",
                                               "apiKey": "AIza..."
                                             }
                                           }
                                           ```

                                           ### Verify the connection

                                           ```bash
                                           openclaw models list
                                           openclaw models test
                                           ```

                                           If `models list` returns empty, the API key is invalid or billing is not configured.

                                           ---

                                           ## 3. Local models with Ollama

                                           ### Why local models

                                           - No API key or billing required
                                           - - Data never leaves your hardware
                                             - - Useful as a fallback when API services are unavailable
                                               - - Required for air-gapped or high-sensitivity deployments
                                                
                                                 - ### Hardware requirements
                                                
                                                 - | Model size | VRAM | Notes |
                                                 - |------------|------|-------|
                                                 - | 7B | 8 GB | Minimum viable; llama3.2, mistral |
                                                 - | 13B | 16 GB | Better reasoning, still fast |
                                                 - | 32B | 24 GB | Good balance of quality and speed |
                                                 - | 70B | 48 GB+ | High quality; requires two GPUs or slow on CPU |
                                                
                                                 - CPU inference works but is significantly slower — acceptable for low-frequency tasks on modern ARM chips (e.g. Apple M-series), not recommended on x86.
                                                
                                                 - ### Install Ollama
                                                
                                                 - ```bash
                                                   curl -fsSL https://ollama.ai/install.sh | sh
                                                   ```

                                                   Windows and macOS installers are available at [ollama.ai](https://ollama.ai).

                                                   ### Pull a model

                                                   ```bash
                                                   ollama pull llama3.2
                                                   ollama pull mistral
                                                   ollama pull deepseek-r1:8b
                                                   ```

                                                   Verify it runs:

                                                   ```bash
                                                   ollama run llama3.2 "Say hello."
                                                   ```

                                                   ### Recommended models

                                                   | Model | Size | Strengths |
                                                   |-------|------|-----------|
                                                   | llama3.2 | 3B / 8B | Fast, general purpose, good instruction following |
                                                   | mistral | 7B | Strong reasoning, low resource use |
                                                   | deepseek-r1 | 8B / 32B | Excellent at multi-step reasoning and code |
                                                   | qwen2.5 | 7B / 32B | Strong multilingual support |
                                                   | phi4 | 14B | High quality for size; good on constrained hardware |

                                                   ### Connect Ollama to OpenClaw

                                                   ```bash
                                                   openclaw config set model.provider ollama
                                                   openclaw config set model.ollamaModel llama3.2
                                                   ```

                                                   ```json
                                                   {
                                                     "model": {
                                                       "provider": "ollama",
                                                       "ollamaHost": "http://127.0.0.1:11434",
                                                       "ollamaModel": "llama3.2"
                                                     }
                                                   }
                                                   ```

                                                   If Ollama is running on a different machine (e.g. a desktop GPU rig), replace `127.0.0.1` with its LAN IP and ensure port 11434 is accessible.

                                                   ---

                                                   ## 4. Hybrid routing via OpenRouter

                                                   OpenRouter provides a single API key that routes to multiple model providers — Anthropic, OpenAI, Google, Mistral, Meta, and dozens of others. It is useful when you want to switch models without managing separate keys or when you want automatic fallback.

                                                   ### Setup

                                                   1. Create an account at [openrouter.ai](https://openrouter.ai).
                                                   2. 2. Add credits under Billing.
                                                      3. 3. Generate an API key.
                                                         4. 4. Configure OpenClaw:
                                                           
                                                            5. ```bash
                                                               openclaw config set model.provider openrouter
                                                               openclaw config set model.apiKey sk-or-...
                                                               openclaw config set model.model anthropic/claude-sonnet-4-5
                                                               ```

                                                               ```json
                                                               {
                                                                 "model": {
                                                                   "provider": "openrouter",
                                                                   "apiKey": "sk-or-...",
                                                                   "model": "anthropic/claude-sonnet-4-5"
                                                                 }
                                                               }
                                                               ```

                                                               ### Fallback routing

                                                               OpenRouter supports automatic fallback if a primary model is unavailable or rate-limited:

                                                               ```json
                                                               {
                                                                 "model": {
                                                                   "provider": "openrouter",
                                                                   "apiKey": "sk-or-...",
                                                                   "model": "anthropic/claude-sonnet-4-5",
                                                                   "fallbackModels": [
                                                                     "openai/gpt-4o",
                                                                     "google/gemini-2.0-flash"
                                                                   ]
                                                                 }
                                                               }
                                                               ```

                                                               ### Cost visibility

                                                               OpenRouter shows per-request costs in the dashboard at openrouter.ai/activity. This is useful for monitoring spend across providers from a single view.

                                                               ---

                                                               ## 5. Model selection strategies

                                                               No single model is best for every task. The right choice depends on cost, latency, capability requirements, and privacy constraints.

                                                               ### Decision guide

                                                               | Priority | Recommended approach |
                                                               |----------|---------------------|
                                                               | Lowest cost | Gemini 2.0 Flash (API free tier) or local Ollama model |
                                                               | Lowest latency | claude-haiku-4-5, gpt-4o-mini, or local 7B model on fast hardware |
                                                               | Best reasoning | claude-opus-4-5, gpt-4o, or deepseek-r1:32b locally |
                                                               | Privacy / air-gap | Ollama only — no data leaves the machine |
                                                               | Maximum flexibility | OpenRouter with fallback chain |

                                                               ### Cost vs. capability tradeoffs

                                                               Frontier models (Opus, GPT-4o) cost roughly 10–30× more per token than their smaller counterparts (Haiku, GPT-4o-mini) and do not perform 10–30× better on most everyday tasks. Reserve them for tasks that genuinely require deep reasoning — long-document analysis, complex code generation, multi-step planning — and route simpler tasks to cheaper models.

                                                               Local models have zero marginal cost after hardware. If you have a GPU with 8 GB+ VRAM, a 7B or 8B model handles most conversational and summarisation tasks acceptably.

                                                               ### Switching models at runtime

                                                               ```bash
                                                               openclaw config set model.model claude-haiku-4-5
                                                               openclaw gateway restart
                                                               ```

                                                               Or temporarily via the CLI without editing config:

                                                               ```bash
                                                               openclaw run --model openai/gpt-4o "Summarise this document."
                                                               ```

                                                               ### Verify active model

                                                               ```bash
                                                               openclaw models current
                                                               ```

                                                               ---

                                                               ## Where to go next

                                                               [03-skills-plugins](../03-skills-plugins/) — vetting and installing ClawHub skills safely.
                                                               [04-prompt-injection](../04-prompt-injection/) — understanding and mitigating prompt injection for agentic systems.
                                                               [05-network-hardening](../05-network-hardening/) — if you are on Path B (cloud VPS), complete network hardening before using OpenClaw for anything sensitive.
