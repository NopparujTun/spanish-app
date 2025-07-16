<template>
  <div class="practice-activity">
    <div v-if="activity.activityType === 'fill-blank'" class="fill-blank-activity">
      <p class="instruction">{{ activity.instruction }}</p>
      <div class="sentence">
        <span v-for="(part, index) in sentenceParts" :key="index">
          <input 
            v-if="part.isBlank" 
            v-model="userAnswers[part.blankIndex]"
            @input="checkAnswer"
            class="blank-input"
            :class="{ 
              correct: feedback[part.blankIndex] === 'correct',
              incorrect: feedback[part.blankIndex] === 'incorrect'
            }"
          />
          <span v-else>{{ part.text }}</span>
        </span>
      </div>
      <div v-if="showFeedback" class="feedback">
        <p :class="allCorrect ? 'success' : 'error'">
          {{ allCorrect ? '✅ ถูกต้อง!' : '❌ ลองใหม่อีกครั้ง' }}
        </p>
      </div>
    </div>

    <div v-if="activity.activityType === 'match-image'" class="match-image-activity">
      <p class="instruction">{{ activity.instruction }}</p>
      <div class="matching-container">
        <div class="words-column">
          <div 
            v-for="(word, index) in activity.words" 
            :key="'word-' + index"
            class="word-item"
            :class="{ selected: selectedWord === index }"
            @click="selectWord(index)"
          >
            {{ word.spanish }}
          </div>
        </div>
        <div class="images-column">
          <div 
            v-for="(image, index) in activity.images" 
            :key="'image-' + index"
            class="image-item"
            :class="{ 
              matched: matches[index] !== null,
              correct: matches[index] !== null && isMatchCorrect(index)
            }"
            @click="selectImage(index)"
          >
            <img :src="image.src" :alt="image.alt" />
          </div>
        </div>
      </div>
      <div v-if="showMatchFeedback" class="feedback">
        <p :class="allMatched ? 'success' : 'info'">
          {{ allMatched ? '✅ เยี่ยม! ถูกต้องทั้งหมด!' : `จับคู่แล้ว ${correctMatches}/${activity.words.length}` }}
        </p>
      </div>
    </div>

    <div v-if="activity.activityType === 'sentence-order'" class="sentence-order-activity">
      <p class="instruction">{{ activity.instruction }}</p>
      <div class="words-container">
        <div 
          v-for="(word, index) in shuffledWords" 
          :key="'shuffled-' + index"
          class="word-chip"
          :class="{ selected: selectedWords.includes(index) }"
          @click="selectWordForOrder(index)"
        >
          {{ word }}
        </div>
      </div>
      <div class="sentence-builder">
        <div class="built-sentence">
          <span v-for="(word, index) in orderedSentence" :key="'ordered-' + index" class="ordered-word">
            {{ word }}
          </span>
        </div>
        <button @click="clearSentence" class="clear-btn">ล้าง</button>
      </div>
      <div v-if="showOrderFeedback" class="feedback">
        <p :class="sentenceCorrect ? 'success' : 'error'">
          {{ sentenceCorrect ? '✅ ถูกต้อง!' : '❌ ลำดับคำไม่ถูกต้อง' }}
        </p>
      </div>
    </div>

    <div v-if="activity.activityType === 'audio-choice'" class="audio-choice-activity">
      <p class="instruction">{{ activity.instruction }}</p>
      <div class="audio-player">
        <button @click="playActivityAudio" class="play-audio-btn">🔊 ฟังเสียง</button>
      </div>
      <div class="choices">
        <button 
          v-for="(choice, index) in activity.choices" 
          :key="index"
          class="choice-btn"
          :class="{ 
            selected: selectedChoice === index,
            correct: showChoiceFeedback && index === activity.correctAnswer,
            incorrect: showChoiceFeedback && selectedChoice === index && index !== activity.correctAnswer
          }"
          @click="selectChoice(index)"
        >
          {{ choice }}
        </button>
      </div>
      <div v-if="showChoiceFeedback" class="feedback">
        <p :class="choiceCorrect ? 'success' : 'error'">
          {{ choiceCorrect ? '✅ ถูกต้อง!' : '❌ ไม่ถูกต้อง ลองใหม่' }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PracticeActivity',
  props: {
    activity: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      // Fill in the blank
      userAnswers: {},
      feedback: {},
      showFeedback: false,
      
      // Match image
      selectedWord: null,
      matches: {},
      showMatchFeedback: false,
      
      // Sentence ordering
      shuffledWords: [],
      selectedWords: [],
      orderedSentence: [],
      showOrderFeedback: false,
      
      // Audio choice
      selectedChoice: null,
      showChoiceFeedback: false
    }
  },
  computed: {
    sentenceParts() {
      if (this.activity.activityType !== 'fill-blank') return []
      
      const sentence = this.activity.sentence
      const blanks = this.activity.blanks
      let parts = []
      let lastIndex = 0
      let blankIndex = 0
      
      blanks.forEach(blank => {
        if (blank.position > lastIndex) {
          parts.push({ text: sentence.substring(lastIndex, blank.position), isBlank: false })
        }
        parts.push({ isBlank: true, blankIndex: blankIndex++ })
        lastIndex = blank.position
      })
      
      if (lastIndex < sentence.length) {
        parts.push({ text: sentence.substring(lastIndex), isBlank: false })
      }
      
      return parts
    },
    allCorrect() {
      if (this.activity.activityType !== 'fill-blank') return false
      return this.activity.blanks.every((blank, index) => 
        this.userAnswers[index] && 
        this.userAnswers[index].toLowerCase().trim() === blank.answer.toLowerCase()
      )
    },
    allMatched() {
      if (this.activity.activityType !== 'match-image') return false
      return Object.keys(this.matches).length === this.activity.words.length && 
             Object.values(this.matches).every(match => match !== null)
    },
    correctMatches() {
      return Object.keys(this.matches).filter(imageIndex => 
        this.isMatchCorrect(parseInt(imageIndex))
      ).length
    },
    sentenceCorrect() {
      if (this.activity.activityType !== 'sentence-order') return false
      return this.orderedSentence.join(' ') === this.activity.correctOrder.join(' ')
    },
    choiceCorrect() {
      if (this.activity.activityType !== 'audio-choice') return false
      return this.selectedChoice === this.activity.correctAnswer
    }
  },
  mounted() {
    this.initializeActivity()
  },
  methods: {
    initializeActivity() {
      if (this.activity.activityType === 'sentence-order') {
        this.shuffledWords = [...this.activity.words].sort(() => Math.random() - 0.5)
      }
    },
    
    // Fill in the blank methods
    checkAnswer() {
      this.showFeedback = true
      this.activity.blanks.forEach((blank, index) => {
        const userAnswer = this.userAnswers[index]
        if (userAnswer) {
          this.feedback[index] = userAnswer.toLowerCase().trim() === blank.answer.toLowerCase() 
            ? 'correct' : 'incorrect'
        }
      })
      
      if (this.allCorrect) {
        setTimeout(() => this.$emit('complete'), 1000)
      }
    },
    
    // Match image methods
    selectWord(index) {
      this.selectedWord = index
    },
    
    selectImage(imageIndex) {
      if (this.selectedWord !== null) {
        this.matches[imageIndex] = this.selectedWord
        this.selectedWord = null
        this.showMatchFeedback = true
        
        if (this.allMatched && this.correctMatches === this.activity.words.length) {
          setTimeout(() => this.$emit('complete'), 1000)
        }
      }
    },
    
    isMatchCorrect(imageIndex) {
      const wordIndex = this.matches[imageIndex]
      if (wordIndex === null || wordIndex === undefined) return false
      return this.activity.words[wordIndex].correctImage === imageIndex
    },
    
    // Sentence ordering methods
    selectWordForOrder(index) {
      if (!this.selectedWords.includes(index)) {
        this.selectedWords.push(index)
        this.orderedSentence.push(this.shuffledWords[index])
        this.checkSentenceOrder()
      }
    },
    
    clearSentence() {
      this.selectedWords = []
      this.orderedSentence = []
      this.showOrderFeedback = false
    },
    
    checkSentenceOrder() {
      if (this.orderedSentence.length === this.activity.words.length) {
        this.showOrderFeedback = true
        if (this.sentenceCorrect) {
          setTimeout(() => this.$emit('complete'), 1000)
        }
      }
    },
    
    // Audio choice methods
    selectChoice(index) {
      this.selectedChoice = index
      this.showChoiceFeedback = true
      
      if (this.choiceCorrect) {
        setTimeout(() => this.$emit('complete'), 1000)
      }
    },
    
    playActivityAudio() {
      if (this.activity.audio) {
        const audio = new Audio(this.activity.audio)
        audio.play().catch(e => console.log('Audio play failed:', e))
      }
    }
  }
}
</script>

<style scoped>
.practice-activity {
  background: #f7fafc;
  border-radius: 12px;
  padding: 1.5rem;
}

.instruction {
  font-size: 1.1rem;
  color: #2d3748;
  margin-bottom: 1.5rem;
  font-weight: 600;
  text-align: center;
}

/* Fill in the blank styles */
.sentence {
  font-size: 1.2rem;
  line-height: 2;
  text-align: center;
  margin-bottom: 1rem;
}

.blank-input {
  border: 2px solid #cbd5e0;
  border-radius: 4px;
  padding: 0.25rem 0.5rem;
  font-size: 1.1rem;
  width: 100px;
  text-align: center;
  margin: 0 0.25rem;
}

.blank-input.correct {
  border-color: #48bb78;
  background: #f0fff4;
}

.blank-input.incorrect {
  border-color: #f56565;
  background: #fed7d7;
}

/* Match image styles */
.matching-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  margin-bottom: 1rem;
}

.words-column, .images-column {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.word-item {
  background: white;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  padding: 1rem;
  text-align: center;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.word-item:hover {
  border-color: #4299e1;
}

.word-item.selected {
  border-color: #4299e1;
  background: #bee3f8;
}

.image-item {
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  padding: 0.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

.image-item:hover {
  border-color: #4299e1;
}

.image-item.matched {
  border-color: #ed8936;
}

.image-item.correct {
  border-color: #48bb78;
  background: #f0fff4;
}

.image-item img {
  width: 100%;
  height: 100px;
  object-fit: cover;
  border-radius: 4px;
}

/* Sentence ordering styles */
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
  background: #bee3f8;
  color: #2c5282;
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

/* Audio choice styles */
.audio-player {
  text-align: center;
  margin-bottom: 1.5rem;
}

.play-audio-btn {
  background: #4299e1;
  color: white;
  border: none;
  border-radius: 50px;
  padding: 1rem 2rem;
  font-size: 1.1rem;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.play-audio-btn:hover {
  background: #3182ce;
  transform: scale(1.05);
}

.choices {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-bottom: 1rem;
}

.choice-btn {
  background: white;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  padding: 1rem;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.choice-btn:hover {
  border-color: #4299e1;
}

.choice-btn.selected {
  border-color: #4299e1;
  background: #bee3f8;
}

.choice-btn.correct {
  border-color: #48bb78;
  background: #f0fff4;
}

.choice-btn.incorrect {
  border-color: #f56565;
  background: #fed7d7;
}

/* Feedback styles */
.feedback {
  text-align: center;
  margin-top: 1rem;
}

.feedback p {
  font-weight: 600;
  font-size: 1.1rem;
  padding: 0.75rem;
  border-radius: 8px;
}

.feedback .success {
  background: #f0fff4;
  color: #22543d;
  border: 2px solid #48bb78;
}

.feedback .error {
  background: #fed7d7;
  color: #c53030;
  border: 2px solid #f56565;
}

.feedback .info {
  background: #bee3f8;
  color: #2c5282;
  border: 2px solid #4299e1;
}

@media (max-width: 768px) {
  .matching-container {
    grid-template-columns: 1fr;
  }
  
  .choices {
    grid-template-columns: 1fr;
  }
  
  .sentence-builder {
    flex-direction: column;
    gap: 1rem;
  }
}
</style>