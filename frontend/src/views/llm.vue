<template>
  <div class="llm-assistant" aria-live="polite">
    <header class="oa-header">
      <div class="brand">
        <div class="logo" aria-hidden>
          <svg width="36" height="36" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <rect x="2" y="2" width="20" height="20" rx="6" fill="url(#g)" />
            <defs>
              <linearGradient id="g" x1="0" x2="1">
                <stop offset="0" stop-color="#7C3AED" />
                <stop offset="1" stop-color="#06B6D4" />
              </linearGradient>
            </defs>
          </svg>
        </div>
        <div>
          <h1>Optical AI Assistant</h1>
          <p class="tag">LLM-powered help for optical design — dark mode</p>
        </div>
      </div>
      <nav class="meta">
        <span class="chip">Model: <strong>Opti-GPT</strong></span>
        <button class="cta">Start New Session</button>
      </nav>
    </header>

    <main class="oa-main">
      <aside class="oa-side">
        <section class="info">
          <h3>Quick Tips</h3>
          <ul>
            <li>Describe your optical setup (lenses, distances, wavelength).</li>
            <li>Attach diagrams or schematics for better help.</li>
            <li>Ask for equations, ray-tracing steps, or optimization ideas.</li>
          </ul>
        </section>

        <section class="prompts">
          <h3>Example Prompts</h3>
          <button class="prompt">Explain aberrations in a doublet</button>
          <button class="prompt">Suggest coatings for 450–650 nm</button>
          <button class="prompt">Simplify lens prescription</button>
        </section>
      </aside>

      <section class="chat-wrap" role="region" aria-label="Conversation with Optical AI">
        <div class="chat-window" ref="chatWindow">
          <div class="message bot">
            <div class="m-avatar">O</div>
            <div class="m-body">
              <div class="m-meta">Opti-GPT · <span>Knowledge: optics</span></div>
              <p>Hi — tell me about your optical design or paste your lens prescription to get started.</p>
            </div>
          </div>

          <div class="message user">
            <div class="m-body">
              <p>I have a singlet lens with focal length 50 mm. How can I reduce spherical aberration?</p>
            </div>
            <div class="m-avatar user-av">Y</div>
          </div>

          <!-- example assistant reply (placeholder) -->
          <div class="message bot">
            <div class="m-avatar">O</div>
            <div class="m-body">
              <p>Consider aspherical surfaces or a multi-element design. I can show equations or ray sketches — which would you like?</p>
            </div>
          </div>
        </div>

        <form class="composer" @submit.prevent="sendMessage">
          <label for="prompt" class="visually-hidden">Message</label>
          <textarea id="prompt" v-model="prompt" placeholder="Ask something e.g. 'optimize for NA 0.2, 532 nm'" rows="1" @input="autoGrow($event)"></textarea>

          <div class="controls">
            <button type="button" class="icon-btn" @click="attach()" title="Attach file">📎</button>
            <button type="button" class="icon-btn" @click="toggleAdvanced" title="Advanced options">⚙️</button>
            <button type="submit" class="send">Send ↵</button>
          </div>
        </form>

        <div class="footer-note">Responses are AI-assisted. Be sure to verify calculations for production use.</div>
      </section>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const prompt = ref('')
const chatWindow = ref(null)

function sendMessage() {
  if (!prompt.value.trim()) return
  // placeholder: in real app you'd push to messages and call the API
  alert('Pretend sending: ' + prompt.value)
  prompt.value = ''
}

function autoGrow(e) {
  const ta = e.target
  ta.style.height = 'auto'
  ta.style.height = ta.scrollHeight + 'px'
}

function attach() {
  alert('Attach disabled in demo')
}

function toggleAdvanced() {
  alert('Advanced options will appear here')
}

onMounted(() => {
  // subtle auto-scroll behaviour
  if (chatWindow.value) chatWindow.value.scrollTop = chatWindow.value.scrollHeight
})
</script>

<style scoped>
:root{
  --bg: #0b0f14;
  --card: rgba(255,255,255,0.03);
  --muted: rgba(255,255,255,0.55);
  --accent1: #7C3AED;
  --accent2: #06B6D4;
  --glass: linear-gradient(135deg, rgba(124,58,237,0.12), rgba(6,182,212,0.06));
}

.llm-assistant{
  min-height: 100vh;
  background: radial-gradient(1200px 600px at 10% 10%, rgba(124,58,237,0.06), transparent 8%), radial-gradient(800px 400px at 90% 90%, rgba(6,182,212,0.04), transparent 8%), var(--bg);
  color: #e6eef6;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
  padding: 28px;
  box-sizing: border-box;
}

.oa-header{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:18px;
}

.brand{display:flex;gap:14px;align-items:center}
.brand h1{font-size:20px;margin:0}
.brand .tag{color:var(--muted);font-size:12px;margin-top:2px}
.logo{width:48px;height:48px;border-radius:10px;display:flex;align-items:center;justify-content:center;box-shadow:0 6px 18px rgba(2,6,23,0.6), inset 0 1px 0 rgba(255,255,255,0.02);background:transparent}

.meta{display:flex;gap:12px;align-items:center}
.chip{background:rgba(255,255,255,0.03);padding:6px 10px;border-radius:999px;font-size:13px;color:var(--muted)}
.cta{background:linear-gradient(90deg,var(--accent1),var(--accent2));padding:8px 12px;border-radius:10px;border:none;color:white;font-weight:600;cursor:pointer}

.oa-main{display:grid;grid-template-columns:300px 1fr;gap:20px}
.oa-side{background:var(--card);padding:16px;border-radius:12px;backdrop-filter: blur(6px);box-shadow: 0 8px 30px rgba(2,6,23,0.6)}
.info h3, .prompts h3{margin:0 0 8px 0}
.info ul{margin:0;padding-left:16px;color:var(--muted);font-size:14px}
.prompts{margin-top:18px}
.prompt{display:block;width:100%;text-align:left;margin-bottom:8px;padding:8px;border-radius:8px;border:none;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));color:#dfeeff;cursor:pointer}

.chat-wrap{display:flex;flex-direction:column;min-height:60vh;background:
  linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:12px;border-radius:12px;box-shadow:0 10px 40px rgba(2,6,23,0.6)}

.chat-window{flex:1;overflow:auto;padding:14px;display:flex;flex-direction:column;gap:12px}
.message{display:flex;gap:12px;align-items:flex-start;max-width:78%}
.message.bot{align-self:flex-start}
.message.user{align-self:flex-end;flex-direction:row-reverse}
.m-avatar{width:44px;height:44px;border-radius:10px;display:flex;align-items:center;justify-content:center;background:linear-gradient(180deg,var(--accent1),var(--accent2));font-weight:700;box-shadow:0 6px 16px rgba(6,12,30,0.6)}
.user-av{background:linear-gradient(180deg,#111827,#0f172a);border:1px solid rgba(255,255,255,0.03)}
.m-body{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:12px;border-radius:10px;border:1px solid rgba(255,255,255,0.02)}
.m-meta{font-size:12px;color:var(--muted);margin-bottom:6px}

.composer{display:flex;gap:10px;padding:10px;border-top:1px dashed rgba(255,255,255,0.02);align-items:end}
.composer textarea{flex:1;min-height:42px;max-height:160px;padding:10px;border-radius:10px;background:transparent;border:1px solid rgba(255,255,255,0.04);color:inherit;font-size:14px;resize:none;outline:none}
.controls{display:flex;gap:8px;align-items:center}
.icon-btn{background:transparent;border:1px solid rgba(255,255,255,0.03);padding:8px;border-radius:8px;cursor:pointer}
.send{background:linear-gradient(90deg,var(--accent2),var(--accent1));padding:10px 14px;border-radius:10px;border:none;color:#021428;font-weight:700;cursor:pointer}

.footer-note{font-size:12px;color:var(--muted);padding:10px 14px}
.visually-hidden{position:absolute;left:-9999px}

/* responsive */
@media (max-width:900px){
  .oa-main{grid-template-columns:1fr;}
  .oa-side{order:2}
  .chat-wrap{order:1}
}
</style>
