# 🧠 typingmind-reasoning-support

Many reasoning-capable models — including **OpenAI OSS**, **MiniMax M2**, and others — currently underperform in TypingMind when tool calls are involved.
The root cause: TypingMind does not preserve the `reasoning_details` field between requests.
This field is critical for continuity — without it, models “forget” mid-reasoning steps and often fail partial tool invocations.

This lightweight extension fixes that.

---

## ✨ What it does

This extension automatically:

1. **Intercepts all LLM responses** in TypingMind that contain `reasoning_details`.
2. **Caches** those reasoning segments locally.
3. **Restores** them on subsequent turns by injecting the correct `reasoning_details` back into the next request.

It works transparently for both **streaming** and **non-streaming** completions.

---

## 🧩 How it identifies reasoning continuity

Each response is uniquely cached and matched back to the right assistant message through a safe two-step process:

1. **Response ID match:**
   When possible, the model’s `response.id` (from the streamed JSON events) is bound to the assistant message that originated it.
   Subsequent requests use this ID to recover and re-inject the correct `reasoning_details`.

2. **Content-level fallback:**
   If a conversation branch is forked or a message ID changes, the extension falls back to a hash of the message **content + tool calls** to ensure stable mapping.

---

### ✅ Safeguards

* **Message chain starts**: Reasoning continuity begins purely from content — safe and deterministic.
* **Ongoing conversations**: Reasoning data is preserved automatically across steps.
* **Forked conversations**: If a user rewinds and edits a previous turn, mismatched reasoning data is ignored safely.

### ⚠️ Limitations

This does **not** preserve reasoning continuity across:

* Different browsers or devices
* Reloaded TypingMind sessions (reasoning cache is in-memory only)
* Edited or truncated message histories that no longer match ID or content hashes

These constraints are deliberate to prevent reasoning corruption.

---

## ⚙️ Configuration

At the top of `script.js`, there is an array named something like:

```js
const ALLOWED_ENDPOINTS = [
  "https://openrouter.ai/api/v1",
  "https://api.minimax.chat/v1",
];
```

Only requests sent to these endpoints will be patched to preserve reasoning continuity.
You can **add, remove, or change** entries in this array to match your setup (for example, if you self-host OpenRouter or use custom proxy URLs).

---

## 🧪 Installation

No manifest or bundling needed — it’s a single-file TypingMind extension.

1. Fork this repository and adjust any settings (e.g. endpoint list) as needed.

2. Commit your changes, then open your fork on GitHub.

3. Copy the **raw script URL** from your fork, e.g.:

   ```
   https://raw.githubusercontent.com/<your-username>/typingmind-reasoning-support/main/script.js
   ```

4. In TypingMind:

   * Open **Settings → Extensions → Add Extension**
   * Paste that raw URL
   * Click **Install**, then refresh the page

You’ll see a console message confirming:

```
✅ Reasoning Continuity Extension active
```

---

## 🧱 Implementation details

The script works by:

* Hooking into TypingMind’s internal Webpack stream parser (which processes the `response.*` events).
* Capturing `reasoning_details` deltas as they arrive.
* Injecting them back into the next outgoing request payload before the network call is sent.

Both **streamed** and **non-streamed** responses are supported via unified caching logic.

---

## 🤝 Attribution

Built by [**Teja Sunku**](https://github.com/tejasunku)  with help from ChatGPT to improve TypingMind’s support for reasoning-capable models.
If you find it useful, please consider leaving a ⭐ on the repo.

GitHub:
👉 [https://github.com/tejasunku/typingmind-reasoning-support](https://github.com/tejasunku/typingmind-reasoning-support)
