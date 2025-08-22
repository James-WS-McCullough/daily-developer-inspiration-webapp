<template>
  <div class="daily-inspiration">
    <header class="header">
      <div class="container">
        <h1 class="title">Daily Developer Inspiration</h1>
        <div class="date-info">
          <span class="current-date">{{ formatDate(currentDate) }}</span>
          <span class="day-counter">Day {{ dayIndex + 1 }} of {{ totalArticles }}</span>
        </div>
      </div>
    </header>

    <main class="main-content">
      <div class="container">
        <article class="article" v-if="currentArticle">
          <div class="article-header">
            <h2 class="article-title">{{ currentArticle.title }}</h2>
            <p class="article-author">by {{ currentArticle.author }}</p>
          </div>
          
          <div class="article-content" v-html="renderedContent"></div>
        </article>

        <div class="navigation">
          <button 
            @click="goToPreviousDay" 
            :disabled="dayIndex === 0"
            class="nav-button prev"
          >
            ← Previous Day
          </button>
          
          <button @click="goToToday" class="nav-button today">
            Today
          </button>
          
          <button 
            @click="goToNextDay" 
            :disabled="dayIndex >= totalArticles - 1"
            class="nav-button next"
          >
            Next Day →
          </button>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { marked } from 'marked'
import articlesData from '../data/articles.json'

// Configuration - you can change this start date
const START_DATE = new Date('2025-01-01') // Articles start from January 1st, 2025

const articles = articlesData.inspiration
const totalArticles = articles.length
const currentDate = ref(new Date())

// Calculate which day we're on based on the start date
const daysSinceStart = computed(() => {
  const today = new Date()
  const diffTime = today.getTime() - START_DATE.getTime()
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24))
  return Math.max(0, diffDays) // Don't go negative
})

const dayIndex = ref(daysSinceStart.value % totalArticles)

const currentArticle = computed(() => {
  return articles[dayIndex.value]
})

const renderedContent = computed(() => {
  if (!currentArticle.value) return ''
  
  // Configure marked for better formatting
  marked.setOptions({
    breaks: true,
    gfm: true,
  })
  
  return marked(currentArticle.value.content)
})

const formatDate = (date) => {
  return date.toLocaleDateString('en-US', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

const goToPreviousDay = () => {
  if (dayIndex.value > 0) {
    dayIndex.value--
    updateCurrentDate()
  }
}

const goToNextDay = () => {
  if (dayIndex.value < totalArticles - 1) {
    dayIndex.value++
    updateCurrentDate()
  }
}

const goToToday = () => {
  dayIndex.value = daysSinceStart.value % totalArticles
  currentDate.value = new Date()
}

const updateCurrentDate = () => {
  const newDate = new Date(START_DATE)
  newDate.setDate(START_DATE.getDate() + dayIndex.value)
  currentDate.value = newDate
}

onMounted(() => {
  updateCurrentDate()
})
</script>

<style scoped>
.daily-inspiration {
  min-height: 100vh;
  background: linear-gradient(135deg, #0f0f0f 0%, #1a1a1a 100%);
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 2rem;
}

.header {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  padding: 2rem 0;
  position: sticky;
  top: 0;
  z-index: 100;
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  background: linear-gradient(135deg, #60a5fa, #34d399);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0.5rem;
}

.date-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.current-date {
  font-size: 1.125rem;
  color: #d1d5db;
}

.day-counter {
  font-size: 0.875rem;
  color: #9ca3af;
  background: rgba(255, 255, 255, 0.1);
  padding: 0.25rem 0.75rem;
  border-radius: 0.5rem;
}

.main-content {
  padding: 3rem 0;
}

.article {
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 1rem;
  padding: 2.5rem;
  margin-bottom: 3rem;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.3);
}

.article-header {
  margin-bottom: 2rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.article-title {
  font-size: 2rem;
  font-weight: 600;
  color: #f9fafb;
  margin-bottom: 0.5rem;
  line-height: 1.2;
}

.article-author {
  font-size: 1rem;
  color: #9ca3af;
  font-style: italic;
}

.article-content {
  font-size: 1.125rem;
  line-height: 1.8;
  color: #e5e7eb;
}

/* Markdown content styling */
.article-content :deep(p) {
  margin-bottom: 1.5rem;
}

.article-content :deep(p:last-child) {
  margin-bottom: 0;
}

.article-content :deep(strong) {
  color: #f9fafb;
  font-weight: 600;
}

.article-content :deep(em) {
  color: #d1d5db;
}

.article-content :deep(code) {
  background: rgba(255, 255, 255, 0.1);
  padding: 0.125rem 0.375rem;
  border-radius: 0.25rem;
  font-family: 'Fira Code', Consolas, monospace;
  font-size: 0.9em;
}

.article-content :deep(blockquote) {
  border-left: 3px solid #60a5fa;
  padding-left: 1.5rem;
  margin: 2rem 0;
  font-style: italic;
  color: #d1d5db;
}

.navigation {
  display: flex;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.nav-button {
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: #e5e7eb;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s ease;
  backdrop-filter: blur(10px);
}

.nav-button:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

.nav-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.nav-button.today {
  background: linear-gradient(135deg, #60a5fa, #34d399);
  border: none;
  color: #000;
  font-weight: 600;
}

.nav-button.today:hover {
  background: linear-gradient(135deg, #3b82f6, #10b981);
  transform: translateY(-2px);
}

/* Responsive design */
@media (max-width: 768px) {
  .container {
    padding: 0 1rem;
  }
  
  .title {
    font-size: 2rem;
  }
  
  .article {
    padding: 1.5rem;
  }
  
  .article-title {
    font-size: 1.5rem;
  }
  
  .article-content {
    font-size: 1rem;
  }
  
  .date-info {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>
