---
theme: seriph
background: https://cover.sli.dev
title: OpenClaw Architecture Deep Dive
info: |
  ## OpenClaw Architecture & Implementation
  Deep technical dive into how the agent runtime and gateway are built.
class: text-center
transition: slide-left
duration: 25min
---

# OpenClaw Deep Dive
## Under the Hood of the Agentic OS

How do you build a system that executes real-world tasks securely?

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space to explore the implementation <carbon:arrow-right />
</div>

---
transition: slide-up
---

# Real-world Automation: Chat to Browser

<p class="text-[15px] opacity-75 -mt-2 mb-4">
<strong>No UI needed.</strong> Command OpenClaw natively via <strong>Feishu</strong> and watch it drive the browser autonomously.
</p>

<div class="grid grid-cols-12 gap-6 items-center mt-6">
  <!-- Left Column: The Phone -->
  <div class="col-span-4 flex justify-center">
    <div class="bg-white dark:bg-gray-800/80 p-2 rounded-2xl border border-gray-200 dark:border-gray-700 shadow-xl h-[360px] w-full max-w-[220px] relative">
      <div class="absolute -top-3 -left-3 bg-indigo-500 text-white text-[11px] font-bold px-3 py-1.5 rounded-full shadow-lg z-10 flex items-center gap-1.5">
        <carbon:chat /> 1. Command via Feishu
      </div>
      <img src="./docs/examples/phone.jpg" class="h-full w-full object-contain rounded-xl" alt="Feishu Command" />
    </div>
  </div>

  <!-- Middle: Arrow -->
  <div class="col-span-1 flex justify-center items-center">
    <div v-click class="text-blue-400 dark:text-blue-500 text-4xl transform animate-pulse">
      <carbon:arrow-right />
    </div>
  </div>

  <!-- Right Column: Browser Execution -->
  <div class="col-span-7 flex flex-col">
    <div class="bg-white dark:bg-gray-800/80 p-2 rounded-2xl border border-gray-200 dark:border-gray-700 shadow-xl relative w-full">
       <div class="absolute -top-3 -left-3 bg-teal-500 text-white text-[11px] font-bold px-3 py-1.5 rounded-full shadow-lg z-10 flex items-center gap-1.5">
        <carbon:bot /> 2. Autonomous Action
      </div>
      <img src="./docs/examples/bilibili.png" class="h-[220px] w-full object-cover object-top rounded-xl border border-gray-100 dark:border-gray-600" alt="Bilibili Automation" />
    </div>

  <div v-motion :initial="{ y: 20, opacity: 0 }" :enter="{ y: 0, opacity: 1, transition: { delay: 600 } }" class="mt-6 p-4 bg-blue-50 dark:bg-blue-900/20 rounded-xl border-l-4 border-blue-400 text-[13px] shadow-sm flex items-start gap-4">
    <div class="text-3xl mt-1">🌐</div>
    <div>
      <div class="font-bold text-blue-600 dark:text-blue-400 mb-1">OpenClaw Browser Relay</div>
      <div class="text-gray-700 dark:text-gray-300 leading-relaxed">
        OpenClaw takes over the browser, searches for <em>"Mu Li - Deep Learning"</em> on Bilibili, summarizes the video content, and streams the structured result back to the user.
      </div>
    </div>
  </div>
  </div>
</div>

---
transition: fade-out
layout: image-right
image: ./docs/images/starthistory.svg
# 核心修正 1：强制内容靠顶并减小内边距
class: "pt-8 pb-4"
# 核心修正 2：调整背景图缩放并居中，防止被文字挤掉
backgroundSize: 85%
backgroundRepeat: no-repeat
backgroundPosition: "center right 5%"
---

# The OpenClaw Phenomenon

<p class="text-sm opacity-80 -mt-2 mb-4">
Before diving into the code, it's crucial to understand the context of the project.
</p>

<div class="space-y-2 text-sm">

- 👤 **Creator**: Created by Peter Steinberger (founder of PSPDFKit) as a weekend project named "Clawdbot".
- 📈 **Growth**: Evolved into "OpenClaw", surpassing 250,000 stars in weeks.
- 🤝 **Independence**: After Peter joined OpenAI in Feb 2026, OpenClaw transitioned to an independent foundation.

</div>

<div class="mt-6 border-l-3 border-blue-400 bg-blue-50/10 p-3 rounded">
<p class="text-xs leading-relaxed">
<span class="font-bold text-blue-400">OpenClaw</span> isn't just a chatbot wrapper—it's a <b>structured execution environment</b> acting as an Operating System for LLMs.
</p>
</div>

<style>
/* 强制收紧列表间距 */
ul {
  @apply list-none p-0 m-0;
}
li {
  @apply mb-2 leading-snug;
}
h1 {
  @apply text-3xl font-bold mb-2;
}
</style>

---
transition: slide-up
layout: two-cols
# 核心修正 1：强制顶部对齐
class: "pt-8"
---

# The Gateway implementation

<div class="pr-4 text-sm flex flex-col gap-3">

<div>
The Gateway (`src/gateway/server.ts`) is the single source of truth for the entire system, built on **Node.js 22** using the `ws` WebSocket library.

- 🔒 Binds to `127.0.0.1:18789` loopback by default.
- 🚦 Uses an **event-driven RPC model**, strictly typed via TypeBox and JSON Schema validation.
- 🔑 Implements **Idempotency Keys** for UI network retries, preventing duplicate tool executions.
</div>

<div class="border-t border-gray-200 dark:border-gray-700 pt-3 text-xs">

**Implementation Flow**

1. **Client sends an action** over WebSocket.
2. **Access Control Check**: Verifies `token` or Challenge.
3. **Dispatch**: Routes payloads to the `Agent Runtime`.

</div>

</div>

::right::

<div class="h-full flex items-center justify-center pl-4">
<img v-click 
     class="rounded-lg shadow-xl w-full object-contain max-h-[85vh]" 
     src="./docs/images/6b32c6866bc3d64865dbb404f12e9b57705d56ff.png" 
     alt="Gateway Architecture"/>
</div>

<style>
/* 核心修正 3：强制两栏顶部对齐的 CSS */
.slidev-layout.two-cols {
  display: flex !important;
  align-items: flex-start !important;
}
</style>

---
transition: slide-left
---

# Channel Adapters & Normalization

<p class="text-sm opacity-75 -mt-2 mb-3">How does OpenClaw handle WhatsApp, Discord, and Telegram simultaneously?</p>

<div class="grid grid-cols-2 gap-3 text-xs">
<div v-click class="p-3 bg-gray-50 dark:bg-gray-800 rounded">

**Underlying Libraries**

Each platform has a dedicated adapter that handles protocol quirks:
- **WhatsApp**: Uses the `Baileys` WebSocket protocol library.
- **Telegram**: Handled via `grammY`.
- **Discord**: Uses `discord.js`.

</div>
<div v-click class="p-3 bg-gray-50 dark:bg-gray-800 rounded">

**Message Normalization**

Adapters intercept platform-specific formats and normalize them into standard OpenClaw Events:
- Extracts attachments & media.
- Translates Markdown dialects.
- Handles UI typing indicators natively.

</div>
<div v-click class="p-3 bg-gray-50 dark:bg-gray-800 rounded col-span-2">

**Access Control Pipeline** — Before an inbound message hits the agent, it must pass the policy check: `channels.whatsapp.allowFrom` arrays are verified. Unknown DMs trigger the `"pairing"` policy, returning a 6-digit confirmation code instead of hitting the LLM.

</div>
</div>

---
transition: slide-up
---

# The Agent Runtime (`PiEmbeddedRunner`)

<p class="text-sm opacity-75 -mt-2 mb-4">The core engine where the intelligence loop executes, powered by <code>@mariozechner/pi-agent-core</code>.</p>

<div class="grid grid-cols-3 gap-4 text-sm mb-4">

<div class="p-3 bg-blue-50 dark:bg-blue-900/20 rounded border-l-3 border-blue-400">
<h3 class="text-blue-500 font-bold mb-1 text-sm">1. Session Resolution</h3>
Resolves incoming message to a strict identifier.<br>
e.g. <code>agent:main:whatsapp
:group:120...@g.us</code><br>
This bounds the sandboxing rule.
</div>

<div class="p-3 bg-green-50 dark:bg-green-900/20 rounded border-l-3 border-green-400">
<h3 class="text-green-500 font-bold mb-1 text-sm">2. Context Assembly</h3>
Loads <code>AGENTS.md</code>, injected SKILL subsets dynamically, and hits the <strong>Memory SQLite db</strong> for historical relevance.
</div>

<div class="p-3 bg-orange-50 dark:bg-orange-900/20 rounded border-l-3 border-orange-400">
<h3 class="text-orange-500 font-bold mb-1 text-sm">3. Stream & Tool Interception</h3>
LLM response is token-streamed. When a Tool Call is detected, it intercepts, evaluates, and streams the stdout result back into the generator loop.
</div>

</div>

<div class="mx-auto max-h-[40vh] overflow-hidden rounded shadow flex items-center justify-center">
  <img class="w-full h-full object-contain object-center" src="./docs/images/4029fffe3ce8831b7e370e1e152a107ede02e7b1.png" />
</div>

---
transition: slide-left
---

# Multi-Model Provider Support

<p class="text-sm opacity-75 -mt-2 mb-5">OpenClaw is <strong>model-agnostic</strong> — swap your LLM without touching a single line of business logic.</p>

<div class="grid grid-cols-5 gap-2 mb-5 text-center text-xs">
  <div v-click class="p-3 rounded-lg bg-emerald-50 dark:bg-emerald-900/20 border border-emerald-300 dark:border-emerald-700 flex flex-col items-center justify-center">
    <img src="./docs/logo/openai.svg" class="h-6 mb-2 object-contain" />
    <div class="font-bold">OpenAI</div>
    <div class="opacity-60 text-[10px]">GPT-5.4 / codex</div>
  </div>
  <div v-click class="p-3 rounded-lg bg-orange-50 dark:bg-orange-900/20 border border-orange-300 dark:border-orange-700 flex flex-col items-center justify-center">
    <img src="./docs/logo/claude.svg" class="h-6 mb-2 object-contain" />
    <div class="font-bold">Claude</div>
    <div class="opacity-60 text-[10px]">4.6 Sonnet / Claude code</div>
  </div>
  <div v-click class="p-3 rounded-lg bg-blue-50 dark:bg-blue-900/20 border border-blue-300 dark:border-blue-700 flex flex-col items-center justify-center">
    <img src="./docs/logo/deepseek.svg" class="h-6 mb-2 object-contain" />
    <div class="font-bold">DeepSeek</div>
    <div class="opacity-60 text-[10px]">V3 / R1</div>
  </div>
  <div v-click class="p-3 rounded-lg bg-sky-50 dark:bg-sky-900/20 border border-sky-300 dark:border-sky-700 flex flex-col items-center justify-center">
    <img src="./docs/logo/gemini-ai.svg" class="h-6 mb-2 object-contain" />
    <div class="font-bold">Gemini</div>
    <div class="opacity-60 text-[10px]">3.1 Pro / Flash</div>
  </div>
  <div v-click class="p-3 rounded-lg bg-gray-100 dark:bg-gray-800 border border-gray-300 dark:border-gray-600 flex flex-col items-center justify-center">
    <img src="./docs/logo/Ollama.svg" class="h-6 mb-2 object-contain" />
    <div class="font-bold">Ollama</div>
    <div class="opacity-60 text-[10px]">Local Models</div>
  </div>
</div>

<div class="grid grid-cols-3 gap-4 text-xs">
  <div class="p-3 bg-purple-50 dark:bg-purple-900/20 rounded-lg border-l-3 border-purple-400">
    <div class="font-bold text-purple-600 dark:text-purple-400 mb-1">🔌 Unified Adapter Layer</div>
    All providers share a single typed interface. Switching from GPT-5 to Claude requires one config line change.
  </div>
  <div class="p-3 bg-red-50 dark:bg-red-900/20 rounded-lg border-l-3 border-red-400">
    <div class="font-bold text-red-600 dark:text-red-400 mb-1">⛓️ Auto Failover Chain</div>
    If a provider is rate-limited or unavailable, OpenClaw automatically cascades to the next configured provider.
  </div>
  <div class="p-3 bg-teal-50 dark:bg-teal-900/20 rounded-lg border-l-3 border-teal-400">
    <div class="font-bold text-teal-600 dark:text-teal-400 mb-1">🔑 BYO API Keys</div>
    Credentials stored locally at <code>~/.openclaw/credentials/</code>. No vendor lock-in, no cloud dependency.
  </div>
</div>

---
transition: slide-left
---

# 1. Native Provider Mode 

<p class="text-[15px] opacity-75 -mt-2 mb-4"><strong>Native runtime: OpenClaw is the brain, and the model is just the backend.</strong></p>

<div class="grid grid-cols-2 gap-6 h-full items-start">
<div>

<div class="text-[13px] leading-relaxed text-gray-700 dark:text-gray-300 space-y-4">
  <p>OpenClaw governs the entire <strong>agent loop</strong>: context assembly, session management, hooking up native tools, and invoking the configured model provider.</p>
  <p>If you use <code>openai-codex/...</code>, Codex acts strictly as a "model supplier / auth channel," not an external agent.</p>
  <div class="p-3 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-400 mt-6 shadow-sm">
    <div class="font-bold text-blue-600 dark:text-blue-400 mb-2 text-sm">🛠️ About Tool Calls</div>
    Session management, discovery, and tool wiring are <strong>owned by OpenClaw</strong>.<br/>Native tools are entirely intercepted and executed on the OpenClaw side.
  </div>
</div>

</div>
<div>

<div class="bg-gray-50 dark:bg-gray-800/50 p-4 rounded-xl border border-gray-200 dark:border-gray-700 h-[280px] flex items-center justify-center">

```mermaid {scale: 0.8}
%%{init: { 'themeVariables': { 'fontSize': '16px', 'fontFamily': 'sans-serif' } } }%%
flowchart TD
    O[OpenClaw Agent Core]
    LLM[(LLM Provider<br/>OpenAI / Claude)]
    T((OpenClaw Tools<br/>Bash / Browser))
    
    O -- Prompt --> LLM
    LLM -- Tool Call --> O
    O -- Execute --> T
    T -. Result .-> O
    
    style O fill:#e0f2fe,stroke:#3b82f6,color:#000
    style LLM fill:#f3f4f6,stroke:#9ca3af,stroke-dasharray: 5 5,color:#000
    style T fill:#dcfce7,stroke:#22c55e,color:#000
```
</div>
</div>
</div>

---
transition: slide-left
---

# 2. CLI Backend Mode

<p class="text-[15px] opacity-75 -mt-2 mb-4"><strong>CLI Backend: OpenClaw invokes a command-line blackbox and fetches the result.</strong></p>

<div class="grid grid-cols-2 gap-6 h-full items-start">
<div>

<div class="text-[13px] leading-relaxed text-gray-700 dark:text-gray-300 space-y-4">
  <p>Treats terminal CLIs like <code>claude</code> or <code>codex</code> as a "text blackbox": assembles the prompt, spawns the CLI process, and reads the text / json / jsonl output.</p>
  <p>Officially defined as a <strong>text-only fallback runtime</strong>. While it supports session continuation, it is fundamentally a <strong>text-in → text-out</strong> flow.</p>
  <div class="p-3 bg-fuchsia-50 dark:bg-fuchsia-900/20 rounded-lg border-l-4 border-fuchsia-400 mt-6 shadow-sm">
    <div class="font-bold text-fuchsia-600 dark:text-fuchsia-400 mb-2 text-sm">🚫 About Tool Calls</div>
    <strong>Tools are disabled / no tool calls.</strong><br/>OpenClaw does not intercept or execute tools locally; it simply "feeds input in, and reads output out."
  </div>
</div>

</div>
<div>

<div class="bg-gray-50 dark:bg-gray-800/50 pt-6 pb-2 px-2 rounded-xl border border-gray-200 dark:border-gray-700 flex justify-center items-center h-[280px]">

```mermaid {scale: 0.45}
%%{init: { 'themeVariables': { 'fontSize': '28px', 'noteFontSize': '24px', 'actorFontSize': '28px' }, 'sequence': { 'actorMargin': 150, 'messageMargin': 60 } } }%%
sequenceDiagram
    participant O as OpenClaw
    participant CX as Codex / Claude CLI
    
    O->>CX: Spawn external CLI process
    Note over O, CX: Blackbox Input (Prompt + Session)
    CX-->>O: Text / JSON / JSONL Output
    Note over O, CX: Blackbox Output Parsing
```

</div>
</div>
</div>

---
transition: slide-up
---

# 3. ACP Mode

<p class="text-[15px] opacity-75 -mt-2 mb-4 leading-snug">
  <strong>ACP Agent: OpenClaw connects to an external agent,<br/>letting it run continuously.</strong>
</p>

<div class="grid grid-cols-2 gap-6 h-full items-start">
<div>

<div class="text-[13px] leading-relaxed text-gray-700 dark:text-gray-300 space-y-4">
  <p>Instead of treating external CLIs as mere "blackbox output sources", OpenClaw integrates them as fully-fledged <strong>external agent runtimes</strong>.</p>
  <p><strong>ACP sessions</strong> are designed to connect external coding harnesses. They grant full lifecycle control: create/resume sessions, bind threads to ACP, and steer/cancel at any time.</p>
  <div class="p-3 bg-emerald-50 dark:bg-emerald-900/20 rounded-lg border-l-4 border-emerald-400 mt-6 shadow-sm">
    <div class="font-bold text-emerald-600 dark:text-emerald-400 mb-2 text-sm">🤝 About Tool Calls</div>
    Core execution capabilities entirely reside with the <strong>external agent (Harness)</strong>.<br/>The external chain does the continuous work, while OpenClaw handles <strong>bridging, routing, and presentation</strong>.
  </div>
</div>

</div>
<div class="-mt-16">

<div class="bg-gray-50 dark:bg-gray-800/50 p-2 rounded-xl border border-gray-200 dark:border-gray-700 h-[420px] flex justify-center items-center">

```mermaid {scale: 0.65}
%%{init: { 'themeVariables': { 'fontSize': '16px', 'fontFamily': 'sans-serif' } } }%%
flowchart TD
    O[OpenClaw<br/>Gateway/Router]
    S((ACP Session<br/>Protocol Bridge))
    EX[External Harness<br/>Claude Code / Codex]
    T((External<br/>Tools / FS))
    
    O <-->|Steer / Route| S
    S <-->|Context / Actions| EX
    EX -->|Execute| T
    
    style O fill:#e0f2fe,stroke:#3b82f6,color:#000
    style S fill:#fef08a,stroke:#eab308,stroke-dasharray: 5 5,color:#000
    style EX fill:#dcfce7,stroke:#22c55e,color:#000
```
</div>
</div>
</div>

---
transition: slide-left
---

# 1. Memory as Source of Truth

<p class="text-[15px] opacity-75 -mt-2 mb-4"><strong>Memory is purely editable Markdown files on disk, not an invisible model state.</strong></p>

<div class="grid grid-cols-2 gap-6 h-full items-center">
<div>

<div class="text-[13px] leading-relaxed text-gray-700 dark:text-gray-300 space-y-4">
  <p><strong>Long-term Notebooks:</strong> OpenClaw writes durable facts and preferences into <code>MEMORY.md</code>, while process notes and daily context are dumped into <code>memory/YYYY-MM-DD.md</code>.</p>
  <p><strong>The Index Layer (SQLite):</strong> Plain files are hard to search. OpenClaw builds a lightweight local index using <strong>SQLite + sqlite-vec</strong> to enable hybrid search (BM25 + Vector Similarity).</p>
  <div class="p-3 bg-blue-50 dark:bg-blue-900/20 rounded-lg border-l-4 border-blue-400 mt-6 shadow-sm">
    <div class="font-bold text-blue-600 dark:text-blue-400 mb-2 text-sm">💡 Core Philosophy</div>
    The true memory resides in <strong>Markdown</strong> files. SQLite is merely a retrieval directory.<br/>You can manually read, audit, and edit the agent's memory at any time.
  </div>
</div>

</div>
<div class="h-full flex items-center -mt-12">

<div class="bg-gray-50 dark:bg-gray-800/50 p-4 w-full rounded-xl border border-gray-200 dark:border-gray-700 h-[380px] flex flex-col justify-center items-center">

```mermaid {scale: 0.75}
%%{init: { 'themeVariables': { 'fontSize': '16px', 'fontFamily': 'sans-serif' } } }%%
flowchart TD
    C[Chat Context<br/>Short-term Memory]
    M1[MEMORY.md<br/>Durable Facts / Rules]
    M2[memory/YYYY-MM-DD.md<br/>Daily Process Logs]
    SQL[(SQLite Database<br/>BM25 + Vector Index)]
    
    C -- "Write Preferences" --> M1
    C -- "Write Daily Notes" --> M2
    M1 -. "File Watcher" .-> SQL
    M2 -. "File Watcher" .-> SQL
    
    style C fill:#fef08a,stroke:#eab308,color:#000
    style M1 fill:#e0f2fe,stroke:#3b82f6,color:#000
    style M2 fill:#e0f2fe,stroke:#3b82f6,color:#000
    style SQL fill:#dcfce7,stroke:#22c55e,color:#000
```
</div>
</div>
</div>

---
transition: fade-out
---

# 2. Retrieval & Compaction

<p class="text-[14px] opacity-75 -mt-3 mb-2"><strong>Find what matters (Embeddings) & forget the chatter (Compaction).</strong></p>

<div class="flex flex-col gap-2 -mt-2">

<div class="grid grid-cols-3 gap-4 text-[12px] leading-tight text-gray-700 dark:text-gray-300">
  <div>
    <strong>Embedding Failover:</strong><br/>
    Vector search requires embeddings. OpenClaw calculates them locally if a model exists; otherwise, it falls back to remote APIs.
  </div>
  <div>
    <strong>Pre-compaction Ping:</strong><br/>
    Approaching context limits triggers a secret <em>Memory Flush</em> agentic turn, demanding the model save important facts.
  </div>
  <div class="px-2 py-1.5 bg-red-50 dark:bg-red-900/20 rounded-lg border border-red-200 mt-0">
    <div class="font-bold text-red-600 dark:text-red-400 mb-0.5">🗜️ Shrinking the Context</div>
    After flushing valid facts, older turns are compacted to summaries, preventing explosion.
  </div>
</div>

<div class="bg-gray-50 dark:bg-gray-800/50 py-1 w-full rounded-xl border border-gray-200 dark:border-gray-700 flex justify-center items-center">

```mermaid {scale: 0.48}
%%{init: { 'themeVariables': { 'fontSize': '22px', 'fontFamily': 'sans-serif' } } }%%
flowchart TD
    subgraph Retrieval["1. Semantic Retrieval"]
        direction LR
        S[memory_search] --> E{Embedding<br/>Engine}
        E -- "Local Model" --> L(Local Compute)
        E -- "Failover" --> R(Remote API)
    end
    
    subgraph Compaction["2. Context Compaction"]
        direction LR
        W[Limit Approaching] --> F[Memory Flush<br/>Save valid facts]
        F --> C[Compact History<br/>Summarize older turns]
    end
    
    Retrieval ~~~ Compaction
    
    style Retrieval fill:#f3f4f6,stroke:#9ca3af,color:#000
    style Compaction fill:#f3f4f6,stroke:#9ca3af,color:#000
    style E fill:#fef08a,stroke:#eab308,color:#000
    style F fill:#dcfce7,stroke:#22c55e,color:#000
```
</div>
</div>

---
layout: center
class: text-center
---

# Sandboxing & Security Boundaries
How OpenClaw prevents a WhatsApp user from deleting your filesystem.

<div class="mt-8 text-left inline-block w-4/5">
  <v-clicks>
    
  - 🛑 **Differential Sandboxing by Session**:
    - `agent:<id>:main` -> Executed natively on Host (Full Bash access).
    - `dm:*` and `group:*` -> Tool execution forced into **Ephemeral Docker Containers**.
  
  - 🛑 **Progressive Tool Policy Matrix**:
    `Profile -> Global -> Provider -> Agent -> Group -> Sandbox`.
    Group policy can restrict tool availability (e.g., disable `browser_control`), overriding global config.
    
  - 🛑 **Prompt Injection Defenses**:
    Sources are tagged. System instructions are strongly isolated from user payloads to prevent the model from parsing a malicious payload as a system instruction override.

  </v-clicks>
</div>

---
transition: slide-up
class: text-center
---

# Extensibility (Plugins)

<p class="text-sm opacity-75 -mt-2 mb-3">OpenClaw is extended without touching <code>src/</code>.</p>

<div class="h-[300px] border border-gray-200 dark:border-gray-700 mx-auto rounded overflow-hidden flex items-center justify-center bg-white dark:bg-gray-900 w-fit p-2">
  <img class="max-h-full max-w-full object-contain" src="./docs/images/a53c6953d6d2605e079a58912a2079588a13283d.png" />
</div>

<div v-motion
  :initial="{ y: 30, opacity: 0 }"
  :enter="{ y: 0, opacity: 1, transition: { delay: 500 } }"
  class="mt-4 text-sm leading-relaxed text-gray-700 dark:text-gray-300"
>
  The Plugin Loader scans <code>package.json</code> for <code>openclaw.extensions</code>.<br>
  Dynamically inject new model providers, custom UI widgets, or external API tools without rebooting the Gateway core.
</div>

---
transition: slide-up
---

# ClawHub — The Skills Marketplace

<p class="text-sm opacity-75 -mt-2 mb-4">OpenClaw's <strong>npm for AI agents</strong> — publish, install, and compose skills on demand.</p>

<div class="grid grid-cols-2 gap-5">

<div>
<div class="grid grid-cols-2 gap-2 text-xs">
  <div v-click class="p-3 bg-indigo-50 dark:bg-indigo-900/20 rounded-lg border border-indigo-200 dark:border-indigo-700">
    <div class="font-bold text-indigo-600 dark:text-indigo-400 mb-1">📦 100+ Skills</div>
    Bash execution, Browser automation, File Ops, Cron jobs, Canvas rendering & more.
  </div>
  <div v-click class="p-3 bg-violet-50 dark:bg-violet-900/20 rounded-lg border border-violet-200 dark:border-violet-700">
    <div class="font-bold text-violet-600 dark:text-violet-400 mb-1">🔍 Vector Search</div>
    Semantic search over the registry — find the right skill by describing what you need.
  </div>
  <div v-click class="p-3 bg-cyan-50 dark:bg-cyan-900/20 rounded-lg border border-cyan-200 dark:border-cyan-700">
    <div class="font-bold text-cyan-600 dark:text-cyan-400 mb-1">🔄 Versioned Packages</div>
    Like npm — skills are published, versioned, and community-maintained by any developer.
  </div>
  <div v-click class="p-3 bg-fuchsia-50 dark:bg-fuchsia-900/20 rounded-lg border border-fuchsia-200 dark:border-fuchsia-700">
    <div class="font-bold text-fuchsia-600 dark:text-fuchsia-400 mb-1">⚡ Hot-loaded</div>
    Skills inject at runtime — no Gateway restart required. Install and invoke instantly.
  </div>
</div>
</div>

<div v-click class="p-4 bg-red-50 dark:bg-red-950/30 border border-red-300 dark:border-red-700 rounded-xl">
  <div class="flex items-center gap-2 mb-3">
    <span class="text-2xl">☠️</span>
    <span class="font-bold text-red-600 dark:text-red-400">The ClawHavoc Incident (Early 2026)</span>
  </div>
  <div class="text-xs space-y-2 text-gray-700 dark:text-gray-300 leading-relaxed">
    <p>Attackers uploaded <strong>hundreds of malicious skills</strong> to ClawHub, mimicking legitimate tools with near-identical names.</p>
    <p>Once installed, they silently <strong>exfiltrated API keys, session tokens, and local credential files</strong> to remote C2 servers.</p>
    <p>The incident exposed the fundamental tension of open skill marketplaces: <em>openness vs. supply chain security</em>.</p>
    <div class="mt-3 p-2 bg-red-100 dark:bg-red-900/50 rounded font-mono text-[10px] border border-red-300 dark:border-red-600">
      ⚠️ Always verify publisher identity and audit permissions before installing any skill.
    </div>
  </div>
</div>

</div>

---
layout: center
class: text-center
---

# What's Next for OpenClaw

<p class="text-sm opacity-75 -mt-2 mb-8">From personal agent to enterprise-grade distributed runtime.</p>

<div class="grid grid-cols-3 gap-5 text-left text-xs w-full max-w-3xl mx-auto">

<div v-click class="p-4 rounded-xl border border-blue-200 dark:border-blue-700" style="background: linear-gradient(135deg, #eff6ff, #dbeafe); ">
  <div class="text-2xl mb-2">📬</div>
  <div class="font-bold text-blue-700 dark:text-blue-300 mb-1 text-sm">Native Task Queue</div>
  <div class="opacity-75">Persistent, prioritized task queue for long-running jobs. No more dropped tasks during LLM timeouts or connection drops.</div>
</div>

<div v-click class="p-4 rounded-xl border border-green-200 dark:border-green-700" style="background: linear-gradient(135deg, #f0fdf4, #dcfce7);">
  <div class="text-2xl mb-2">🌐</div>
  <div class="font-bold text-green-700 dark:text-green-300 mb-1 text-sm">Multi-Node Distributed</div>
  <div class="opacity-75">Horizontal scaling of Agent Runtimes across machines. One Gateway, many Executor nodes — true enterprise scale.</div>
</div>

<div v-click class="p-4 rounded-xl border border-orange-200 dark:border-orange-700" style="background: linear-gradient(135deg, #fff7ed, #ffedd5);">
  <div class="text-2xl mb-2">📋</div>
  <div class="font-bold text-orange-700 dark:text-orange-300 mb-1 text-sm">SDD Paradigm</div>
  <div class="opacity-75">Shift from <em>prompt engineering</em> to <strong>Specification-Driven Development</strong> — agents guided by structured behavioral specs.</div>
</div>

</div>

<div v-click class="mt-8 text-sm text-gray-400 dark:text-gray-500 italic">
  "OpenClaw is not a chatbot. It's the first real Operating System for LLM agents." — Peter Steinberger
</div>

---
layout: center
class: text-center
---

# Thank You

<div class="mt-10 border-t border-gray-200 dark:border-gray-700 pt-8 flex flex-col items-center justify-center gap-5">
  <div class="text-3xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-blue-500 to-teal-400">
    YuJieHuang
  </div>
  <div class="flex items-center gap-2 text-gray-600 dark:text-gray-400 bg-gray-50 dark:bg-gray-900/50 px-5 py-2 rounded-full border border-gray-200 dark:border-gray-700 shadow-sm">
    <carbon:email class="text-xl opacity-80" />
    <span class="font-mono text-sm">YuJieHuang0526@gmail.com</span>
  </div>
  
  <div class="mt-10 pt-6 border-t border-gray-100 dark:border-gray-800 flex flex-col gap-2 text-[11px] text-gray-400 font-mono opacity-80">
    <div class="mb-1 text-gray-500 font-bold uppercase tracking-wider text-[10px]">References & Further Reading</div>
    <a href="https://ppaolo.substack.com/p/openclaw-system-architecture-overview" target="_blank" class="hover:text-teal-500 transition-colors flex items-center gap-1 justify-center">
      <carbon:link /> OpenClaw System Architecture Overview
    </a>
    <a href="https://github.com/openclaw/openclaw" target="_blank" class="hover:text-teal-500 transition-colors flex items-center gap-1 justify-center">
      <carbon:logo-github /> github.com/openclaw/openclaw
    </a>
  </div>
</div>
