<template>
  <div class="window glass fade-in">
    <div class="titlebar">
      <div class="mac-buttons">
        <span class="btn red"></span>
        <span class="btn yellow"></span>
        <span class="btn green"></span>
      </div>
      <div class="title">akaruineko@akaruineko-void &#8211; zsh</div>
    </div>

    <div class="terminal" ref="terminal">
      <pre ref="output"></pre>
      <span v-if="!showNext" class="cursor">█</span>
      <div v-if="showNext"><pre ref="nextOutput"></pre></div>
    </div>
  </div>

  <div v-if="!animationsReady" class="waiting-interaction">
    <p>click anywhere or press any key to start...</p>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'

const output = ref(null)
const nextOutput = ref(null)
const terminal = ref(null)
const showNext = ref(false)
const typedOnce = ref(false)
const animationsReady = ref(false)
const delay = 40

let audioCtx = null
function initAudioCtx(){if(!audioCtx)audioCtx=new(window.AudioContext||window.webkitAudioContext)()}

function startupBeep(){
  initAudioCtx()
  const o=audioCtx.createOscillator()
  const g=audioCtx.createGain()
  o.type="square"; o.frequency.value=420
  g.gain.value=0.25
  o.connect(g); g.connect(audioCtx.destination)
  o.start(); o.stop(audioCtx.currentTime+0.25)
}

function playTypeSound(){
  initAudioCtx()
  const dur=0.03
  const buffer=audioCtx.createBuffer(1,audioCtx.sampleRate*dur,audioCtx.sampleRate)
  const data=buffer.getChannelData(0)
  for(let i=0;i<data.length;i++)data[i]=(Math.random()*0.4-0.2)*Math.pow(1-i/data.length,1.5)
  const src=audioCtx.createBufferSource()
  src.buffer=buffer
  src.connect(audioCtx.destination)
  src.start()
}

const intro=[
  "> initializing system...",
  "> connecting to akaruineko@akaruineko-void",
  "> loading environment variables...",
  "> ready.\n"
]

const lines=[
  "привет, я akaruineko 🐱",
  "я школьник и программист-энтузиаст.",
  "",
  "🗣 языки общения:",
  "- русский 🇷🇺",
  "- английский 🇬🇧",
  "- украинский 🇺🇦",
  "",
  "💻 языки программирования:",
  "- Rust",
  "- Python",
  "- C",
  "",
  "🔗 контакты:",
  "- github: https://github.com/BrightCat14",
  "- email: brightcat1950@gmail.com"
]

const nextLines=[
  "",
  "> loading projects...",
  "",
  "- bf2asm",
  "- righton",
  "- AN/do",
  "",
  "спасибо, что заглянул ❤️"
]

function isMac(){return /Mac/.test(navigator.platform)}

async function typeText(el,textArray){
  for(const line of textArray){
    for(const ch of line){
      el.textContent+=ch
      if(ch!==' '&&ch!=='\n') playTypeSound()
      await new Promise(r=>setTimeout(r,delay+Math.random()*20))
    }
    el.textContent+='\n'
    terminal.value.scrollTop = terminal.value.scrollHeight
  }
}

function startAnimations(){
  animationsReady.value=true
  setTimeout(async()=>{
    await typeText(output.value,intro)
    await new Promise(r=>setTimeout(r,400))
    await typeText(output.value,lines)

    const hint=document.createElement("div")
    hint.className="press-enter"
    hint.textContent=isMac() ? "press [Return] to continue..." : "press [Enter] to continue..."
    hint.addEventListener("click",async()=>{if(!typedOnce.value){typedOnce.value=true; triggerNext(hint)}})
    terminal.value.appendChild(hint)
    terminal.value.scrollTop = terminal.value.scrollHeight

    window.addEventListener("keydown",async(e)=>{
      if(!typedOnce.value&&(e.key==="Enter"||e.key==="Return")){
        typedOnce.value=true
        triggerNext(hint)
      }
    })
  },100)
}

async function triggerNext(hintEl){
  showNext.value=true
  await nextTick()
  hintEl.remove()
  await typeText(nextOutput.value,nextLines)
  terminal.value.scrollTop = terminal.value.scrollHeight
}

onMounted(()=>{
  const handler=()=>{
    startupBeep()
    startAnimations()
    window.removeEventListener('click',handler)
    window.removeEventListener('keydown',handler)
  }
  window.addEventListener('click',handler)
  window.addEventListener('keydown',handler)
})
</script>
