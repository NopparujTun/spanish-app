<template>
  <div id="app">
    <header class="app-header">
      <div class="header-content">
        <h1 class="app-title">
          <span class="flag">🇪🇸</span>
          เรียนภาษาสเปน A1
          <span class="flag">🇹🇭</span>
        </h1>
        <div class="overall-progress">
          <span>ความคืบหน้าทั้งหมด: {{ completedLessons }}/{{ lessons.length }}</span>
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: overallProgress + '%' }"></div>
          </div>
        </div>
      </div>
    </header>

    <main class="main-content">
      <div v-if="currentView === 'home'" class="home-view">
        <div class="welcome-section">
          <h2>ยินดีต้อนรับสู่การเรียนภาษาสเปน!</h2>
          <p>เริ่มต้นการเรียนรู้ภาษาสเปนระดับ A1 ด้วยบทเรียนที่สนุกและมีปฏิสัมพันธ์</p>
        </div>

        <div class="content-grid">
          <div class="lessons-section">
            <h3>📚 บทเรียน</h3>
            <div class="lessons-grid">
              <div 
                v-for="lesson in lessons" 
                :key="lesson.id"
                class="lesson-card"
                :class="{ 
                  locked: !isLessonUnlocked(lesson.id),
                  completed: isLessonCompleted(lesson.id)
                }"
                @click="openLesson(lesson)"
              >
                <div class="lesson-icon">{{ lesson.icon }}</div>
                <div class="lesson-content">
                  <h4>{{ lesson.title }}</h4>
                  <p>{{ lesson.description }}</p>
                  <div class="lesson-status">
                    <span v-if="isLessonCompleted(lesson.id)" class="status completed">✅ เสร็จสิ้น</span>
                    <span v-else-if="isLessonUnlocked(lesson.id)" class="status available">📖 พร้อมเรียน</span>
                    <span v-else class="status locked">🔒 ล็อค</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="quizzes-section">
            <h3>📝 แบบทดสอบ</h3>
            <div class="quizzes-grid">
              <div 
                v-for="quiz in quizzes" 
                :key="quiz.id"
                class="quiz-card"
                :class="{ 
                  locked: !isQuizUnlocked(quiz.id),
                  completed: isQuizCompleted(quiz.id)
                }"
                @click="openQuiz(quiz)"
              >
                <div class="quiz-icon">📝</div>
                <div class="quiz-content">
                  <h4>{{ quiz.title }}</h4>
                  <p>{{ quiz.description }}</p>
                  <div class="quiz-status">
                    <span v-if="isQuizCompleted(quiz.id)" class="status completed">✅ ผ่านแล้ว</span>
                    <span v-else-if="isQuizUnlocked(quiz.id)" class="status available">📝 พร้อมทำ</span>
                    <span v-else class="status locked">🔒 ล็อค</span>
                  </div>
                  <div v-if="getQuizScore(quiz.id)" class="quiz-score">
                    คะแนน: {{ getQuizScore(quiz.id) }}%
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <LessonView 
        v-if="currentView === 'lesson'" 
        :lesson="currentLesson"
        @back="goHome"
        @complete="completeLesson"
      />

      <QuizView 
        v-if="currentView === 'quiz'" 
        :quiz="currentQuiz"
        @back="goHome"
        @complete="completeQuiz"
      />
    </main>

    <footer class="app-footer">
      <p>© 2025 Spanish Learning App - สำหรับผู้เรียนชาวไทย</p>
    </footer>

    <!-- Audio Debug Panel (for development) -->
    <div v-if="showAudioDebug" class="audio-debug">
      <h4>Audio Debug</h4>
      <p>Last audio attempt: {{ lastAudioFile }}</p>
      <p>Audio status: {{ audioStatus }}</p>
      <button @click="testAudio">Test Audio</button>
    </div>
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
      currentView: 'home',
      currentLesson: null,
      currentQuiz: null,
      lessons: lessonsData,
      quizzes: quizzesData,
      completedLessons: [],
      completedQuizzes: [],
      quizScores: {},
      showAudioDebug: false, // Set to true for debugging
      lastAudioFile: '',
      audioStatus: 'ready'
    }
  },
  computed: {
    overallProgress() {
      return (this.completedLessons.length / this.lessons.length) * 100
    }
  },
  mounted() {
    this.loadProgress()
    // Enable audio debug in development
    if (process.env.NODE_ENV === 'development') {
      this.showAudioDebug = true
    }
  },
  methods: {
    openLesson(lesson) {
      if (this.isLessonUnlocked(lesson.id)) {
        this.currentLesson = lesson
        this.currentView = 'lesson'
      }
    },
    
    openQuiz(quiz) {
      if (this.isQuizUnlocked(quiz.id)) {
        this.currentQuiz = quiz
        this.currentView = 'quiz'
      }
    },
    
    goHome() {
      this.currentView = 'home'
      this.currentLesson = null
      this.currentQuiz = null
    },
    
    completeLesson(lessonId) {
      if (!this.completedLessons.includes(lessonId)) {
        this.completedLessons.push(lessonId)
        this.saveProgress()
      }
      this.goHome()
    },
    
    completeQuiz(quizId, score) {
      if (!this.completedQuizzes.includes(quizId)) {
        this.completedQuizzes.push(quizId)
      }
      if (score !== undefined) {
        this.quizScores[quizId] = score
      }
      this.saveProgress()
      this.goHome()
    },
    
    isLessonUnlocked(lessonId) {
      if (lessonId === 1) return true
      return this.completedLessons.includes(lessonId - 1)
    },
    
    isLessonCompleted(lessonId) {
      return this.completedLessons.includes(lessonId)
    },
    
    isQuizUnlocked(quizId) {
      return this.completedLessons.includes(quizId)
    },
    
    isQuizCompleted(quizId) {
      return this.completedQuizzes.includes(quizId)
    },
    
    getQuizScore(quizId) {
      return this.quizScores[quizId]
    },
    
    saveProgress() {
      const progress = {
        completedLessons: this.completedLessons,
        completedQuizzes: this.completedQuizzes,
        quizScores: this.quizScores
      }
      localStorage.setItem('spanishLearningProgress', JSON.stringify(progress))
    },
    
    loadProgress() {
      const saved = localStorage.getItem('spanishLearningProgress')
      if (saved) {
        const progress = JSON.parse(saved)
        this.completedLessons = progress.completedLessons || []
        this.completedQuizzes = progress.completedQuizzes || []
        this.quizScores = progress.quizScores || {}
      }
    },

    // Audio testing method
    testAudio() {
      this.playAudio('/audio/test.mp3')
    },

    // Global audio method that can be used by child components
    playAudio(audioFile) {
      this.lastAudioFile = audioFile
      this.audioStatus = 'loading'
      
      if (!audioFile) {
        this.audioStatus = 'no file'
        console.warn('No audio file provided')
        return
      }

      try {
        // Create a simple beep sound as fallback for missing audio files
        if (audioFile.includes('placeholder') || !audioFile) {
          this.createBeepSound()
          return
        }

        const audio = new Audio(audioFile)
        
        audio.addEventListener('loadstart', () => {
          this.audioStatus = 'loading'
        })
        
        audio.addEventListener('canplay', () => {
          this.audioStatus = 'ready'
        })
        
        audio.addEventListener('play', () => {
          this.audioStatus = 'playing'
        })
        
        audio.addEventListener('ended', () => {
          this.audioStatus = 'ended'
        })
        
        audio.addEventListener('error', (e) => {
          this.audioStatus = 'error: ' + e.message
          console.warn('Audio failed to load:', audioFile, e)
          // Fallback to beep sound
          this.createBeepSound()
        })

        audio.play().catch(e => {
          this.audioStatus = 'play error: ' + e.message
          console.warn('Audio play failed:', audioFile, e)
          // Fallback to beep sound
          this.createBeepSound()
        })
        
      } catch (error) {
        this.audioStatus = 'catch error: ' + error.message
        console.error('Audio error:', error)
        this.createBeepSound()
      }
    },

    // Create a simple beep sound as fallback
    createBeepSound() {
      try {
        const audioContext = new (window.AudioContext || window.webkitAudioContext)()
        const oscillator = audioContext.createOscillator()
        const gainNode = audioContext.createGain()
        
        oscillator.connect(gainNode)
        gainNode.connect(audioContext.destination)
        
        oscillator.frequency.value = 800
        oscillator.type = 'sine'
        
        gainNode.gain.setValueAtTime(0.3, audioContext.currentTime)
        gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.3)
        
        oscillator.start(audioContext.currentTime)
        oscillator.stop(audioContext.currentTime + 0.3)
        
        this.audioStatus = 'beep played'
      } catch (error) {
        this.audioStatus = 'beep failed'
        console.error('Beep sound failed:', error)
      }
    }
  },

  // Provide audio method to child components
  provide() {
    return {
      playAudio: this.playAudio
    }
  }
}
</script>

<style>
/* Reset and base styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  line-height: 1.6;
  color: #2d3748;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Header */
.app-header {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  padding: 1rem 0;
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.app-title {
  font-size: 1.8rem;
  font-weight: 700;
  color: #2d3748;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.flag {
  font-size: 1.5rem;
}

.overall-progress {
  display: flex;
  align-items: center;
  gap: 1rem;
  font-weight: 600;
  color: #4a5568;
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

/* Main content */
.main-content {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  width: 100%;
}

.home-view {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 2rem;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.welcome-section {
  text-align: center;
  margin-bottom: 3rem;
}

.welcome-section h2 {
  font-size: 2rem;
  color: #2d3748;
  margin-bottom: 1rem;
}

.welcome-section p {
  font-size: 1.1rem;
  color: #718096;
  max-width: 600px;
  margin: 0 auto;
}

.content-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 3rem;
}

.lessons-section h3,
.quizzes-section h3 {
  font-size: 1.5rem;
  color: #2d3748;
  margin-bottom: 1.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.lessons-grid,
.quizzes-grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

/* Cards */
.lesson-card,
.quiz-card {
  background: white;
  border-radius: 15px;
  padding: 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
}

.lesson-card:hover,
.quiz-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
  border-color: #4299e1;
}

.lesson-card.locked,
.quiz-card.locked {
  opacity: 0.6;
  cursor: not-allowed;
  background: #f7fafc;
}

.lesson-card.locked:hover,
.quiz-card.locked:hover {
  transform: none;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
  border-color: transparent;
}

.lesson-card.completed {
  border-color: #48bb78;
  background: linear-gradient(135deg, #f0fff4 0%, #c6f6d5 100%);
}

.lesson-icon,
.quiz-icon {
  font-size: 3rem;
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f7fafc;
  border-radius: 15px;
  flex-shrink: 0;
}

.lesson-content,
.quiz-content {
  flex: 1;
}

.lesson-content h4,
.quiz-content h4 {
  font-size: 1.25rem;
  color: #2d3748;
  margin-bottom: 0.5rem;
}

.lesson-content p,
.quiz-content p {
  color: #718096;
  margin-bottom: 1rem;
  line-height: 1.5;
}

.lesson-status,
.quiz-status {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.status {
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.875rem;
  font-weight: 600;
}

.status.completed {
  background: #c6f6d5;
  color: #22543d;
}

.status.available {
  background: #bee3f8;
  color: #2c5282;
}

.status.locked {
  background: #e2e8f0;
  color: #718096;
}

.quiz-score {
  background: #fbb6ce;
  color: #97266d;
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.875rem;
  font-weight: 600;
  display: inline-block;
}

/* Footer */
.app-footer {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  text-align: center;
  padding: 1rem;
  color: white;
  font-size: 0.875rem;
}

/* Audio Debug Panel */
.audio-debug {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 1rem;
  border-radius: 8px;
  font-size: 0.875rem;
  z-index: 1000;
  max-width: 300px;
}

.audio-debug h4 {
  margin-bottom: 0.5rem;
}

.audio-debug p {
  margin: 0.25rem 0;
}

.audio-debug button {
  background: #4299e1;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 0.5rem;
}

/* Responsive design */
@media (max-width: 1024px) {
  .content-grid {
    grid-template-columns: 1fr;
    gap: 2rem;
  }
  
  .header-content {
    flex-direction: column;
    text-align: center;
  }
  
  .overall-progress {
    flex-direction: column;
    gap: 0.5rem;
  }
}

@media (max-width: 768px) {
  .main-content {
    padding: 1rem;
  }
  
  .home-view {
    padding: 1rem;
  }
  
  .lesson-card,
  .quiz-card {
    flex-direction: column;
    text-align: center;
  }
  
  .lesson-icon,
  .quiz-icon {
    width: 60px;
    height: 60px;
    font-size: 2rem;
  }
  
  .app-title {
    font-size: 1.5rem;
  }
  
  .progress-bar {
    width: 150px;
  }
}

@media (max-width: 480px) {
  .header-content {
    padding: 0 1rem;
  }
  
  .welcome-section h2 {
    font-size: 1.5rem;
  }
  
  .lesson-content h4,
  .quiz-content h4 {
    font-size: 1.1rem;
  }
}
</style>