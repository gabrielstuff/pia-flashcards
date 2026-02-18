<script setup lang="ts">
import { flashcards } from './data/flashcards'

const currentIndex = ref(0)
const showAnswer = ref(false)
const mode = ref<'sequential' | 'random'>('sequential')
const masteredCards = ref<Set<number>>(new Set())
const shuffledOrder = ref<number[]>([])

// Initialize shuffled order
const initShuffle = () => {
  shuffledOrder.value = [...Array(flashcards.length).keys()].sort(() => Math.random() - 0.5)
}

onMounted(() => {
  initShuffle()
  // Load mastered cards from localStorage
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
  } else {
    currentIndex.value = flashcards.length - 1
  }
}

const toggleAnswer = () => {
  showAnswer.value = !showAnswer.value
}

const markMastered = () => {
  masteredCards.value.add(currentCard.value.id)
  localStorage.setItem('pia-mastered', JSON.stringify([...masteredCards.value]))
  nextCard()
}

const resetProgress = () => {
  masteredCards.value.clear()
  localStorage.removeItem('pia-mastered')
}

const toggleMode = () => {
  mode.value = mode.value === 'sequential' ? 'random' : 'sequential'
  currentIndex.value = 0
  showAnswer.value = false
  if (mode.value === 'random') initShuffle()
}

// Keyboard shortcuts
onMounted(() => {
  window.addEventListener('keydown', (e) => {
    if (e.key === ' ' || e.key === 'Enter') {
      e.preventDefault()
      toggleAnswer()
    } else if (e.key === 'ArrowRight' || e.key === 'n') {
      nextCard()
    } else if (e.key === 'ArrowLeft' || e.key === 'p') {
      prevCard()
    } else if (e.key === 'm') {
      markMastered()
    }
  })
})
</script>

<template>
  <div class="app">
    <header>
      <h1>📚 Fiches de Révision</h1>
      <p class="subtitle">Technologie - 50 questions</p>
    </header>

    <div class="progress-bar">
      <div class="progress-fill" :style="{ width: progress + '%' }"></div>
      <span class="progress-text">{{ masteredCards.size }}/{{ flashcards.length }} maîtrisées ({{ progress }}%)</span>
    </div>

    <div class="controls-top">
      <button @click="toggleMode" class="mode-btn">
        {{ mode === 'sequential' ? '🔢 Ordre' : '🎲 Aléatoire' }}
      </button>
      <span class="card-counter">Carte {{ currentIndex + 1 }}/{{ flashcards.length }}</span>
      <button @click="resetProgress" class="reset-btn" v-if="masteredCards.size > 0">
        🔄 Reset
      </button>
    </div>

    <div 
      class="flashcard" 
      :class="{ flipped: showAnswer, mastered: masteredCards.has(currentCard.id) }"
      @click="toggleAnswer"
    >
      <div class="card-inner">
        <div class="card-front">
          <span class="card-number">#{{ currentCard.id }}</span>
          <p class="question">{{ currentCard.question }}</p>
          <span class="hint">👆 Touche pour voir la réponse</span>
        </div>
        <div class="card-back">
          <span class="card-number">#{{ currentCard.id }}</span>
          <p class="answer">{{ currentCard.answer }}</p>
          <span class="hint">👆 Touche pour revenir</span>
        </div>
      </div>
    </div>

    <div class="controls">
      <button @click="prevCard" class="nav-btn">⬅️ Précédent</button>
      <button @click="markMastered" class="master-btn" :disabled="masteredCards.has(currentCard.id)">
        {{ masteredCards.has(currentCard.id) ? '✅ Maîtrisée' : '✨ Je sais !' }}
      </button>
      <button @click="nextCard" class="nav-btn">Suivant ➡️</button>
    </div>

    <footer>
      <p>Raccourcis : <kbd>Espace</kbd> voir réponse · <kbd>←→</kbd> navigation · <kbd>M</kbd> maîtrisée</p>
    </footer>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  color: #333;
}

.app {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

header {
  text-align: center;
  color: white;
  margin-bottom: 20px;
}

header h1 {
  font-size: 1.8rem;
  margin-bottom: 5px;
}

.subtitle {
  opacity: 0.9;
  font-size: 1rem;
}

.progress-bar {
  background: rgba(255,255,255,0.3);
  border-radius: 20px;
  height: 30px;
  position: relative;
  margin-bottom: 15px;
  overflow: hidden;
}

.progress-fill {
  background: linear-gradient(90deg, #4ade80, #22c55e);
  height: 100%;
  border-radius: 20px;
  transition: width 0.3s ease;
}

.progress-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-weight: 600;
  font-size: 0.85rem;
  text-shadow: 0 1px 2px rgba(0,0,0,0.3);
}

.controls-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  gap: 10px;
}

.mode-btn, .reset-btn {
  background: rgba(255,255,255,0.2);
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background 0.2s;
}

.mode-btn:hover, .reset-btn:hover {
  background: rgba(255,255,255,0.3);
}

.card-counter {
  color: white;
  font-weight: 600;
}

.flashcard {
  perspective: 1000px;
  flex: 1;
  min-height: 300px;
  cursor: pointer;
  margin-bottom: 20px;
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 300px;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.flashcard.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front, .card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  min-height: 300px;
  backface-visibility: hidden;
  background: white;
  border-radius: 20px;
  padding: 30px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  box-shadow: 0 10px 40px rgba(0,0,0,0.2);
}

.card-back {
  transform: rotateY(180deg);
  background: linear-gradient(135deg, #f0fdf4, #dcfce7);
}

.flashcard.mastered .card-front,
.flashcard.mastered .card-back {
  border: 3px solid #22c55e;
}

.card-number {
  position: absolute;
  top: 15px;
  left: 20px;
  background: #667eea;
  color: white;
  padding: 5px 12px;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: 600;
}

.question {
  font-size: 1.3rem;
  line-height: 1.5;
  color: #1f2937;
}

.answer {
  font-size: 1.2rem;
  line-height: 1.5;
  color: #166534;
  font-weight: 500;
}

.hint {
  position: absolute;
  bottom: 15px;
  color: #9ca3af;
  font-size: 0.8rem;
}

.controls {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-bottom: 20px;
}

.nav-btn, .master-btn {
  padding: 12px 24px;
  border: none;
  border-radius: 12px;
  font-size: 1rem;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.nav-btn {
  background: white;
  color: #667eea;
}

.master-btn {
  background: linear-gradient(135deg, #4ade80, #22c55e);
  color: white;
  font-weight: 600;
}

.master-btn:disabled {
  background: #d1d5db;
  cursor: not-allowed;
}

.nav-btn:hover, .master-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

footer {
  text-align: center;
  color: rgba(255,255,255,0.8);
  font-size: 0.8rem;
}

footer kbd {
  background: rgba(255,255,255,0.2);
  padding: 2px 6px;
  border-radius: 4px;
  font-family: inherit;
}

@media (max-width: 480px) {
  .app {
    padding: 15px;
  }
  
  header h1 {
    font-size: 1.4rem;
  }
  
  .question, .answer {
    font-size: 1.1rem;
  }
  
  .controls {
    flex-wrap: wrap;
  }
  
  footer {
    display: none;
  }
}
</style>
