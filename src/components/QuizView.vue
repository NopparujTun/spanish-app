<template>
  <div class="quiz-view">
    <div class="quiz-header">
      <button @click="$emit('back')" class="back-btn">← กลับ</button>
      <h2>{{ quiz.title }}</h2>
      <div class="quiz-progress">
        {{ currentQuestion + 1 }}/{{ quiz.questions.length }}
      </div>
    </div>

    <div class="quiz-progress-bar">
      <div class="progress-fill" :style="{ width: questionProgress + '%' }"></div>
    </div>

    <div v-if="!showResults" class="question-container">
      <div class="question">
        <h3>{{ currentQuestionData.question }}</h3>
        <div v-if="currentQuestionData.audio" class="question-audio">
          <button @click="playQuestionAudio" class="audio-btn">🔊 ฟังคำถาม</button>
        </div>
      </div>

      <div class="answers">
        <div v-if="currentQuestionData.type === 'multiple-choice'" class="multiple-choice">
          <button 
            v-for="(option, index) in currentQuestionData.options" 
            :key="index"
            class="option-btn"
            :class="{ 
              selected: selectedAnswer === index,
              correct: showQuestionFeedback && index === currentQuestionData.correct,
              incorrect: showQuestionFeedback && selectedAnswer === index && index !== currentQuestionData.correct
            }"
            @click="selectAnswer(index)"
            :disabled="showQuestionFeedback"
          >
            {{ option }}
          </button>
        </div>

        <div v-if="currentQuestionData.type === 'fill-blank'" class="fill-blank">
          <div class="sentence">
            <span v-for="(part, index) in questionSentenceParts" :key="index">
              <input 
                v-if="part.isBlank" 
                v-model="fillAnswer"
                @keyup.enter="checkFillAnswer"
                class="blank-input"
                :class="{ 
                  correct: showQuestionFeedback && fillCorrect,
                  incorrect: showQuestionFeedback && !fillCorrect
                }"
                :disabled="showQuestionFeedback"
              />
              <span v-else>{{ part.text }}</span>
            </span>
          </div>
          <button 
            @click="checkFillAnswer" 
            class="check-btn"
            :disabled="!fillAnswer || showQuestionFeedback"
          >
            ตรวจคำตอบ
          </button>
        </div>

        <div v-if="currentQuestionData.type === 'match-image'" class="match-quiz">
          <div class="quiz-word">{{ currentQuestionData.word }}</div>
          <div class="image-options">
            <div 
              v-for="(image, index) in currentQuestionData.images" 
              :key="index"
              class="image-option"
              :class="{ 
                selected: selectedAnswer === index,
                correct: showQuestionFeedback && index === currentQuestionData.correct,
                incorrect: showQuestionFeedback && selectedAnswer === index && index !== currentQuestionData.correct
              }"
              @click="selectAnswer(index)"
            >
              <img :src="image.src" :alt="image.alt" />
            </div>
          </div>
        </div>

        <div v-if="currentQuestionData.type === 'listening'" class="listening-quiz">
          <div class="audio-player">
            <button @click="playListeningAudio" class="play-audio-btn">🔊 ฟังเสียง</button>
          </div>
          <div class="listening-options">
            <button 
              v-for="(option, index) in currentQuestionData.options" 
              :key="index"
              class="option-btn"
              :class="{ 
                selected: selectedAnswer === index,
                correct: showQuestionFeedback && index === currentQuestionData.correct,
                incorrect: showQuestionFeedback && selectedAnswer === index && index !== currentQuestionData.correct
              }"
              @click="selectAnswer(index)"
              :disabled="showQuestionFeedback"
            >
              {{ option }}
            </button>
          </div>
        </div>

        <div v-if="currentQuestionData.type === 'sentence-order'" class="sentence-order-quiz">
          <div class="words-container">
            <div 
              v-for="(word, index) in shuffledQuizWords" 
              :key="'quiz-word-' + index"
              class="word-chip"
              :class="{ selected: selectedQuizWords.includes(index) }"
              @click="selectQuizWord(index)"
            >
              {{ word }}
            </div>
          </div>
          <div class="sentence-builder">
            <div class="built-sentence">
              <span v-for="(word, index) in orderedQuizSentence" :key="'ordered-quiz-' + index" class="ordered-word">
                {{ word }}
              </span>
            </div>
            <button @click="clearQuizSentence" class="clear-btn">ล้าง</button>
          </div>
          <button 
            @click="checkSentenceOrder" 
            class="check-btn"
            :disabled="orderedQuizSentence.length !== currentQuestionData.words.length || showQuestionFeedback"
          >
            ตรวจคำตอบ
          </button>
        </div>
      </div>

      <div v-if="showQuestionFeedback" class="question-feedback">
        <div class="feedback-content">
          <p :class="isAnswerCorrect ? 'success' : 'error'">
            {{ isAnswerCorrect ? '✅ ถูกต้อง!' : '❌ ไม่ถูกต้อง' }}
          </p>
          <p v-if="currentQuestionData.explanation" class="explanation">
            {{ currentQuestionData.explanation }}
          </p>
        </div>
        <button @click="nextQuestion" class="next-question-btn">
          {{ currentQuestion < quiz.questions.length - 1 ? 'คำถามถัดไป' : 'ดูผลลัพธ์' }}
        </button>
      </div>
    </div>

    <div v-if="showResults" class="quiz-results">
      <div class="results-header">
        <h2>ผลลัพธ์แบบทดสอบ</h2>
        <div class="score-circle">
          <div class="score-text">
            <span class="score">{{ correctAnswers }}</span>
            <span class="total">/{{ quiz.questions.length }}</span>
          </div>
        </div>
      </div>

      <div class="score-details">
        <div class="score-percentage">
          คะแนน: {{ Math.round((correctAnswers / quiz.questions.length) * 100) }}%
        </div>
        <div class="score-message" :class="scoreClass">
          {{ scoreMessage }}
        </div>
      </div>

      <div class="results-actions">
        <button @click="retakeQuiz" class="retry-btn">ทำแบบทดสอบใหม่</button>
        <button @click="completeQuiz" class="complete-btn">เสร็จสิ้น</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'QuizView',
  props: {
    quiz: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      currentQuestion: 0,
      selectedAnswer: null,
      fillAnswer: '',
      showQuestionFeedback: false,
      showResults: false,
      correctAnswers: 0,
      userAnswers: [],
      
      // Sentence ordering
      shuffledQuizWords: [],
      selectedQuizWords: [],
      orderedQuizSentence: []
    }
  },
  computed: {
    currentQuestionData() {
      return this.quiz.questions[this.currentQuestion]
    },
    questionProgress() {
      return ((this.currentQuestion + 1) / this.quiz.questions.length) * 100
    },
    questionSentenceParts() {
      if (this.currentQuestionData.type !== 'fill-blank') return []
      
      const sentence = this.currentQuestionData.sentence
      const blankPos = this.currentQuestionData.blankPosition
      
      return [
        { text: sentence.substring(0, blankPos), isBlank: false },
        { isBlank: true },
        { text: sentence.substring(blankPos), isBlank: false }
      ]
    },
    fillCorrect() {
      return this.fillAnswer.toLowerCase().trim() === this.currentQuestionData.correct.toLowerCase()
    },
    sentenceOrderCorrect() {
      return this.orderedQuizSentence.join(' ') === this.currentQuestionData.correctOrder.join(' ')
    },
    isAnswerCorrect() {
      const question = this.currentQuestionData
      
      switch (question.type) {
        case 'multiple-choice':
        case 'match-image':
        case 'listening':
          return this.selectedAnswer === question.correct
        case 'fill-blank':
          return this.fillCorrect
        case 'sentence-order':
          return this.sentenceOrderCorrect
        default:
          return false
      }
    },
    scoreClass() {
      const percentage = (this.correctAnswers / this.quiz.questions.length) * 100
      if (percentage >= 80) return 'excellent'
      if (percentage >= 60) return 'good'
      return 'needs-improvement'
    },
    scoreMessage() {
      const percentage = (this.correctAnswers / this.quiz.questions.length) * 100
      if (percentage >= 80) return '🎉 ยอดเยี่ยม! คุณเข้าใจบทเรียนนี้ดีมาก'
      if (percentage >= 60) return '👍 ดีมาก! ลองทบทวนอีกนิดนะ'
      return '💪 ลองทบทวนบทเรียนและทำแบบทดสอบใหม่นะ'
    }
  },
  mounted() {
    this.initializeQuestion()
  },
  methods: {
    initializeQuestion() {
      if (this.currentQuestionData.type === 'sentence-order') {
        this.shuffledQuizWords = [...this.currentQuestionData.words].sort(() => Math.random() - 0.5)
        this.selectedQuizWords = []
        this.orderedQuizSentence = []
      }
      this.selectedAnswer = null
      this.fillAnswer = ''
      this.showQuestionFeedback = false
    },
    
    selectAnswer(index) {
      if (!this.showQuestionFeedback) {
        this.selectedAnswer = index
        this.checkAnswer()
      }
    },
    
    checkFillAnswer() {
      if (this.fillAnswer.trim()) {
        this.checkAnswer()
      }
    },
    
    checkSentenceOrder() {
      this.checkAnswer()
    },
    
    checkAnswer() {
      this.showQuestionFeedback = true
      
      if (this.isAnswerCorrect) {
        this.correctAnswers++
      }
      
      this.userAnswers.push({
        question: this.currentQuestion,
        answer: this.getFormattedAnswer(),
        correct: this.isAnswerCorrect
      })
    },
    
    getFormattedAnswer() {
      const question = this.currentQuestionData
      
      switch (question.type) {
        case 'multiple-choice':
        case 'listening':
          return question.options[this.selectedAnswer]
        case 'match-image':
          return `Image ${this.selectedAnswer + 1}`
        case 'fill-blank':
          return this.fillAnswer
        case 'sentence-order':
          return this.orderedQuizSentence.join(' ')
        default:
          return ''
      }
    },
    
    nextQuestion() {
      if (this.currentQuestion < this.quiz.questions.length - 1) {
        this.currentQuestion++
        this.initializeQuestion()
      } else {
        this.showResults = true
      }
    },
    
    selectQuizWord(index) {
      if (!this.selectedQuizWords.includes(index)) {
        this.selectedQuizWords.push(index)
        this.orderedQuizSentence.push(this.shuffledQuizWords[index])
      }
    },
    
    clearQuizSentence() {
      this.selectedQuizWords = []
      this.orderedQuizSentence = []
    },
    
    playQuestionAudio() {
      if (this.currentQuestionData.audio) {
        const audio = new Audio(this.currentQuestionData.audio)
        audio.play().catch(e => console.log('Audio play failed:', e))
      }
    },
    
    playListeningAudio() {
      if (this.currentQuestionData.listeningAudio) {
        const audio = new Audio(this.currentQuestionData.listeningAudio)
        audio.play().catch(e => console.log('Audio play failed:', e))
      }
    },
    
    retakeQuiz() {
      this.currentQuestion = 0
      this.correctAnswers = 0
      this.userAnswers = []
      this.showResults = false
      this.initializeQuestion()
    },
    
    completeQuiz() {
      this.$emit('complete', this.quiz.id)
    }
  }
}
</script>

<style scoped>
.quiz-view {
  background: white;
  border-radius: 15px;
  padding: 2rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.quiz-header {
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

.quiz-header h2 {
  color: #2d3748;
  font-size: 1.5rem;
}

.quiz-progress {
  background: #fbb6ce;
  color: #97266d;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-weight: 600;
}

.quiz-progress-bar {
  width: 100%;
  height: 8px;
  background: #e2e8f0;
  border-radius: 4px;
  margin-bottom: 2rem;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #ed64a6, #d53f8c);
  transition: width 0.3s ease;
}

.question-container {
  margin-bottom: 2rem;
}

.question {
  background: #f7fafc;
  border-radius: 12px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  text-align: center;
}

.question h3 {
  color: #2d3748;
  font-size: 1.25rem;
  margin-bottom: 1rem;
}

.question-audio {
  margin-top: 1rem;
}

.audio-btn, .play-audio-btn {
  background: #4299e1;
  color: white;
  border: none;
  border-radius: 25px;
  padding: 0.75rem 1.5rem;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.audio-btn:hover, .play-audio-btn:hover {
  background: #3182ce;
  transform: scale(1.05);
}

.multiple-choice, .listening-options {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.option-btn {
  background: white;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  padding: 1rem;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  text-align: left;
}

.option-btn:hover:not(:disabled) {
  border-color: #4299e1;
}

.option-btn.selected {
  border-color: #4299e1;
  background: #bee3f8;
}

.option-btn.correct {
  border-color: #48bb78;
  background: #f0fff4;
}

.option-btn.incorrect {
  border-color: #f56565;
  background: #fed7d7;
}

.option-btn:disabled {
  cursor: not-allowed;
}

.fill-blank {
  text-align: center;
}

.sentence {
  font-size: 1.2rem;
  line-height: 2;
  margin-bottom: 1rem;
}

.blank-input {
  border: 2px solid #cbd5e0;
  border-radius: 4px;
  padding: 0.5rem;
  font-size: 1.1rem;
  width: 150px;
  text-align: center;
  margin: 0 0.5rem;
}

.blank-input.correct {
  border-color: #48bb78;
  background: #f0fff4;
}

.blank-input.incorrect {
  border-color: #f56565;
  background: #fed7d7;
}

.check-btn {
  background: #4299e1;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 0.75rem 1.5rem;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.check-btn:hover:not(:disabled) {
  background: #3182ce;
}

.check-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.match-quiz {
  text-align: center;
}

.quiz-word {
  font-size: 1.5rem;
  font-weight: bold;
  color: #2d3748;
  margin-bottom: 1.5rem;
}

.image-options {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
}

.image-option {
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  padding: 0.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

.image-option:hover {
  border-color: #4299e1;
}

.image-option.selected {
  border-color: #4299e1;
  background: #bee3f8;
}

.image-option.correct {
  border-color: #48bb78;
  background: #f0fff4;
}

.image-option.incorrect {
  border-color: #f56565;
  background: #fed7d7;
}

.image-option img {
  width: 100%;
  height: 100px;
  object-fit: cover;
  border-radius: 4px;
}

.listening-quiz {
  text-align: center;
}

.audio-player {
  margin-bottom: 2rem;
}

.sentence-order-quiz {
  text-align: center;
}

.words-container {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1rem;
  justify-content: center;
}

.word-chip {
  background: white;
  border: 2px solid #e2e8f0;
  border-radius: 20px;
  padding: 0.5rem 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  user-select: none;
}

.word-chip:hover {
  border-color: #4299e1;
}

.word-chip.selected {
  background: #e2e8f0;
  opacity: 0.5;
  cursor: not-allowed;
}

.sentence-builder {
  background: white;
  border: 2px dashed #cbd5e0;
  border-radius: 8px;
  padding: 1rem;
  min-height: 60px;
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.built-sentence {
  flex: 1;
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.ordered-word {
  background: #fbb6ce;
  color: #97266d;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-weight: 600;
}

.clear-btn {
  background: #fed7d7;
  color: #c53030;
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  cursor: pointer;
  font-weight: 600;
}

.question-feedback {
  background: #f7fafc;
  border-radius: 12px;
  padding: 1.5rem;
  text-align: center;
  margin-top: 2rem;
}

.feedback-content {
  margin-bottom: 1.5rem;
}

.feedback-content p {
  font-weight: 600;
  font-size: 1.1rem;
  margin-bottom: 0.5rem;
}

.feedback-content .success {
  color: #22543d;
}

.feedback-content .error {
  color: #c53030;
}

.explanation {
  color: #4a5568;
  font-style: italic;
}

.next-question-btn {
  background: #4299e1;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 0.75rem 2rem;
  cursor: pointer;
  font-weight: 600;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.next-question-btn:hover {
  background: #3182ce;
  transform: translateY(-2px);
}

.quiz-results {
  text-align: center;
}

.results-header {
  margin-bottom: 2rem;
}

.results-header h2 {
  color: #2d3748;
  margin-bottom: 1.5rem;
}

.score-circle {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background: linear-gradient(135deg, #ed64a6, #d53f8c);
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  color: white;
}

.score-text {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.score {
  font-size: 2rem;
  font-weight: bold;
}

.total {
  font-size: 1rem;
}

.score-details {
  margin-bottom: 2rem;
}

.score-percentage {
  font-size: 1.5rem;
  font-weight: bold;
  color: #2d3748;
  margin-bottom: 1rem;
}

.score-message {
  font-size: 1.1rem;
  font-weight: 600;
  padding: 1rem;
  border-radius: 8px;
}

.score-message.excellent {
  background: #f0fff4;
  color: #22543d;
  border: 2px solid #48bb78;
}

.score-message.good {
  background: #bee3f8;
  color: #2c5282;
  border: 2px solid #4299e1;
}

.score-message.needs-improvement {
  background: #fed7d7;
  color: #c53030;
  border: 2px solid #f56565;
}

.results-actions {
  display: flex;
  gap: 1rem;
  justify-content: center;
  flex-wrap: wrap;
}

.retry-btn, .complete-btn {
  padding: 0.75rem 2rem;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 150px;
}

.retry-btn {
  background: #e2e8f0;
  color: #4a5568;
}

.retry-btn:hover {
  background: #cbd5e0;
}

.complete-btn {
  background: #48bb78;
  color: white;
}

.complete-btn:hover {
  background: #38a169;
  transform: translateY(-2px);
}

@media (max-width: 768px) {
  .quiz-view {
    padding: 1rem;
  }

  .quiz-header {
    flex-direction: column;
    align-items: stretch;
    text-align: center;
  }

  .multiple-choice, .listening-options {
    grid-template-columns: 1fr;
  }

  .image-options {
    grid-template-columns: repeat(2, 1fr);
  }

  .sentence-builder {
    flex-direction: column;
    gap: 1rem;
  }

  .results-actions {
    flex-direction: column;
  }
}
</style>