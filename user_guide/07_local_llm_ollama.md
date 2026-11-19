# Using a local model (Ollama)

Agentic-J can talk to any OpenAI-compatible Chat Completions server instead of
OpenAI / OpenRouter. This was tested with [Ollama](https://ollama.com) on Windows 11
(NVIDIA RTX A4000, 16 GB) with `qwen3:14b`, running on the host while Agentic-J runs in Docker.

This is experimental: a local 14B model is much weaker than the hosted defaults.

## 1. Ollama on the host

```
ollama pull qwen3:14b
setx OLLAMA_NUM_PARALLEL 1
setx OLLAMA_CONTEXT_LENGTH 32768
setx OLLAMA_FLASH_ATTENTION 1
setx OLLAMA_KV_CACHE_TYPE q8_0
```

Restart Ollama afterwards (quit from the tray icon and start it again) so the variables apply.
Check with `ollama ps`: it should show `CONTEXT 32768` and `100% GPU`.

The context matters: the agent's system prompt plus tool definitions is about 22,000 tokens.
With Ollama's smaller default (or a context split across parallel requests) the prompt is
silently truncated (`truncating input prompt` in the Ollama log) and the agent loops without progress.

## 2. Point Agentic-J at it

In `.env`:

```
LOCAL_LLM_BASE_URL=http://host.docker.internal:11434/v1
```

(`LOCAL_LLM_API_KEY` is optional; it defaults to `ollama`.)

In `imagentj_config.yaml`, set every role to a model that Ollama serves and that supports tool calling:

```yaml
models:
  supervisor: "qwen3:14b"
  worker:     "qwen3:14b"
  analyst:    "qwen3:14b"
  nano:       "qwen3:14b"
  curator:    "qwen3:14b"
  vlm:        "qwen3:14b"
```

Then `docker compose up -d --force-recreate`.

If the container cannot reach Ollama, set `OLLAMA_HOST=0.0.0.0` on the host and restart Ollama.

## Known limitations

- Documentation search (RAG) still uses OpenAI `text-embedding-3-large` and needs an OpenAI/OpenRouter key.
- The Vision (VLM) judge is not available without one of those keys.
- When `LOCAL_LLM_BASE_URL` is set it takes precedence over `OPEN_ROUTER_API_KEY` and `OPENAI_API_KEY` for the chat models.
