<script setup lang="ts">
import { flashcards } from './data/flashcards'

const currentIndex = ref(0)
const showAnswer = ref(false)
const mode = ref<'sequential' | 'random'>('sequential')
const masteredCards = ref<Set<number>>(new Set())
const shuffledOrder = ref<number[]>([])
const showCelebration = ref(false)

// Initialize shuffled order
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

const level = computed(() => {
  const p = progress.value
  if (p >= 100) return { name: 'LÉGENDE', icon: '👑' }
  if (p >= 80) return { name: 'EXPERT', icon: '⚡' }
  if (p >= 60) return { name: 'PRO', icon: '🔥' }
  if (p >= 40) return { name: 'INTERMÉDIAIRE', icon: '💪' }
  if (p >= 20) return { name: 'APPRENTI', icon: '📚' }
  return { name: 'DÉBUTANT', icon: '🌱' }
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
    // Celebration on milestone
    if (newSet.size % 10 === 0 || newSet.size === flashcards.length) {
      triggerCelebration()
    }
  }
  
  masteredCards.value = newSet
  localStorage.setItem('pia-mastered', JSON.stringify([...newSet]))
}

const triggerCelebration = () => {
  showCelebration.value = true
  setTimeout(() => {
    showCelebration.value = false
  }, 2000)
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
  <div class="game-container">
    <!-- Celebration overlay -->
    <div v-if="showCelebration" class="celebration">
      <div class="celebration-text">LEVEL UP! {{ level.icon }}</div>
    </div>

    <!-- Header -->
    <header class="header">
      <div class="logo">
        <span class="logo-icon">🎮</span>
        <span class="logo-text">FLASHCARDS</span>
      </div>
      <div class="level-badge">
        <span class="level-icon">{{ level.icon }}</span>
        <span class="level-name">{{ level.name }}</span>
      </div>
    </header>

    <!-- XP Bar -->
    <div class="xp-section">
      <div class="xp-label">
        <span>PROGRESSION</span>
        <span class="xp-percent">{{ progress }}%</span>
      </div>
      <div class="xp-bar">
        <div class="xp-fill" :style="{ width: progress + '%' }"></div>
        <div class="xp-glow" :style="{ width: progress + '%' }"></div>
      </div>
      <div class="xp-stats">
        <span>{{ masteredCards.size }}/{{ flashcards.length }} maîtrisées</span>
        <span class="remaining">{{ remainingCards }} restantes</span>
      </div>
    </div>

    <!-- Mode Toggle -->
    <div class="mode-toggle">
      <button 
        :class="['mode-btn', { active: mode === 'sequential' }]"
        @click="mode = 'sequential'"
      >
        📖 Ordre
      </button>
      <button 
        :class="['mode-btn', { active: mode === 'random' }]"
        @click="mode = 'random'; initShuffle()"
      >
        🎲 Aléatoire
      </button>
    </div>

    <!-- Card -->
    <div class="card-container">
      <div :class="['card', { flipped: showAnswer, mastered: isMastered }]" @click="showAnswer = !showAnswer">
        <div class="card-inner">
          <!-- Question side -->
          <div class="card-front">
            <div class="card-badge">Q.{{ currentCard.id }}</div>
            <div class="card-content">
              <p class="question-text">{{ currentCard.question }}</p>
            </div>
            <div class="card-hint">TAP POUR VOIR LA RÉPONSE</div>
          </div>
          
          <!-- Answer side -->
          <div class="card-back">
            <div class="card-badge answer-badge">RÉPONSE</div>
            <div class="card-content">
              <p class="answer-text">{{ currentCard.answer }}</p>
            </div>
            <div class="card-hint">TAP POUR REVOIR LA QUESTION</div>
          </div>
        </div>
      </div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <button class="ctrl-btn prev" @click="prevCard" :disabled="currentIndex === 0">
        ◀ PRÉC
      </button>
      
      <button 
        :class="['ctrl-btn master', { active: isMastered }]"
        @click="toggleMastered"
      >
        {{ isMastered ? '✓ MAÎTRISÉE' : '☆ MAÎTRISER' }}
      </button>
      
      <button class="ctrl-btn next" @click="nextCard">
        SUIV ▶
      </button>
    </div>

    <!-- Card counter -->
    <div class="counter">
      {{ currentIndex + 1 }} / {{ flashcards.length }}
    </div>

    <!-- Footer -->
    <footer class="footer">
      <button class="reset-btn" @click="resetProgress">
        🔄 Reset
      </button>
      <span class="footer-text">Bon courage Pia ! 💪</span>
    </footer>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&family=DM+Sans:wght@400;500;700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg-dark: #0d0d0d;
  --bg-card: #1a1a2e;
  --cyan: #00fff5;
  --magenta: #ff00ff;
  --yellow: #ffff00;
  --green: #00ff88;
  --text: #ffffff;
  --text-dim: #888;
  --glow-cyan: 0 0 20px rgba(0, 255, 245, 0.5);
  --glow-magenta: 0 0 20px rgba(255, 0, 255, 0.5);
  --glow-green: 0 0 20px rgba(0, 255, 136, 0.5);
}

body {
  font-family: 'DM Sans', sans-serif;
  background: var(--bg-dark);
  color: var(--text);
  min-height: 100vh;
  overflow-x: hidden;
}

.game-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
  background: 
    radial-gradient(ellipse at top, rgba(0, 255, 245, 0.05) 0%, transparent 50%),
    radial-gradient(ellipse at bottom, rgba(255, 0, 255, 0.05) 0%, transparent 50%),
    var(--bg-dark);
}

/* Celebration */
.celebration {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  animation: celebrationFade 2s ease-out forwards;
}

.celebration-text {
  font-family: 'Press Start 2P', monospace;
  font-size: 24px;
  color: var(--yellow);
  text-shadow: var(--glow-cyan), 0 0 40px var(--cyan);
  animation: celebrationPulse 0.5s ease-in-out infinite alternate;
}

@keyframes celebrationFade {
  0%, 70% { opacity: 1; }
  100% { opacity: 0; pointer-events: none; }
}

@keyframes celebrationPulse {
  from { transform: scale(1); }
  to { transform: scale(1.1); }
}

/* Header */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 10px;
}

.logo-icon {
  font-size: 24px;
}

.logo-text {
  font-family: 'Press Start 2P', monospace;
  font-size: 14px;
  color: var(--cyan);
  text-shadow: var(--glow-cyan);
}

.level-badge {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: linear-gradient(135deg, rgba(255, 0, 255, 0.2), rgba(0, 255, 245, 0.2));
  border: 1px solid var(--magenta);
  border-radius: 20px;
}

.level-icon {
  font-size: 16px;
}

.level-name {
  font-family: 'Press Start 2P', monospace;
  font-size: 8px;
  color: var(--magenta);
}

/* XP Section */
.xp-section {
  margin-bottom: 20px;
}

.xp-label {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-size: 12px;
  color: var(--text-dim);
}

.xp-percent {
  font-family: 'Press Start 2P', monospace;
  font-size: 10px;
  color: var(--cyan);
}

.xp-bar {
  height: 12px;
  background: var(--bg-card);
  border-radius: 6px;
  overflow: hidden;
  position: relative;
  border: 1px solid rgba(0, 255, 245, 0.3);
}

.xp-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--cyan), var(--magenta));
  transition: width 0.5s ease-out;
  position: relative;
}

.xp-glow {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: linear-gradient(90deg, var(--cyan), var(--magenta));
  filter: blur(8px);
  opacity: 0.5;
  transition: width 0.5s ease-out;
}

.xp-stats {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
  font-size: 11px;
  color: var(--text-dim);
}

.remaining {
  color: var(--magenta);
}

/* Mode Toggle */
.mode-toggle {
  display: flex;
  gap: 10px;
  margin-bottom: 24px;
}

.mode-btn {
  flex: 1;
  padding: 12px;
  background: transparent;
  border: 1px solid var(--text-dim);
  border-radius: 8px;
  color: var(--text-dim);
  font-family: 'DM Sans', sans-serif;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.mode-btn.active {
  border-color: var(--cyan);
  color: var(--cyan);
  background: rgba(0, 255, 245, 0.1);
  box-shadow: var(--glow-cyan);
}

/* Card */
.card-container {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  perspective: 1000px;
  margin-bottom: 24px;
}

.card {
  width: 100%;
  max-width: 400px;
  height: 280px;
  cursor: pointer;
  position: relative;
}

.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.card.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 16px;
  padding: 24px;
  display: flex;
  flex-direction: column;
  background: var(--bg-card);
  border: 2px solid transparent;
  background-clip: padding-box;
}

.card-front {
  border-color: var(--cyan);
  box-shadow: 0 0 30px rgba(0, 255, 245, 0.2), inset 0 0 30px rgba(0, 255, 245, 0.05);
}

.card-back {
  transform: rotateY(180deg);
  border-color: var(--magenta);
  box-shadow: 0 0 30px rgba(255, 0, 255, 0.2), inset 0 0 30px rgba(255, 0, 255, 0.05);
}

.card.mastered .card-front,
.card.mastered .card-back {
  border-color: var(--green);
  box-shadow: 0 0 30px rgba(0, 255, 136, 0.3), inset 0 0 30px rgba(0, 255, 136, 0.05);
}

.card-badge {
  font-family: 'Press Start 2P', monospace;
  font-size: 10px;
  color: var(--cyan);
  margin-bottom: 16px;
}

.answer-badge {
  color: var(--magenta);
}

.card-content {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.question-text,
.answer-text {
  font-size: 16px;
  line-height: 1.6;
  font-weight: 500;
}

.card-hint {
  font-family: 'Press Start 2P', monospace;
  font-size: 8px;
  color: var(--text-dim);
  text-align: center;
  animation: blink 2s ease-in-out infinite;
}

@keyframes blink {
  0%, 100% { opacity: 0.5; }
  50% { opacity: 1; }
}

/* Controls */
.controls {
  display: flex;
  gap: 12px;
  margin-bottom: 16px;
}

.ctrl-btn {
  flex: 1;
  padding: 14px 8px;
  border: none;
  border-radius: 8px;
  font-family: 'Press Start 2P', monospace;
  font-size: 9px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.ctrl-btn.prev,
.ctrl-btn.next {
  background: var(--bg-card);
  color: var(--text);
  border: 1px solid var(--text-dim);
}

.ctrl-btn.prev:hover:not(:disabled),
.ctrl-btn.next:hover:not(:disabled) {
  border-color: var(--cyan);
  color: var(--cyan);
}

.ctrl-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.ctrl-btn.master {
  background: transparent;
  color: var(--yellow);
  border: 2px solid var(--yellow);
}

.ctrl-btn.master:hover {
  background: rgba(255, 255, 0, 0.1);
  box-shadow: 0 0 20px rgba(255, 255, 0, 0.3);
}

.ctrl-btn.master.active {
  background: var(--green);
  border-color: var(--green);
  color: var(--bg-dark);
  box-shadow: var(--glow-green);
}

/* Counter */
.counter {
  text-align: center;
  font-family: 'Press Start 2P', monospace;
  font-size: 12px;
  color: var(--text-dim);
  margin-bottom: 24px;
}

/* Footer */
.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 16px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.reset-btn {
  background: transparent;
  border: 1px solid var(--text-dim);
  color: var(--text-dim);
  padding: 8px 16px;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.reset-btn:hover {
  border-color: #ff4444;
  color: #ff4444;
}

.footer-text {
  font-size: 14px;
  color: var(--text-dim);
}

/* Mobile optimizations */
@media (max-width: 400px) {
  .game-container {
    padding: 16px;
  }
  
  .logo-text {
    font-size: 11px;
  }
  
  .level-name {
    font-size: 7px;
  }
  
  .card {
    height: 260px;
  }
  
  .question-text,
  .answer-text {
    font-size: 14px;
  }
  
  .ctrl-btn {
    font-size: 8px;
    padding: 12px 6px;
  }
}
</style>
