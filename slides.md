---
theme: default
title: LLMの基礎知識
transition: fade
canvasWidth: 980
aspectRatio: 16/9
routerMode: hash
layout: cover
---

<div class="cover">
  <p class="eyebrow">01</p>
  <h1>LLMの基礎知識</h1>
</div>

<style>
.cover {
  display: grid;
  height: 100%;
  align-content: center;
  gap: 1.2rem;
}

.eyebrow {
  color: #2563eb;
  font-size: 1rem;
  font-weight: 700;
  letter-spacing: 0;
  text-transform: uppercase;
}

h1 {
  max-width: 820px;
  color: #111827;
  font-size: 4.4rem;
  line-height: 1.05;
  letter-spacing: 0;
}

.subtitle {
  max-width: 620px;
  color: #4b5563;
  font-size: 1.35rem;
  line-height: 1.5;
}
</style>

---
layout: default
---

# LLMとは

<p class="statement">
  <strong>LLMは、入力されたPromptの続きを1Tokenずつ予測して出力する仕組みである。</strong>
</p>

<div class="generator">
  <div class="panel prompt-panel">
    <span class="label">Prompt</span>
    <p class="prompt">{{ promptText }}</p>
  </div>

  <div class="cycle" aria-hidden="true">
    <div class="cycle-arrow top">→</div>
    <div class="cycle-arrow bottom">←</div>
  </div>

  <div class="panel token-panel" :class="{ complete: isComplete }">
    <span class="label">Output</span>
    <p class="token">{{ currentToken }}</p>
  </div>
</div>

<div class="tokens">
  <span class="seed">吾輩は猫である。</span>
  <span
    v-for="(token, index) in generatedTokens"
    :key="token + index"
    class="token-chip"
    :class="{ active: index === tokenIndex - 1, done: index < tokenIndex }"
  >
    {{ token }}
  </span>
</div>

<p class="caption">
  1 token を出力するたびに、それを prompt に追加して次の予測へ進む。
</p>

<script setup>
import { computed, ref } from 'vue'
import { onSlideEnter, onSlideLeave } from '@slidev/client'

const initialPrompt = '吾輩は猫である。'
const generatedTokens = ['名前', 'は', 'まだ', '無い', '。']
const tokenDelay = 850
const replayDelay = 1400
const tokenIndex = ref(0)
let timerId

const promptText = computed(() =>
  initialPrompt + generatedTokens.slice(0, tokenIndex.value).join(''),
)
const isComplete = computed(() => tokenIndex.value >= generatedTokens.length)
const currentToken = computed(() => {
  if (isComplete.value) {
    return '生成完了'
  }

  return generatedTokens[tokenIndex.value]
})

function stopGeneration() {
  if (timerId) {
    clearTimeout(timerId)
    timerId = undefined
  }
}

function queueNextStep(delay = tokenDelay) {
  timerId = setTimeout(() => {
    if (tokenIndex.value >= generatedTokens.length) {
      tokenIndex.value = 0
      queueNextStep(tokenDelay)
      return
    }

    tokenIndex.value += 1
    queueNextStep(tokenIndex.value >= generatedTokens.length ? replayDelay : tokenDelay)
  }, delay)
}

function startGeneration() {
  stopGeneration()
  tokenIndex.value = 0
  queueNextStep()
}

onSlideEnter(startGeneration)
onSlideLeave(stopGeneration)
</script>

<style>
h1 {
  margin-bottom: 1.1rem;
  color: #111827;
  font-size: 3rem;
  letter-spacing: 0;
}

.statement {
  margin-bottom: 1.6rem;
  color: #111827;
  font-size: 1.35rem;
  line-height: 1.5;
}

.generator {
  display: grid;
  grid-template-columns: 1fr 96px 0.8fr;
  align-items: center;
  gap: 1.2rem;
}

.panel {
  height: 240px;
  border: 2px solid #d1d5db;
  border-radius: 8px;
  padding: 1.5rem;
  background: #ffffff;
  box-shadow: 0 18px 36px rgb(17 24 39 / 8%);
}

.label {
  display: block;
  margin-bottom: 1.4rem;
  color: #2563eb;
  font-size: 0.9rem;
  font-weight: 700;
  text-transform: uppercase;
}

.prompt {
  height: 150px;
  color: #111827;
  font-size: 2.05rem;
  font-weight: 700;
  line-height: 1.45;
}

.token-panel {
  border-color: #2563eb;
}

.token-panel.complete {
  border-color: #0f766e;
  background: #ecfdf5;
}

.token {
  color: #111827;
  font-size: 2.4rem;
  font-weight: 800;
  line-height: 1.2;
}

.cycle {
  display: grid;
  place-items: center;
  gap: 0.7rem;
}

.cycle-arrow {
  display: grid;
  place-items: center;
  width: 64px;
  height: 42px;
  border-radius: 999px;
  background: #f3f4f6;
  color: #0f766e;
  font-size: 2rem;
  font-weight: 800;
}

.top {
  animation: move-forward 850ms ease-in-out infinite;
}

.bottom {
  animation: move-back 850ms ease-in-out infinite;
}

.tokens {
  display: flex;
  flex-wrap: wrap;
  gap: 0.8rem;
  margin-top: 2rem;
  align-items: center;
}

.seed,
.token-chip {
  border-radius: 8px;
  padding: 0.65rem 0.9rem;
  font-size: 1.15rem;
  font-weight: 700;
}

.seed {
  background: #f3f4f6;
  color: #374151;
}

.token-chip {
  border: 2px solid #d1d5db;
  background: #ffffff;
  color: #9ca3af;
  transition:
    background 220ms ease,
    border-color 220ms ease,
    color 220ms ease,
    transform 220ms ease;
}

.token-chip.done {
  border-color: #2563eb;
  background: #eff6ff;
  color: #1d4ed8;
}

.token-chip.active {
  transform: translateY(-4px);
}

.caption {
  margin-top: 1.5rem;
  color: #4b5563;
  font-size: 1.1rem;
  font-weight: 600;
}

@keyframes move-forward {
  50% {
    transform: translateX(10px);
  }
}

@keyframes move-back {
  50% {
    transform: translateX(-10px);
  }
}
</style>

---
layout: default
---

# Tokenとは

<p class="statement">
  <strong>Tokenとは、LLMにとって、自然言語の最小構成単位である。</strong>
</p>

<div class="splitter">
  <div class="block raw">
    <span class="label">原文</span>
    <p class="text">吾輩は猫である。名前はまだ無い。</p>
  </div>

  <div class="arrow" aria-hidden="true">↓</div>

  <div class="block token-block">
    <span class="label">Token分割</span>
    <div class="token-list">
      <span class="token-chip">吾輩</span>
      <span class="token-chip">は</span>
      <span class="token-chip">猫</span>
      <span class="token-chip">で</span>
      <span class="token-chip">ある</span>
      <span class="token-chip">。</span>
      <span class="token-chip">名前</span>
      <span class="token-chip">は</span>
      <span class="token-chip">まだ</span>
      <span class="token-chip">無い</span>
      <span class="token-chip">。</span>
    </div>
  </div>
</div>

<p class="caption">
  LLMは文章をこのような最小単位に分割して処理する。
</p>

<style>
h1 {
  margin-bottom: 1.1rem;
  color: #111827;
  font-size: 3rem;
  letter-spacing: 0;
}

.statement {
  margin-bottom: 1.6rem;
  color: #111827;
  font-size: 1.35rem;
  line-height: 1.5;
}

.splitter {
  display: grid;
  gap: 0.6rem;
  justify-items: center;
}

.block {
  width: 100%;
  border: 2px solid #d1d5db;
  border-radius: 8px;
  padding: 1.3rem 1.5rem;
  background: #ffffff;
  box-shadow: 0 18px 36px rgb(17 24 39 / 8%);
}

.label {
  display: block;
  margin-bottom: 1rem;
  color: #2563eb;
  font-size: 0.9rem;
  font-weight: 700;
  text-transform: uppercase;
}

.text {
  color: #111827;
  font-size: 2rem;
  font-weight: 700;
  line-height: 1.45;
}

.arrow {
  display: grid;
  place-items: center;
  width: 56px;
  height: 56px;
  border-radius: 999px;
  background: #f3f4f6;
  color: #0f766e;
  font-size: 2rem;
  font-weight: 800;
}

.token-block {
  border-color: #2563eb;
}

.token-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.7rem;
  align-items: center;
}

.token-chip {
  border-radius: 8px;
  padding: 0.65rem 0.9rem;
  font-size: 1.25rem;
  font-weight: 700;
  border: 2px solid #2563eb;
  background: #eff6ff;
  color: #1d4ed8;
}

.caption {
  margin-top: 1.5rem;
  color: #4b5563;
  font-size: 1.1rem;
  font-weight: 600;
  text-align: center;
}
</style>

---
layout: default
---

# なぜ文字単位で分割しない？

<p class="statement">
  <strong>1文字のみだとあまり意味がないときもある、<br>わざわざ分割して学習させるのはコスパが悪い。</strong>
</p>

<div class="bubbles">
  <div class="bubble">
    <span class="side left">魑魅</span>
    <span class="main">魍</span>
    <span class="side right">魎</span>
  </div>
  <div class="bubble">
    <span class="main">瘴</span>
    <span class="side right">気</span>
  </div>
  <div class="bubble">
    <span class="main">憑</span>
    <span class="side right">依</span>
  </div>
</div>

<p class="caption">
  人間が自然言語を学ぶときも同じで、これら3つの語は1つのまとまりとして覚える。<br>1文字ずつ分解して記憶する人はほとんどいない。
</p>

<style>
h1 {
  margin-bottom: 1.1rem;
  color: #111827;
  font-size: 3rem;
  letter-spacing: 0;
}

.statement {
  margin-bottom: 2rem;
  color: #111827;
  font-size: 1.35rem;
  line-height: 1.5;
}

.bubbles {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin-top: 1rem;
}

.bubble {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.15em;
  width: 260px;
  height: 150px;
  border: 2px solid #d1d5db;
  border-radius: 16px;
  background: #ffffff;
  box-shadow: 0 18px 36px rgb(17 24 39 / 8%);
}

.main {
  font-size: 3.2rem;
  font-weight: 800;
  color: #111827;
}

.side {
  font-size: 2.4rem;
  font-weight: 700;
  color: #2563eb;
  opacity: 0;
  animation: fadeInOut 6s ease-in-out infinite;
}

.side.left {
  text-align: right;
}

.side.right {
  text-align: left;
}

.caption {
  margin-top: 2rem;
  color: #4b5563;
  font-size: 1.1rem;
  font-weight: 600;
  text-align: center;
}

@keyframes fadeInOut {
  0%, 15% {
    opacity: 0;
    transform: translateY(8px);
  }
  35%, 65% {
    opacity: 1;
    transform: translateY(0);
  }
  85%, 100% {
    opacity: 0;
    transform: translateY(8px);
  }
}
</style>

---
layout: default
---

# Promptのベストプラクティス

<p class="subtitle">Promptは、LLMに送信する文のことである</p>

<div class="columns">
  <div class="column">
    <div class="bubble bad">
      <span class="tag">NG</span>
      <p class="prompt-text">私が一番好きな料理は</p>
    </div>
    <div class="arrow" aria-hidden="true">↓</div>
    <div class="bubble output">
      <span class="tag">出力</span>
      <div class="food-cycle">
        <span class="food">寿司</span>
        <span class="food">ラーメン</span>
        <span class="food">天ぷら</span>
        <span class="food">うどん</span>
        <span class="food">お好み焼き</span>
      </div>
    </div>
  </div>

  <div class="column">
    <div class="bubble good">
      <span class="tag">GOOD</span>
      <p class="prompt-text">痺れる辛さが好きなので、私が一番好きな料理は</p>
    </div>
    <div class="arrow" aria-hidden="true">↓</div>
    <div class="bubble output">
      <span class="tag">出力</span>
      <p class="food single">麻婆豆腐</p>
    </div>
  </div>
</div>

<p class="caption">
  指示が明確で文脈が十分なほど、出力が自分の意図に近づく。<br>人間でも同じで、依頼時の指示が明確で説明が十分なほど、相手が期待通りにこなしてくれる確率が高くなる。
</p>

<style>
h1 {
  margin-bottom: 0.4rem;
  color: #111827;
  font-size: 2.6rem;
  letter-spacing: 0;
}

.subtitle {
  margin-bottom: 1.6rem;
  color: #4b5563;
  font-size: 1.15rem;
  font-weight: 600;
}

.columns {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  align-items: start;
}

.column {
  display: grid;
  gap: 0.6rem;
  justify-items: center;
}

.bubble {
  position: relative;
  width: 100%;
  box-sizing: border-box;
  border: 2px solid #d1d5db;
  border-radius: 12px;
  padding: 1rem 1.2rem;
  background: #ffffff;
  box-shadow: 0 18px 36px rgb(17 24 39 / 8%);
}

.bubble.bad {
  border-color: #dc2626;
  height: 110px;
}

.bubble.good {
  border-color: #0f766e;
  height: 110px;
}

.bubble.output {
  border-color: #d1d5db;
  height: 95px;
  display: grid;
  place-items: center;
}

.tag {
  position: absolute;
  top: -0.7rem;
  left: 1rem;
  padding: 0.1rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 800;
  letter-spacing: 0.05em;
  background: #2563eb;
  color: #ffffff;
}

.bad .tag {
  background: #dc2626;
}

.good .tag {
  background: #0f766e;
}

.prompt-text {
  margin: 0.5rem 0 0;
  color: #111827;
  font-size: 1.25rem;
  font-weight: 700;
  line-height: 1.5;
}

.arrow {
  display: grid;
  place-items: center;
  width: 44px;
  height: 44px;
  border-radius: 999px;
  background: #f3f4f6;
  color: #0f766e;
  font-size: 1.6rem;
  font-weight: 800;
}

.food-cycle {
  position: relative;
  width: 100%;
  height: 2.4rem;
}

.food {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  color: #1d4ed8;
  font-size: 1.7rem;
  font-weight: 800;
  opacity: 0;
  animation: foodCycle 10s infinite;
}

.food:nth-child(1) { animation-delay: 0s; }
.food:nth-child(2) { animation-delay: 2s; }
.food:nth-child(3) { animation-delay: 4s; }
.food:nth-child(4) { animation-delay: 6s; }
.food:nth-child(5) { animation-delay: 8s; }

.food.single {
  position: static;
  opacity: 1;
  animation: none;
}

@keyframes foodCycle {
  0% { opacity: 0; transform: translateY(10px); }
  5% { opacity: 1; transform: translateY(0); }
  20% { opacity: 1; transform: translateY(0); }
  25% { opacity: 0; transform: translateY(-10px); }
  100% { opacity: 0; transform: translateY(-10px); }
}

.caption {
  margin-top: 1.8rem;
  color: #4b5563;
  font-size: 1.05rem;
  font-weight: 600;
  line-height: 1.6;
  text-align: center;
}
</style>

---
layout: default
---

# LLMはこの世界を理解していない

<p class="subtitle">2026年6月時点で「World Model」という「世界を理解してくれるAIモデル」も存在するが、それはまた別の話</p>

<div class="columns">
  <div class="column">
    <div class="bubble real">
      <span class="tag">現実世界</span>
      <div class="coin">
        <div class="face heads">表</div>
        <div class="face tails">裏</div>
      </div>
      <div class="prob">
        <span>表 <strong>50%</strong></span>
        <span class="sep">/</span>
        <span>裏 <strong>50%</strong></span>
      </div>
    </div>
  </div>
  <div class="column">
    <div class="bubble ai">
      <span class="tag">AIに「回転するコインが止まった」の続きを書かせる</span>
      <div class="coin">
        <div class="face heads">表</div>
        <div class="face tails">裏</div>
      </div>
      <div class="prob">
        <span>表 <strong class="unknown">??%</strong></span>
        <span class="sep">/</span>
        <span>裏 <strong class="unknown">??%</strong></span>
      </div>
    </div>
  </div>
</div>

<p class="caption">
  AIはこの世界の物理・論理・法則を本当に理解しているわけではない。<br>ただデータセットから得た「経験則」に完全に依存し、物理法則の結果を推測しているだけである。<br>この「理解しないまま推測する」性質が、もっともらしい嘘を自信満々に出力する「ハルシネーション（幻覚）」を生み出す。
</p>

<style>
h1 {
  margin-bottom: 0.4rem;
  color: #111827;
  font-size: 2.6rem;
  letter-spacing: 0;
}

.subtitle {
  margin-bottom: 1.4rem;
  color: #4b5563;
  font-size: 1.05rem;
  font-weight: 600;
  line-height: 1.5;
}

.columns {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  align-items: stretch;
}

.bubble {
  position: relative;
  display: grid;
  gap: 1rem;
  justify-items: center;
  align-content: center;
  padding: 1.6rem 1.2rem 1.4rem;
  border: 2px solid #d1d5db;
  border-radius: 12px;
  background: #ffffff;
  box-shadow: 0 18px 36px rgb(17 24 39 / 8%);
  perspective: 700px;
  min-height: 230px;
}

.bubble.real {
  border-color: #0f766e;
}

.bubble.ai {
  border-color: #2563eb;
}

.tag {
  position: absolute;
  top: -0.7rem;
  left: 1rem;
  padding: 0.1rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 800;
  letter-spacing: 0.05em;
  color: #ffffff;
}

.real .tag {
  background: #0f766e;
}

.ai .tag {
  background: #2563eb;
}

.coin {
  width: 88px;
  height: 88px;
  position: relative;
  transform-style: preserve-3d;
  animation: spin 1.6s linear infinite;
}

.face {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  display: grid;
  place-items: center;
  backface-visibility: hidden;
  font-size: 1.8rem;
  font-weight: 800;
  border: 3px solid #b45309;
}

.heads {
  background: linear-gradient(135deg, #fde68a, #f59e0b);
  color: #78350f;
}

.tails {
  background: linear-gradient(135deg, #fcd34d, #d97706);
  color: #78350f;
  transform: rotateY(180deg);
}

@keyframes spin {
  from { transform: rotateY(0deg); }
  to { transform: rotateY(360deg); }
}

.prob {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  color: #111827;
  font-size: 1.2rem;
  font-weight: 700;
}

.prob strong {
  color: #0f766e;
}

.ai .prob strong.unknown {
  color: #2563eb;
}

.sep {
  color: #9ca3af;
}

.caption {
  margin-top: 1.6rem;
  color: #4b5563;
  font-size: 1.05rem;
  font-weight: 600;
  line-height: 1.6;
  text-align: center;
}
</style>

---
layout: default
---

# コンテキストウィンドウ

<p class="subtitle">LLMが一度に保持できるトークン数には上限があり、これを超えると古い内容は「捨てられる」か「圧縮される」。</p>

<div class="windows">
  <div class="window-col">
    <span class="mode-tag discard">破棄モード</span>
    <div class="window">
      <div class="text-viewport">
        <div class="text-flow" :class="{ out: resetting }">{{ flowText }}</div>
      </div>
    </div>
    <p class="hint">溢れたら完全に消える</p>
  </div>

  <div class="window-col">
    <span class="mode-tag compress">圧縮モード</span>
    <div class="window">
      <div class="compressed-bar" :class="{ pulse: pulsing }">⬆ 圧縮済み: {{ compressedCount }}字</div>
      <div class="text-viewport">
        <div class="text-flow" :class="{ out: resetting }">{{ flowText }}</div>
      </div>
    </div>
    <p class="hint">要約として残る（精度は落ちる）</p>
  </div>
</div>

<p class="caption">
  コンテキストウィンドウから溢れた情報は、破棄されれば完全に失われ、圧縮されれば要約として残る。<br>LLM の「記憶」は無限ではない。
</p>

<script setup>
import { ref } from 'vue'
import { onSlideEnter, onSlideLeave } from '@slidev/client'

const sourceText = '吾輩は猫である。名前はまだ無い。どこで生れたかとんと見当がつかぬ。何でも薄暗いじめじめした所でニャーニャー泣いていた事だけは記憶している。吾輩はここで始めて人間というものを見た。しかもあとで聞くとそれは書生という人間中で一番獰悪な種族であったそうだ。この書生というのは時々我々を捕えて煮て食うという話である。しかしその当時は何という考もなかったから別段恐しいとも思わなかった。ただ彼の掌に載せられてスーと持ち上げられた時何だかフワフワした感じがあったばかりである。掌の上で少し落ちついて書生の顔を見たのがいわゆる人間というものの見始であろう。この時妙なものだと思った感じが今でも残っている。第一毛をもって装飾されべきはずの顔がつるつるしてまるで薬缶だ。その後猫にもだいぶ逢ったがこんな片輪には一度も出会わした事がない。のみならず顔の真中があまりに突-起している。そうしてその穴の中から時々ぷうぷうと煙を吹く。どうも咽せぽくて実に弱った。これが人間の飲む煙草というものである事はようやくこの頃知った。'

const STEP_CHARS = 3
const STEP_MS = 240
const COMPRESS_START = sourceText.indexOf('この書生というのは時々我々') + 'この書生というのは時々我々'.length
const flowText = ref('')
const compressedCount = ref(0)
const pulsing = ref(false)
const resetting = ref(false)
let pos = 0
let timerId

function step() {
  if (resetting.value) return
  if (pos >= sourceText.length) {
    resetting.value = true
    setTimeout(() => {
      flowText.value = ''
      compressedCount.value = 0
      pos = 0
      resetting.value = false
    }, 1400)
    return
  }
  const next = sourceText.slice(pos, pos + STEP_CHARS)
  pos += STEP_CHARS
  flowText.value += next
  const overflow = Math.max(0, flowText.value.length - COMPRESS_START)
  if (overflow > compressedCount.value) {
    compressedCount.value = overflow
    pulsing.value = true
    setTimeout(() => { pulsing.value = false }, 400)
  }
}

function start() {
  stop()
  flowText.value = ''
  compressedCount.value = 0
  pulsing.value = false
  resetting.value = false
  pos = 0
  step()
  timerId = setInterval(step, STEP_MS)
}

function stop() {
  if (timerId) {
    clearInterval(timerId)
    timerId = undefined
  }
}

onSlideEnter(start)
onSlideLeave(stop)
</script>

<style>
h1 {
  margin-bottom: 0.4rem;
  color: #111827;
  font-size: 2.6rem;
  letter-spacing: 0;
}

.subtitle {
  margin-bottom: 1.2rem;
  color: #4b5563;
  font-size: 1.05rem;
  font-weight: 600;
  line-height: 1.5;
}

.windows {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  align-items: start;
}

.window-col {
  display: grid;
  gap: 0.5rem;
  justify-items: center;
}

.mode-tag {
  padding: 0.2rem 0.9rem;
  border-radius: 999px;
  font-size: 0.85rem;
  font-weight: 800;
  color: #ffffff;
}

.mode-tag.discard {
  background: #dc2626;
}

.mode-tag.compress {
  background: #0f766e;
}

.window {
  width: 100%;
  height: 220px;
  border: 2px solid #d1d5db;
  border-radius: 12px;
  background: #ffffff;
  box-shadow: 0 18px 36px rgb(17 24 39 / 8%);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.compressed-bar {
  padding: 0.3rem 0.7rem;
  background: #ecfdf5;
  color: #0f766e;
  border-bottom: 1px solid #a7f3d0;
  font-size: 0.8rem;
  font-weight: 700;
  text-align: center;
  flex-shrink: 0;
  transition: background 0.2s, transform 0.2s;
}

.compressed-bar.pulse {
  background: #d1fae5;
  transform: scale(1.03);
}

.text-viewport {
  position: relative;
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  padding: 0.7rem 0.8rem;
}

.text-flow {
  color: #111827;
  font-size: 1.05rem;
  font-weight: 600;
  line-height: 1.7;
  word-break: break-all;
  transition: opacity 0.5s ease;
}

.text-flow.out {
  opacity: 0;
}

.hint {
  color: #6b7280;
  font-size: 0.85rem;
  font-weight: 600;
}

.caption {
  margin-top: 1.4rem;
  color: #4b5563;
  font-size: 1.05rem;
  font-weight: 600;
  line-height: 1.6;
  text-align: center;
}
</style>

---
layout: default
---

# まとめ

<p class="subtitle">ここまで学んだ LLM の基礎</p>

<div class="summary-grid">
  <div class="card">
    <span class="num">1</span>
    <div>
      <p class="card-title">LLMとは</p>
      <p class="card-desc">入力の続きを 1 トークンずつ予測して出力する仕組み</p>
    </div>
  </div>
  <div class="card">
    <span class="num">2</span>
    <div>
      <p class="card-title">トークン分割</p>
      <p class="card-desc">自然言語の最小構成単位。1文字単位よりも適切な粒度がある</p>
    </div>
  </div>
  <div class="card">
    <span class="num">3</span>
    <div>
      <p class="card-title">Prompt 設計</p>
      <p class="card-desc">指示が明確で、文脈が十分であるほど、出力は意図に近づく</p>
    </div>
  </div>
  <div class="card">
    <span class="num">4</span>
    <div>
      <p class="card-title">LLMは世界を理解しない</p>
      <p class="card-desc">物理・論理・法則を本当に理解しているわけではない</p>
    </div>
  </div>
  <div class="card">
    <span class="num">5</span>
    <div>
      <p class="card-title">ハルシネーション</p>
      <p class="card-desc">理解せずに推測する性質が、もっともらしい嘘を生む</p>
    </div>
  </div>
  <div class="card">
    <span class="num">6</span>
    <div>
      <p class="card-title">コンテキストウィンドウ</p>
      <p class="card-desc">記憶の上限を超えると、古い情報は破棄または圧縮される</p>
    </div>
  </div>
</div>

<p class="caption">
  LLMは強力な道具だが、その仕組みと限界を理解した上で扱うことが重要である。
</p>

<style>
h1 {
  margin-bottom: 0.4rem;
  color: #111827;
  font-size: 2.8rem;
  letter-spacing: 0;
}

.subtitle {
  margin-bottom: 1.6rem;
  color: #4b5563;
  font-size: 1.1rem;
  font-weight: 600;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  margin-top: 0.5rem;
}

.card {
  display: flex;
  align-items: flex-start;
  gap: 0.65rem;
  padding: 1rem 1.1rem;
  border: 2px solid #e5e7eb;
  border-radius: 10px;
  background: #ffffff;
  box-shadow: 0 8px 24px rgb(17 24 39 / 6%);
}

.num {
  flex-shrink: 0;
  display: grid;
  place-items: center;
  width: 28px;
  height: 28px;
  border-radius: 999px;
  background: #2563eb;
  color: #ffffff;
  font-size: 0.8rem;
  font-weight: 800;
}

.card-title {
  margin: 0 0 0.2rem;
  color: #111827;
  font-size: 1rem;
  font-weight: 700;
}

.card-desc {
  margin: 0;
  color: #4b5563;
  font-size: 0.85rem;
  font-weight: 500;
  line-height: 1.5;
}

.caption {
  margin-top: 1.6rem;
  color: #4b5563;
  font-size: 1.05rem;
  font-weight: 600;
  line-height: 1.6;
  text-align: center;
}
</style>
