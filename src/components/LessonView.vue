<template>
  <div class="lesson-view">
    <div class="lesson-header">
      <button @click="$emit('back')" class="back-btn">← กลับ</button>
      <h2>{{ lesson.title }}</h2>
      <div class="lesson-progress">
        {{ currentSection + 1 }}/{{ lesson.sections.length }}
      </div>
    </div>

    <div class="lesson-progress-bar">
      <div class="progress-fill" :style="{ width: sectionProgress + '%' }"></div>
    </div>

    <div class="lesson-content">
      <div v-if="currentSectionData.type === 'vocabulary'" class="vocabulary-section">
        <h3>{{ currentSectionData.title }}</h3>
        <div class="vocabulary-grid">
          <div 
            v-for="(word, index) in currentSectionData.words" 
            :key="index"
            class="vocabulary-card"
          >
            <div class="word-image">
              <img :src="word.image" :alt="word.spanish" />
            </div>
            <div class="word-content">
              <h4 class="spanish-word">{{ word.spanish }}</h4>
              <p class="thai-translation">{{ word.thai }}</p>
              <button @click="playAudio(word.audio)" class="audio-btn">🔊 ฟัง</button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="currentSectionData.type === 'grammar'" class="grammar-section">
        <h3>{{ currentSectionData.title }}</h3>
        <div class="grammar-content">
          <div class="explanation">
            <p>{{ currentSectionData.explanation }}</p>
          </div>
          <div class="examples">
            <h4>ตัวอย่าง:</h4>
            <div v-for="(example, index) in currentSectionData.examples" :key="index" class="example">
              <p class="spanish">{{ example.spanish }}</p>
              <p class="thai">{{ example.thai }}</p>
              <button @click="playAudio(example.audio)" class="audio-btn">🔊</button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="currentSectionData.type === 'dialogue'" class="dialogue-section">
        <h3>{{ currentSectionData.title }}</h3>
        <div class="dialogue-container">
          <div 
            v-for="(message, index) in currentSectionData.messages" 
            :key="index"
            class="dialogue-message"
            :class="{ 'speaker-a': message.speaker === 'A', 'speaker-b': message.speaker === 'B' }"
          >
            <div class="speaker-avatar">{{ message.speaker }}</div>
            <div class="message-content">
              <p class="spanish">{{ message.spanish }}</p>
              <p class="thai">{{ message.thai }}</p>
              <button @click="playAudio(message.audio)" class="audio-btn">🔊</button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="currentSectionData.type === 'practice'" class="practice-section">
        <h3>{{ currentSectionData.title }}</h3>
        <PracticeActivity 
          :activity="currentSectionData"
          @complete="onPracticeComplete"
        />
      </div>
    </div>

    <div class="lesson-navigation">
      <button 
        @click="previousSection" 
        :disabled="currentSection === 0"
        class="nav-btn prev"
      >
        ← ก่อนหน้า
      </button>
      
      <button 
        v-if="currentSection < lesson.sections.length - 1"
        @click="nextSection" 
        class="nav-btn next"
      >
        ถัดไป →
      </button>
      
      <button 
        v-else
        @click="completeLesson" 
        class="nav-btn complete"
      >
        เสร็จสิ้นบทเรียน ✅
      </button>
    </div>
  </div>
</template>

<script>
import PracticeActivity from './PracticeActivity.vue'

export default {
  name: 'LessonView',
  components: {
    PracticeActivity
  },
  inject: ['playAudio'],
  props: {
    lesson: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      currentSection: 0,
      practiceCompleted: false
    }
  },
  computed: {
    currentSectionData() {
      return this.lesson.sections[this.currentSection]
    },
    sectionProgress() {
      return ((this.currentSection + 1) / this.lesson.sections.length) * 100
    }
  },
  methods: {
    nextSection() {
      if (this.currentSection < this.lesson.sections.length - 1) {
        this.currentSection++
        this.practiceCompleted = false
      }
    },
    previousSection() {
      if (this.currentSection > 0) {
        this.currentSection--
        this.practiceCompleted = false
      }
    },
    completeLesson() {
      this.$emit('complete', this.lesson.id)
    },
    onPracticeComplete() {
      this.practiceCompleted = true
    },
    playAudio(audioFile) {
      // Use the injected playAudio method from parent
      this.playAudio(audioFile)
    }
  }
}
</script>

<style scoped>
.lesson-view {
  background: white;
  border-radius: 15px;
  padding: 2rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.lesson-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1rem;
  flex-wrap: wrap;
  gap: 1rem;
}

.back-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 8px;
  background: #e2e8f0;
  color: #4a5568;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.back-btn:hover {
  background: #cbd5e0;
}

.lesson-header h2 {
  color: #2d3748;
  font-size: 1.5rem;
}

.lesson-progress {
  background: #bee3f8;
  color: #2c5282;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-weight: 600;
}

.lesson-progress-bar {
  width: 100%;
  height: 8px;
  background: #e2e8f0;
  border-radius: 4px;
  margin-bottom: 2rem;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #48bb78, #38a169);
  transition: width 0.3s ease;
}

.lesson-content {
  margin-bottom: 2rem;
}

.vocabulary-section h3,
.grammar-section h3,
.dialogue-section h3,
.practice-section h3 {
  color: #2d3748;
  margin-bottom: 1.5rem;
  font-size: 1.25rem;
}

.vocabulary-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.vocabulary-card {
  background: #f7fafc;
  border-radius: 12px;
  padding: 1rem;
  text-align: center;
  transition: all 0.3s ease;
}

.vocabulary-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.word-image img {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: 8px;
  margin-bottom: 1rem;
}

.spanish-word {
  font-size: 1.25rem;
  color: #2d3748;
  margin-bottom: 0.5rem;
}

.thai-translation {
  color: #718096;
  margin-bottom: 1rem;
}

.audio-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 20px;
  background: #bee3f8;
  color: #2c5282;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.audio-btn:hover {
  background: #90cdf4;
}

.grammar-content {
  background: #f7fafc;
  border-radius: 12px;
  padding: 1.5rem;
}

.explanation {
  margin-bottom: 1.5rem;
  line-height: 1.6;
  color: #4a5568;
}

.examples h4 {
  color: #2d3748;
  margin-bottom: 1rem;
}

.example {
  background: white;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.example .spanish {
  font-weight: 600;
  color: #2d3748;
}

.example .thai {
  color: #718096;
}

.dialogue-container {
  max-width: 600px;
  margin: 0 auto;
}

.dialogue-message {
  display: flex;
  margin-bottom: 1rem;
  align-items: flex-start;
  gap: 1rem;
}

.dialogue-message.speaker-b {
  flex-direction: row-reverse;
}

.speaker-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #4299e1;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  flex-shrink: 0;
}

.speaker-b .speaker-avatar {
  background: #48bb78;
}

.message-content {
  background: #f7fafc;
  border-radius: 12px;
  padding: 1rem;
  max-width: 70%;
}

.speaker-b .message-content {
  background: #e6fffa;
}

.message-content .spanish {
  font-weight: 600;
  color: #2d3748;
  margin-bottom: 0.5rem;
}

.message-content .thai {
  color: #718096;
  margin-bottom: 0.5rem;
}

.lesson-navigation {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  flex-wrap: wrap;
}

.nav-btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  flex: 1;
  min-width: 120px;
}

.nav-btn.prev {
  background: #e2e8f0;
  color: #4a5568;
}

.nav-btn.next {
  background: #4299e1;
  color: white;
}

.nav-btn.complete {
  background: #48bb78;
  color: white;
}

.nav-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.nav-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

@media (max-width: 768px) {
  .lesson-view {
    padding: 1rem;
  }

  .lesson-header {
    flex-direction: column;
    align-items: stretch;
    text-align: center;
  }

  .vocabulary-grid {
    grid-template-columns: 1fr;
  }

  .dialogue-message {
    flex-direction: column !important;
    align-items: center;
  }

  .message-content {
    max-width: 100%;
  }

  .lesson-navigation {
    flex-direction: column;
  }
}
</style>