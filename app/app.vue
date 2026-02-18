<script setup lang="ts">
import { flashcards } from './data/flashcards'

const currentIndex = ref(0)
const showAnswer = ref(false)
const mode = ref<'sequential' | 'random'>('sequential')
const masteredCards = ref<Set<number>>(new Set())
const shuffledOrder = ref<number[]>([])

const initShuffle = () => {
  shuffledOrder.value = [...Array(flashcards.length).keys()].sort(() => Math.random() - 0.5)
}

onMounted(() => {
  initShuffle()
  const saved = localStorage.getItem('pia-mastered')
  if (saved) {
    masteredCards.value = new Set(JSON.parse(saved))
  }
})

const currentCard = computed(() => {
  const idx = mode.value === 'random' ? shuffledOrder.value[currentIndex.value] : currentIndex.value
  return flashcards[idx]
})

const progress = computed(() => {
  return Math.round((masteredCards.value.size / flashcards.length) * 100)
})

const remainingCards = computed(() => {
  return flashcards.length - masteredCards.value.size
})

const nextCard = () => {
  showAnswer.value = false
  if (currentIndex.value < flashcards.length - 1) {
    currentIndex.value++
  } else {
    currentIndex.value = 0
    if (mode.value === 'random') initShuffle()
  }
}

const prevCard = () => {
  showAnswer.value = false
  if (currentIndex.value > 0) {
    currentIndex.value--
  }
}

const toggleMastered = () => {
  const cardId = currentCard.value.id
  const newSet = new Set(masteredCards.value)
  
  if (newSet.has(cardId)) {
    newSet.delete(cardId)
  } else {
    newSet.add(cardId)
  }
  
  masteredCards.value = newSet
  localStorage.setItem('pia-mastered', JSON.stringify([...newSet]))
}

const resetProgress = () => {
  if (confirm('Remettre ta progression à zéro ?')) {
    masteredCards.value = new Set()
    localStorage.removeItem('pia-mastered')
  }
}

const isMastered = computed(() => masteredCards.value.has(currentCard.value.id))
</script>

<template>
  <div class="container">
    <!-- Grid overlay -->
    <div class="grid-overlay"></div>
    
    <!-- Corner metadata -->
    <div class="meta meta-tl">©2026</div>
    <div class="meta meta-tr">{{ String(currentIndex + 1).padStart(2, '0') }}/{{ flashcards.length }}</div>
    <div class="meta meta-bl">SYSTÈMES<br>D'INFORMATION</div>
    <div class="meta meta-br">RÉVISIONS<br>COLLÈGE</div>
    
    <!-- Floating decorations -->
    <div class="deco deco-1">+</div>
    <div class="deco deco-2">+</div>
    <div class="deco deco-3">✦</div>
    
    <!-- Main content -->
    <main class="main">
      <!-- Header -->
      <header class="header">
        <div class="title-block">
          <span class="label">FLASHCARDS</span>
          <h1 class="title">Pia's<br>Study Cards</h1>
        </div>
        
        <div class="stats-block">
          <div class="stat">
            <span class="stat-value">{{ masteredCards.size }}</span>
            <span class="stat-label">MAÎTRISÉES</span>
          </div>
          <div class="stat">
            <span class="stat-value">{{ remainingCards }}</span>
            <span class="stat-label">RESTANTES</span>
          </div>
        </div>
      </header>

      <!-- Progress bar -->
      <div class="progress-section">
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: progress + '%' }"></div>
        </div>
        <span class="progress-text">{{ progress }}% COMPLÉTÉ</span>
      </div>

      <!-- Mode selector -->
      <div class="mode-selector">
        <button 
          :class="['mode-btn', { active: mode === 'sequential' }]"
          @click="mode = 'sequential'"
        >
          SÉQUENTIEL
        </button>
        <button 
          :class="['mode-btn', { active: mode === 'random' }]"
          @click="mode = 'random'; initShuffle()"
        >
          ALÉATOIRE
        </button>
      </div>

      <!-- Card -->
      <div class="card-wrapper">
        <div 
          :class="['card', { flipped: showAnswer, mastered: isMastered }]" 
          @click="showAnswer = !showAnswer"
        >
          <div class="card-inner">
            <!-- Front -->
            <div class="card-face card-front">
              <div class="card-header">
                <span class="card-type">QUESTION</span>
                <span class="card-num">N°{{ String(currentCard.id).padStart(2, '0') }}</span>
              </div>
              <div class="card-body">
                <p>{{ currentCard.question }}</p>
              </div>
              <div class="card-footer">
                <span>TAP TO REVEAL →</span>
              </div>
            </div>
            
            <!-- Back -->
            <div class="card-face card-back">
              <div class="card-header">
                <span class="card-type card-type-answer">RÉPONSE</span>
                <span class="card-num">N°{{ String(currentCard.id).padStart(2, '0') }}</span>
              </div>
              <div class="card-body">
                <p>{{ currentCard.answer }}</p>
              </div>
              <div class="card-footer">
                <span>← TAP TO GO BACK</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Controls -->
      <div class="controls">
        <button class="ctrl prev" @click="prevCard" :disabled="currentIndex === 0">
          ← PRÉC
        </button>
        
        <button 
          :class="['ctrl master', { active: isMastered }]"
          @click="toggleMastered"
        >
          {{ isMastered ? '✓ ACQUIS' : '○ MARQUER' }}
        </button>
        
        <button class="ctrl next" @click="nextCard">
          SUIV →
        </button>
      </div>
    </main>

    <!-- Footer -->
    <footer class="footer">
      <button class="reset" @click="resetProgress">RESET</button>
      <span class="credit">MADE WITH ♥ BY MURRAY</span>
    </footer>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700&family=Space+Mono&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg: #141827;
  --bg-light: #1e2438;
  --card-bg: #0d1019;
  --text: #f5f5f5;
  --text-dim: #6b7280;
  --accent: #ff6b35;
  --accent-glow: rgba(255, 107, 53, 0.3);
  --success: #10b981;
  --border: rgba(255, 255, 255, 0.08);
}

body {
  font-family: 'Outfit', sans-serif;
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  overflow-x: hidden;
}

.container {
  min-height: 100vh;
  padding: 24px;
  position: relative;
  display: flex;
  flex-direction: column;
  max-width: 600px;
  margin: 0 auto;
}

/* Grid overlay */
.grid-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: 
    linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px);
  background-size: 60px 60px;
  pointer-events: none;
  z-index: 0;
}

/* Corner metadata */
.meta {
  position: fixed;
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  letter-spacing: 0.5px;
  color: var(--text-dim);
  z-index: 1;
}

.meta-tl { top: 20px; left: 20px; }
.meta-tr { top: 20px; right: 20px; text-align: right; }
.meta-bl { bottom: 20px; left: 20px; line-height: 1.4; }
.meta-br { bottom: 20px; right: 20px; text-align: right; line-height: 1.4; }

/* Decorations */
.deco {
  position: fixed;
  color: var(--text-dim);
  font-size: 14px;
  z-index: 1;
}

.deco-1 { top: 30%; left: 15px; }
.deco-2 { top: 60%; right: 15px; }
.deco-3 { top: 45%; right: 20px; color: var(--accent); }

/* Main */
.main {
  flex: 1;
  display: flex;
  flex-direction: column;
  position: relative;
  z-index: 2;
  padding-top: 40px;
}

/* Header */
.header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 32px;
}

.title-block {
  flex: 1;
}

.label {
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  letter-spacing: 2px;
  color: var(--accent);
  display: block;
  margin-bottom: 8px;
}

.title {
  font-size: 32px;
  font-weight: 600;
  line-height: 1.1;
  letter-spacing: -1px;
}

.stats-block {
  display: flex;
  gap: 24px;
}

.stat {
  text-align: right;
}

.stat-value {
  font-size: 28px;
  font-weight: 600;
  display: block;
  color: var(--accent);
}

.stat-label {
  font-family: 'Space Mono', monospace;
  font-size: 9px;
  letter-spacing: 1px;
  color: var(--text-dim);
}

/* Progress */
.progress-section {
  margin-bottom: 24px;
}

.progress-bar {
  height: 3px;
  background: var(--border);
  border-radius: 2px;
  overflow: hidden;
  margin-bottom: 8px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--accent), #ff8f35);
  transition: width 0.4s ease;
}

.progress-text {
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  color: var(--text-dim);
}

/* Mode selector */
.mode-selector {
  display: flex;
  gap: 12px;
  margin-bottom: 32px;
}

.mode-btn {
  flex: 1;
  padding: 12px;
  background: transparent;
  border: 1px solid var(--border);
  border-radius: 6px;
  color: var(--text-dim);
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  letter-spacing: 1px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.mode-btn:hover {
  border-color: var(--text-dim);
}

.mode-btn.active {
  border-color: var(--accent);
  color: var(--accent);
  background: rgba(255, 107, 53, 0.05);
}

/* Card */
.card-wrapper {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  perspective: 1200px;
  margin-bottom: 32px;
  min-height: 280px;
}

.card {
  width: 100%;
  height: 280px;
  cursor: pointer;
}

.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.7s cubic-bezier(0.4, 0, 0.2, 1);
}

.card.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-face {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 12px;
  padding: 24px;
  display: flex;
  flex-direction: column;
  background: var(--card-bg);
  border: 1px solid var(--border);
}

.card-front {
  border-left: 3px solid var(--text-dim);
}

.card-back {
  transform: rotateY(180deg);
  border-left: 3px solid var(--accent);
}

.card.mastered .card-front,
.card.mastered .card-back {
  border-left-color: var(--success);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.card-type {
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  letter-spacing: 2px;
  color: var(--text-dim);
}

.card-type-answer {
  color: var(--accent);
}

.card-num {
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  color: var(--text-dim);
}

.card-body {
  flex: 1;
  display: flex;
  align-items: center;
}

.card-body p {
  font-size: 17px;
  line-height: 1.6;
  font-weight: 400;
}

.card-footer {
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  color: var(--text-dim);
  text-align: center;
  opacity: 0.6;
}

/* Controls */
.controls {
  display: flex;
  gap: 12px;
}

.ctrl {
  flex: 1;
  padding: 14px 12px;
  border: 1px solid var(--border);
  border-radius: 6px;
  background: transparent;
  color: var(--text);
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  letter-spacing: 1px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.ctrl:hover:not(:disabled) {
  border-color: var(--text-dim);
}

.ctrl:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.ctrl.master {
  border-color: var(--accent);
  color: var(--accent);
}

.ctrl.master:hover {
  background: rgba(255, 107, 53, 0.1);
}

.ctrl.master.active {
  background: var(--success);
  border-color: var(--success);
  color: var(--bg);
}

/* Footer */
.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 24px;
  margin-top: auto;
  position: relative;
  z-index: 2;
}

.reset {
  background: transparent;
  border: none;
  color: var(--text-dim);
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  letter-spacing: 1px;
  cursor: pointer;
  padding: 8px 0;
}

.reset:hover {
  color: #ef4444;
}

.credit {
  font-family: 'Space Mono', monospace;
  font-size: 9px;
  letter-spacing: 1px;
  color: var(--text-dim);
}

/* Responsive */
@media (max-width: 500px) {
  .meta {
    display: none;
  }
  
  .deco {
    display: none;
  }
  
  .title {
    font-size: 26px;
  }
  
  .stat-value {
    font-size: 24px;
  }
  
  .card {
    height: 260px;
  }
  
  .card-body p {
    font-size: 15px;
  }
}
</style>
