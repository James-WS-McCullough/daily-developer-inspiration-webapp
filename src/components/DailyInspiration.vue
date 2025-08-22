<template>
  <div class="daily-inspiration">
    <header class="header" :class="{ 'header-minimized': isScrolled }">
      <div class="container">
        <h1 class="title">📚 Daily Developer Wisdom</h1>
        <div class="date-info" v-show="!isScrolled || !isMobile">
          <span class="current-date">{{ formatDate(currentDate) }}</span>
          <span class="day-counter" v-if="isDeveloperMode">Day {{ weekdayIndex + 1 }} of {{ totalArticles }}</span>
        </div>
        
        <!-- Navigation moved to header -->
        <div class="header-navigation" v-if="hasArticlesAvailable">
          <button 
            @click="goToPreviousDay" 
            :disabled="weekdayIndex === 0"
            class="nav-button prev"
            title="Previous Day"
          >
            Previous
          </button>
          
          <button @click="goToToday" class="nav-button today" title="Today">
            Today
          </button>
          
          <button 
            @click="goToNextDay" 
            :disabled="!canGoToNextDay"
            class="nav-button next"
            :title="nextButtonTooltip"
          >
            {{ nextButtonText }}
          </button>
        </div>
      </div>
    </header>

    <main class="main-content">
      <div class="container">
        <!-- No articles available yet -->
        <div class="no-articles" v-if="!hasArticlesAvailable">
          <div class="no-articles-content">
            <h2 class="no-articles-title">Coming Soon</h2>
            <p class="no-articles-message">
              Daily Developer Wisdom starts on 
              <strong>{{ formatDate(new Date(START_DATE)) }}</strong>
            </p>
            <p class="no-articles-subtitle">
              Check back then for your first dose of programming wisdom!
            </p>
          </div>
        </div>

        <!-- Article content -->
        <article class="article" v-else-if="currentArticle">
          <div class="article-header">
            <h2 class="article-title">{{ currentArticle.title }}</h2>
            <p class="article-author">by {{ currentArticle.author }}</p>
          </div>
          
          <div class="article-content" v-html="renderedContent"></div>
        </article>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { marked } from 'marked'
import articlesData from '../data/articles.json'

// Configuration
const START_DATE = new Date('2025-09-01') // Articles start date
const isDeveloperMode = import.meta.env.DEV // Enable developer features in development

const articles = articlesData.inspiration
const totalArticles = articles.length
const currentDate = ref(new Date())
const isScrolled = ref(false)
const isMobile = ref(false)

// Calculate weekdays only since start date
const getWeekdaysSinceStart = () => {
  const today = new Date()
  let current = new Date(START_DATE)
  let weekdays = 0
  
  while (current <= today) {
    const dayOfWeek = current.getDay()
    // Monday = 1, Tuesday = 2, ..., Friday = 5
    if (dayOfWeek >= 1 && dayOfWeek <= 5) {
      weekdays++
    }
    current.setDate(current.getDate() + 1)
  }
  
  return Math.max(0, weekdays - 1) // Subtract 1 since we want 0-based indexing
}

const weekdayIndex = ref(getWeekdaysSinceStart() % totalArticles)

const hasArticlesAvailable = computed(() => {
  const today = new Date()
  const startDate = new Date(START_DATE)
  return today >= startDate || isDeveloperMode
})

const currentArticle = computed(() => {
  if (!hasArticlesAvailable.value) return null
  return articles[weekdayIndex.value]
})

const canGoToNextDay = computed(() => {
  const todayIndex = getWeekdaysSinceStart() % totalArticles
  
  if (isDeveloperMode) {
    // In dev mode, can go to any article
    return weekdayIndex.value < totalArticles - 1
  } else {
    // In production, can only go up to today's article
    return weekdayIndex.value < todayIndex
  }
})

const getNextUnlockDay = () => {
  const today = new Date()
  const tomorrow = new Date(today)
  tomorrow.setDate(today.getDate() + 1)
  
  // Find the next weekday
  while (tomorrow.getDay() === 0 || tomorrow.getDay() === 6) {
    tomorrow.setDate(tomorrow.getDate() + 1)
  }
  
  return tomorrow
}

const nextButtonText = computed(() => {
  if (isDeveloperMode || canGoToNextDay.value) {
    return 'Next'
  } else {
    const nextUnlock = getNextUnlockDay()
    const today = new Date()
    const tomorrow = new Date(today)
    tomorrow.setDate(today.getDate() + 1)
    
    if (nextUnlock.toDateString() === tomorrow.toDateString()) {
      return 'Unlocks Tomorrow'
    } else {
      const dayName = nextUnlock.toLocaleDateString('en-US', { weekday: 'long' })
      return `Unlocks ${dayName}`
    }
  }
})

const nextButtonTooltip = computed(() => {
  if (isDeveloperMode || canGoToNextDay.value) {
    return 'Next Day'
  } else {
    const nextUnlock = getNextUnlockDay()
    return `Next article unlocks on ${nextUnlock.toLocaleDateString('en-US', { 
      weekday: 'long', 
      month: 'short', 
      day: 'numeric' 
    })}`
  }
})

const renderedContent = computed(() => {
  if (!currentArticle.value) return ''
  
  // Configure marked for better formatting
  marked.setOptions({
    breaks: true, // This will convert single \n to <br> tags
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
  if (weekdayIndex.value > 0) {
    weekdayIndex.value--
    updateCurrentDate()
    scrollToTop()
  }
}

const goToNextDay = () => {
  if (canGoToNextDay.value) {
    weekdayIndex.value++
    updateCurrentDate()
    scrollToTop()
  }
}

const goToToday = () => {
  weekdayIndex.value = getWeekdaysSinceStart() % totalArticles
  currentDate.value = new Date()
  scrollToTop()
}

const updateCurrentDate = () => {
  // Calculate the actual date for the current weekday index
  let current = new Date(START_DATE)
  let weekdaysCount = 0
  
  while (weekdaysCount < weekdayIndex.value) {
    current.setDate(current.getDate() + 1)
    const dayOfWeek = current.getDay()
    if (dayOfWeek >= 1 && dayOfWeek <= 5) {
      weekdaysCount++
    }
  }
  
  currentDate.value = current
}

// Scroll handling for header minimization
const handleScroll = () => {
  isScrolled.value = window.scrollY > 100
}

// Smooth scroll to top
const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: 'instant'
  })
}

// Mobile detection
const handleResize = () => {
  isMobile.value = window.innerWidth < 768
}

onMounted(() => {
  updateCurrentDate()
  window.addEventListener('scroll', handleScroll)
  window.addEventListener('resize', handleResize)
  handleResize() // Initial check
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
  window.removeEventListener('resize', handleResize)
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
  transition: all 0.3s ease;
}

.header-minimized {
  padding: 1rem 0;
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(15px);
}

.header-minimized .title {
  font-size: 1.8rem;
  margin-bottom: 0;
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  background: linear-gradient(135deg, #60a5fa, #34d399);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0.5rem;
  transition: all 0.3s ease;
}

.date-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
  transition: all 0.3s ease;
  margin-bottom: 1rem;
}

.header-navigation {
  display: flex;
  justify-content: center;
  gap: 1rem;
  align-items: center;
}

.nav-button {
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: #e5e7eb;
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  font-size: 0.875rem;
  cursor: pointer;
  transition: all 0.2s ease;
  backdrop-filter: blur(10px);
  white-space: nowrap;
  min-width: fit-content;
}

.nav-button:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

.nav-button:disabled {
  opacity: 0.3;
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

.no-articles {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 60vh;
}

.no-articles-content {
  text-align: center;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 1rem;
  padding: 3rem 2rem;
  max-width: 500px;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.3);
}

.no-articles-title {
  font-size: 2.5rem;
  font-weight: 600;
  background: linear-gradient(135deg, #60a5fa, #34d399);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 1.5rem;
}

.no-articles-message {
  font-size: 1.125rem;
  color: #e5e7eb;
  margin-bottom: 1rem;
  line-height: 1.6;
}

.no-articles-subtitle {
  font-size: 1rem;
  color: #9ca3af;
  font-style: italic;
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
  margin-bottom: 2rem;
}

.article-content :deep(p:last-child) {
  margin-bottom: 0;
}

.article-content :deep(br) {
  display: block;
  margin: 1.5rem 0;
  content: "";
}

.article-content :deep(ul) {
  margin: 2rem 0;
  padding-left: 2rem;
}

.article-content :deep(li) {
  margin-bottom: 1rem;
  line-height: 1.8;
}

.article-content :deep(li:last-child) {
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

/* Responsive design */
@media (max-width: 768px) {
  .container {
    padding: 0 1rem;
  }
  
  .title {
    font-size: 2rem;
  }
  
  .header-minimized .title {
    font-size: 1.5rem;
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
  
  .header-navigation {
    margin-top: 0.5rem;
    gap: 0.5rem;
  }
  
  .nav-button {
    padding: 0.375rem 0.75rem;
    font-size: 0.75rem;
  }
  
  .no-articles-content {
    padding: 2rem 1.5rem;
    margin: 0 1rem;
  }
  
  .no-articles-title {
    font-size: 2rem;
  }
}
</style>
