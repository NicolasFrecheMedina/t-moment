<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const scene = ref<HTMLElement | null>(null)
const pointerX = ref(0.5)
const pointerY = ref(0.5)
const isRewinding = ref(false)
const mode = ref<'time' | 'memory'>('time')
let rewindTimer: number | undefined

const sceneStyle = computed(() => ({
  '--pointer-x': pointerX.value.toFixed(3),
  '--pointer-y': pointerY.value.toFixed(3),
  '--shift-x': `${((pointerX.value - 0.5) * 2).toFixed(2)}`,
  '--shift-y': `${((pointerY.value - 0.5) * 2).toFixed(2)}`,
}))

function trackPointer(event: PointerEvent) {
  if (!scene.value || event.pointerType === 'touch') return
  const bounds = scene.value.getBoundingClientRect()
  pointerX.value = Math.min(1, Math.max(0, (event.clientX - bounds.left) / bounds.width))
  pointerY.value = Math.min(1, Math.max(0, (event.clientY - bounds.top) / bounds.height))
}

function resetPointer() {
  pointerX.value = 0.5
  pointerY.value = 0.5
}

function rewind() {
  window.clearTimeout(rewindTimer)
  isRewinding.value = false
  requestAnimationFrame(() => {
    isRewinding.value = true
    rewindTimer = window.setTimeout(() => { isRewinding.value = false }, 1100)
  })
}

function handleKeydown(event: KeyboardEvent) {
  if (event.key === 'Escape') resetPointer()
}

onMounted(() => window.addEventListener('keydown', handleKeydown))
onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeydown)
  window.clearTimeout(rewindTimer)
})
</script>

<template>
  <main ref="scene" class="scene" :class="{ 'is-rewinding': isRewinding, 'is-memory': mode === 'memory' }" :style="sceneStyle" @pointermove="trackPointer" @pointerleave="resetPointer">
    <div class="ambient" aria-hidden="true">
      <div class="ambient__halo"></div><div class="ambient__grid"></div><div class="ambient__grain"></div>
    </div>

    <header class="topbar reveal reveal--meta">
      <div class="mode-switch" role="group" aria-label="Perspective T-Moment">
        <button type="button" :class="{ active: mode === 'time' }" :aria-pressed="mode === 'time'" aria-label="Mode Temps" @click="mode = 'time'"><span>T</span><em>TEMPS</em></button>
        <i aria-hidden="true"></i>
        <button type="button" :class="{ active: mode === 'memory' }" :aria-pressed="mode === 'memory'" aria-label="Mode Mémoire" @click="mode = 'memory'"><span>M</span><em>MÉMOIRE</em></button>
      </div>
    </header>

    <section id="moment" class="hero" aria-labelledby="title">
      <div class="title-block parallax parallax--text">
        <div class="eyebrow mode-content reveal reveal--labels" aria-live="polite"><span :class="{ active: mode === 'time' }">FLUX TEMPOREL ACTIF</span><span :class="{ active: mode === 'memory' }">MÉMOIRE TEMPORAIRE</span></div>
        <h1 id="title" class="reveal reveal--title" aria-label="T-Moment"><span>T</span><span class="title-dash">-</span><span>MOMENT</span></h1>
      </div>

      <div class="buffer-wrap parallax parallax--buffer">
        <div class="buffer-labels reveal reveal--labels" aria-hidden="true">
          <div class="mode-set" :class="{ active: mode === 'time' }"><span>PASSÉ ACCESSIBLE</span><span>MAINTENANT</span></div>
          <div class="mode-set" :class="{ active: mode === 'memory' }"><span>MÉMOIRE TEMPORAIRE</span><span>TRACE CONSERVÉE</span></div>
        </div>
        <div class="buffer reveal reveal--buffer" aria-label="Représentation du temps récent encore accessible">
          <div class="buffer__memory" aria-hidden="true"></div><div class="buffer__cursor" aria-hidden="true"></div>
          <div class="buffer__rail" aria-hidden="true"><span v-for="index in 34" :key="index" :style="{ '--tick-height': `${5 + (index % 3) * 2}px` }"></span></div>

          <svg class="signal" viewBox="0 0 1200 160" preserveAspectRatio="none" aria-hidden="true">
            <defs>
              <linearGradient id="trace" x1="0" x2="1"><stop offset="0" stop-color="#78dfe9" stop-opacity="0"/><stop offset=".22" stop-color="#78dfe9" stop-opacity=".35"/><stop offset=".7" stop-color="#d9fbff" stop-opacity=".7"/><stop offset="1" stop-color="#efffff" stop-opacity=".08"/></linearGradient>
              <filter id="soft-glow" x="-20%" y="-100%" width="140%" height="300%"><feGaussianBlur stdDeviation="3" result="blur"/><feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
            </defs>
            <path class="signal__ghost" d="M0 80 C80 80 104 78 158 80 S242 80 300 80 C335 80 344 49 370 80 S405 108 422 80 S454 75 485 80 S548 80 585 80 C615 80 624 62 642 80 S670 96 686 80 S715 79 760 80 S850 80 916 80 C944 80 951 54 970 80 S998 103 1016 80 S1045 80 1200 80"/>
            <path class="signal__main" d="M0 80 C80 80 104 78 158 80 S242 80 300 80 C335 80 344 49 370 80 S405 108 422 80 S454 75 485 80 S548 80 585 80 C615 80 624 62 642 80 S670 96 686 80 S715 79 760 80 S850 80 916 80 C944 80 951 54 970 80 S998 103 1016 80 S1045 80 1200 80"/>
            <path class="signal__memory signal__memory--one" d="M40 80 C155 80 185 80 252 80 C300 80 306 42 349 42 C392 42 402 118 448 118 C494 118 505 80 560 80 C650 80 700 80 770 80"/>
            <path class="signal__memory signal__memory--two" d="M205 80 C275 80 294 57 326 57 C360 57 375 103 410 103 C445 103 463 80 525 80 C600 80 647 80 720 80"/>
            <path class="signal__rewind" d="M770 80 C720 80 705 49 680 80 S630 108 608 80 S565 52 540 80 S490 103 468 80 S420 80 335 80"/>
          </svg>

          <div class="particles" aria-hidden="true"><i v-for="index in 18" :key="index" :style="{ '--particle': index, '--particle-y': `${(index % 5 - 2) * 6}px`, '--particle-opacity': .12 + (index % 4) * .08, '--particle-duration': `${5 + (index % 4)}s` }"></i></div>
          <button class="undo reveal reveal--undo" type="button" aria-label="Revenir quelques instants en arrière" @click="rewind" @pointerenter="rewind">
            <span class="undo__orbit" aria-hidden="true"></span><span class="undo__icon" aria-hidden="true">↶</span><span class="undo__label">RETOUR</span>
          </button>
          <div class="now" aria-hidden="true"><span class="now__pulse"></span><i></i></div>
        </div>
        <div class="buffer-data reveal reveal--labels" aria-hidden="true">
          <div class="mode-set" :class="{ active: mode === 'time' }"><span>00:08.42</span><span>TRACE ACTIVE</span><span>BUFFER / 12 S</span></div>
          <div class="mode-set" :class="{ active: mode === 'memory' }"><span>00:08.42</span><span>BUFFER ACTIF</span><span>TRACE CONSERVÉE</span></div>
        </div>
      </div>

      <div class="copy-block parallax parallax--text">
        <div class="mode-content claim reveal reveal--claim" aria-live="polite">
          <p :class="{ active: mode === 'time' }">Revenir dans le temps,<br/>juste quand on en a besoin.</p>
          <p :class="{ active: mode === 'memory' }">Ce qui vient de disparaître<br/>peut encore être retrouvé.</p>
        </div>
        <div class="mode-content description reveal reveal--description">
          <p :class="{ active: mode === 'time' }">Une nouvelle façon d’interagir avec<br class="desktop-break"/> ce qui vient juste de se produire.</p>
          <p :class="{ active: mode === 'memory' }">Une mémoire courte, disponible juste assez longtemps<br class="desktop-break"/> pour revenir sur l’instant.</p>
        </div>
      </div>
    </section>

    <footer class="footer reveal reveal--labels">
      <p><span>PRODUIT EN DÉVELOPPEMENT</span></p>
      <a href="https://nacodev.com" target="_blank" rel="noopener noreferrer">NACODEV <span aria-hidden="true">↗</span></a>
    </footer>
  </main>
</template>
