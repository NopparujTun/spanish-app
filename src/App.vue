<template>
  <div id="app">
    <header class="app-header">
      <h1>🇪🇸 เรียนภาษาสเปน A1</h1>
      <div class="progress-overview">
        <span>ความคืบหน้า: {{ completedLessons }}/{{ totalLessons }} บทเรียน</span>
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
        </div>
      </div>
    </header>

    <nav class="main-nav">
      <button 
        @click="currentView = 'lessons'" 
        :class="{ active: currentView === 'lessons' }"
        class="nav-btn"
      >
        📚 บทเรียน
      </button>
      <button 
        @click="currentView = 'quizzes'" 
        :class="{ active: currentView === 'quizzes' }"
        class="nav-btn"
      >
        🎯 แบบทดสอบ
      </button>
    </nav>

    <main class="main-content">
      <!-- Lessons View -->
      <div v-if="currentView === 'lessons'" class="lessons-view">
        <div v-if="!selectedLesson" class="lessons-grid">
          <div 
            v-for="lesson in lessons" 
            :key="lesson.id"
            class="lesson-card"
            :class="{ 
              completed: isLessonCompleted(lesson.id),
              locked: !isLessonUnlocked(lesson.id)
            }"
            @click="selectLesson(lesson)"
          >
            <div class="lesson-icon">{{ lesson.icon }}</div>
            <h3>{{ lesson.title }}</h3>
            <p>{{ lesson.description }}</p>
            <div class="lesson-status">
              <span v-if="isLessonCompleted(lesson.id)" class="status completed">✅ เสร็จแล้ว</span>
              <span v-else-if="!isLessonUnlocked(lesson.id)" class="status locked">🔒 ล็อค</span>
              <span v-else class="status available">▶️ เริ่มเรียน</span>
            </div>
          </div>
        </div>

        <LessonView 
          v-if="selectedLesson" 
          :lesson="selectedLesson"
          @complete="completeLesson"
          @back="selectedLesson = null"
        />
      </div>

      <!-- Quizzes View -->
      <div v-if="currentView === 'quizzes'" class="quizzes-view">
        <div v-if="!selectedQuiz" class="quizzes-grid">
          <div 
            v-for="quiz in quizzes" 
            :key="quiz.id"
            class="quiz-card"
            :class="{ 
              completed: isQuizCompleted(quiz.id),
              locked: !isQuizUnlocked(quiz.id)
            }"
            @click="selectQuiz(quiz)"
          >
            <div class="quiz-icon">🎯</div>
            <h3>{{ quiz.title }}</h3>
            <p>{{ quiz.description }}</p>
            <div class="quiz-status">
              <span v-if="isQuizCompleted(quiz.id)" class="status completed">✅ เสร็จแล้ว</span>
              <span v-else-if="!isQuizUnlocked(quiz.id)" class="status locked">🔒 ล็อค</span>
              <span v-else class="status available">🎯 ทำแบบทดสอบ</span>
            </div>
          </div>
        </div>

        <QuizView 
          v-if="selectedQuiz" 
          :quiz="selectedQuiz"
          @complete="completeQuiz"
          @back="selectedQuiz = null"
        />
      </div>
    </main>
  </div>
</template>

<script>
import LessonView from './components/LessonView.vue'
import QuizView from './components/QuizView.vue'
import lessonsData from './data/lessons.json'
import quizzesData from './data/quizzes.json'

export default {
  name: 'App',
  components: {
    LessonView,
    QuizView
  },
  data() {
    return {
      currentView: 'lessons',
      selectedLesson: null,
      selectedQuiz: null,
      lessons: lessonsData,
      quizzes: quizzesData,
      completedLessons: this.getCompletedLessons(),
      completedQuizzes: this.getCompletedQuizzes()
    }
  },
  computed: {
    totalLessons() {
      return this.lessons.length
    },
    progressPercentage() {
      return (this.completedLessons.length / this.totalLessons) * 100
    }
  },
  methods: {
    getCompletedLessons() {
      const completed = localStorage.getItem('completedLessons')
      return completed ? JSON.parse(completed) : []
    },
    getCompletedQuizzes() {
      const completed = localStorage.getItem('completedQuizzes')
      return completed ? JSON.parse(completed) : []
    },
    isLessonCompleted(lessonId) {
      return this.completedLessons.includes(lessonId)
    },
    isLessonUnlocked(lessonId) {
      if (lessonId === 1) return true
      return this.completedLessons.includes(lessonId - 1)
    },
    isQuizCompleted(quizId) {
      return this.completedQuizzes.includes(quizId)
    },
    isQuizUnlocked(quizId) {
      return this.completedLessons.includes(quizId)
    },
    selectLesson(lesson) {
      if (this.isLessonUnlocked(lesson.id)) {
        this.selectedLesson = lesson
      }
    },
    selectQuiz(quiz) {
      if (this.isQuizUnlocked(quiz.id)) {
        this.selectedQuiz = quiz
      }
    },
    completeLesson(lessonId) {
      if (!this.completedLessons.includes(lessonId)) {
        this.completedLessons.push(lessonId)
        localStorage.setItem('completedLessons', JSON.stringify(this.completedLessons))
      }
      this.selectedLesson = null
    },
    completeQuiz(quizId) {
      if (!this.completedQuizzes.includes(quizId)) {
        this.completedQuizzes.push(quizId)
        localStorage.setItem('completedQuizzes', JSON.stringify(this.completedQuizzes))
      }
      this.selectedQuiz = null
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  color: #333;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  background: rgba(255, 255, 255, 0.95);
  padding: 1rem 2rem;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.app-header h1 {
  font-size: 2rem;
  color: #4a5568;
  margin-bottom: 1rem;
  text-align: center;
}

.progress-overview {
  display: flex;
  align-items: center;
  gap: 1rem;
  justify-content: center;
  flex-wrap: wrap;
}

.progress-bar {
  width: 200px;
  height: 8px;
  background: #e2e8f0;
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #48bb78, #38a169);
  transition: width 0.3s ease;
}

.main-nav {
  display: flex;
  justify-content: center;
  gap: 1rem;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.1);
}

.nav-btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 25px;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.nav-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

.nav-btn.active {
  background: rgba(255, 255, 255, 0.9);
  color: #4a5568;
}

.main-content {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

.lessons-grid, .quizzes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
}

.lesson-card, .quiz-card {
  background: white;
  border-radius: 15px;
  padding: 1.5rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
}

.lesson-card:hover, .quiz-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.lesson-card.completed, .quiz-card.completed {
  border-color: #48bb78;
  background: linear-gradient(135deg, #f0fff4, #c6f6d5);
}

.lesson-card.locked, .quiz-card.locked {
  opacity: 0.6;
  cursor: not-allowed;
  background: #f7fafc;
}

.lesson-card.locked:hover, .quiz-card.locked:hover {
  transform: none;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.lesson-icon, .quiz-icon {
  font-size: 3rem;
  text-align: center;
  margin-bottom: 1rem;
}

.lesson-card h3, .quiz-card h3 {
  font-size: 1.25rem;
  margin-bottom: 0.5rem;
  color: #2d3748;
}

.lesson-card p, .quiz-card p {
  color: #718096;
  margin-bottom: 1rem;
  line-height: 1.5;
}

.lesson-status, .quiz-status {
  text-align: center;
}

.status {
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-weight: 600;
  font-size: 0.9rem;
}

.status.completed {
  background: #c6f6d5;
  color: #22543d;
}

.status.locked {
  background: #e2e8f0;
  color: #718096;
}

.status.available {
  background: #bee3f8;
  color: #2c5282;
}

@media (max-width: 768px) {
  .app-header {
    padding: 1rem;
  }

  .app-header h1 {
    font-size: 1.5rem;
  }

  .progress-overview {
    flex-direction: column;
    gap: 0.5rem;
  }

  .main-content {
    padding: 1rem;
  }

  .lessons-grid, .quizzes-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .nav-btn {
    padding: 0.5rem 1rem;
    font-size: 0.9rem;
  }
}
</style>