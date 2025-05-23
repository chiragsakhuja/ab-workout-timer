<template>
  <div class="workout-app">
    <!-- Header -->
    <div class="header">
      <h1>Ab Workout Timer</h1>
      <div class="round-indicator">
        Round {{ currentRound }} / 3
      </div>
    </div>

    <!-- Main Timer Display -->
    <div class="timer-container">
      <div class="exercise-name">
        {{ currentExercise.name }}
      </div>
      
      <div class="timer-display" :class="{ 'warning': timeLeft <= 5 && timeLeft > 0, 'finished': timeLeft === 0 }">
        {{ formatTime(timeLeft) }}
      </div>
      
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: progressPercent + '%' }"></div>
      </div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <button v-if="!isRunning && !isFinished" @click="startWorkout" class="start-btn">
        Start Workout
      </button>
      
      <div v-if="isRunning" class="running-controls">
        <button @click="pauseWorkout" class="pause-btn">
          {{ isPaused ? 'Resume' : 'Pause' }}
        </button>
        <button @click="resetWorkout" class="reset-btn">
          Reset
        </button>
      </div>
      
      <button v-if="isFinished" @click="resetWorkout" class="restart-btn">
        Start Again
      </button>
    </div>

    <!-- Workout Complete -->
    <div v-if="isFinished" class="completion-message">
      <h2>🎉 Workout Complete! 🎉</h2>
      <p>Great job completing all 3 rounds!</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AbWorkoutTimer',
  data() {
    return {
      currentRound: 1,
      totalRounds: 3,
      currentExerciseIndex: 0,
      timeLeft: 0,
      isRunning: false,
      isPaused: false,
      isFinished: false,
      timer: null,
      wakeLock: null,
      
      // Audio contexts for different sounds
      audioContext: null,
      
      // Workout routine (repeated 3 times)
      exercises: [
        { name: "Side Plank (Right)", duration: 45, type: "exercise" },
        { name: "Rest", duration: 15, type: "break" },
        { name: "Side Plank (Left)", duration: 45, type: "exercise" },
        { name: "Rest", duration: 15, type: "break" },
        { name: "Plank - Straight Arms Tap Forward", duration: 45, type: "exercise" },
        { name: "Front Plank on Forearms", duration: 45, type: "exercise" },
        { name: "Rest", duration: 10, type: "break" },
        { name: "Russian Twists", duration: 20, type: "exercise" },
        { name: "Rest", duration: 15, type: "break" },
        { name: "Dead Bug", duration: 45, type: "exercise" },
        { name: "Rest", duration: 60, type: "break" }
      ]
    }
  },
  
  computed: {
    currentExercise() {
      return this.exercises[this.currentExerciseIndex] || { name: "Complete", duration: 0 };
    },
    
    progressPercent() {
      if (this.currentExercise.duration === 0) return 100;
      return ((this.currentExercise.duration - this.timeLeft) / this.currentExercise.duration) * 100;
    }
  },
  
  mounted() {
    this.initializeAudio();
  },
  
  beforeUnmount() {
    this.releaseWakeLock();
    if (this.timer) {
      clearInterval(this.timer);
    }
  },
  
  methods: {
    async initializeAudio() {
      try {
        this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
      } catch (error) {
        console.warn('Audio context not supported:', error);
      }
    },
    
    // Create different tones for different events
    playSound(frequency, duration = 200, type = 'start') {
      if (!this.audioContext) return;
      
      const oscillator = this.audioContext.createOscillator();
      const gainNode = this.audioContext.createGain();
      
      oscillator.connect(gainNode);
      gainNode.connect(this.audioContext.destination);
      
      // Different frequencies for different events
      oscillator.frequency.value = frequency;
      oscillator.type = 'sine';
      
      gainNode.gain.setValueAtTime(0, this.audioContext.currentTime);
      gainNode.gain.linearRampToValueAtTime(0.3, this.audioContext.currentTime + 0.01);
      gainNode.gain.exponentialRampToValueAtTime(0.01, this.audioContext.currentTime + duration / 1000);
      
      oscillator.start(this.audioContext.currentTime);
      oscillator.stop(this.audioContext.currentTime + duration / 1000);
    },
    
    playStartSound() {
      // High pitched double beep for start
      this.playSound(800, 150);
      setTimeout(() => this.playSound(800, 150), 200);
    },
    
    playWarningSound() {
      // Medium pitched single beep for 5 second warning
      this.playSound(600, 300);
    },
    
    playEndSound() {
      // Low pitched long beep for end
      this.playSound(400, 500);
    },
    
    async requestWakeLock() {
      try {
        if ('wakeLock' in navigator) {
          this.wakeLock = await navigator.wakeLock.request('screen');
          console.log('Screen wake lock acquired');
        }
      } catch (error) {
        console.warn('Wake lock not supported or failed:', error);
      }
    },
    
    async releaseWakeLock() {
      if (this.wakeLock) {
        await this.wakeLock.release();
        this.wakeLock = null;
        console.log('Screen wake lock released');
      }
    },
    
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = seconds % 60;
      return `${mins}:${secs.toString().padStart(2, '0')}`;
    },
    
    async startWorkout() {
      this.isRunning = true;
      this.isPaused = false;
      this.isFinished = false;
      this.currentRound = 1;
      this.currentExerciseIndex = 0;
      this.timeLeft = this.currentExercise.duration;
      
      await this.requestWakeLock();
      this.playStartSound();
      this.startTimer();
    },
    
    startTimer() {
      this.timer = setInterval(() => {
        if (!this.isPaused) {
          if (this.timeLeft > 0) {
            this.timeLeft--;
            
            // Play warning sound at 5 seconds
            if (this.timeLeft === 5) {
              this.playWarningSound();
            }
          } else {
            // Exercise completed
            this.playEndSound();
            this.nextExercise();
          }
        }
      }, 1000);
    },
    
    nextExercise() {
      this.currentExerciseIndex++;
      
      // Check if we've completed all exercises in current round
      if (this.currentExerciseIndex >= this.exercises.length) {
        this.currentRound++;
        
        // Check if we've completed all 3 rounds
        if (this.currentRound > this.totalRounds) {
          this.completeWorkout();
          return;
        }
        
        // Start next round
        this.currentExerciseIndex = 0;
      }
      
      // Set up next exercise
      this.timeLeft = this.currentExercise.duration;
      this.playStartSound();
    },
    
    completeWorkout() {
      this.isRunning = false;
      this.isFinished = true;
      clearInterval(this.timer);
      this.releaseWakeLock();
      
      // Play completion sound sequence
      setTimeout(() => this.playSound(800, 200), 0);
      setTimeout(() => this.playSound(1000, 200), 300);
      setTimeout(() => this.playSound(1200, 400), 600);
    },
    
    pauseWorkout() {
      this.isPaused = !this.isPaused;
    },
    
    resetWorkout() {
      this.isRunning = false;
      this.isPaused = false;
      this.isFinished = false;
      this.currentRound = 1;
      this.totalRounds = 3;
      this.currentExerciseIndex = 0;
      this.timeLeft = 0;
      
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
      
      this.releaseWakeLock();
    }
  }
}
</script>

<style scoped>
.workout-app {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  color: white;
  text-align: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.header {
  width: 100%;
  margin-bottom: 20px;
}

.header h1 {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.round-indicator {
  font-size: 1.2rem;
  background: rgba(255,255,255,0.2);
  padding: 8px 16px;
  border-radius: 20px;
  display: inline-block;
}

.timer-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  max-width: 90%;
}

.exercise-name {
  font-size: 1.8rem;
  font-weight: 600;
  margin-bottom: 30px;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.3);
  line-height: 1.2;
}

.timer-display {
  font-size: 6rem;
  font-weight: bold;
  margin-bottom: 30px;
  text-shadow: 3px 3px 6px rgba(0,0,0,0.4);
  transition: all 0.3s ease;
}

.timer-display.warning {
  color: #ff6b6b;
  animation: pulse 1s infinite;
}

.timer-display.finished {
  color: #51cf66;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: rgba(255,255,255,0.3);
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 20px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #51cf66, #40c057);
  transition: width 1s linear;
  border-radius: 4px;
}

.controls {
  width: 100%;
  margin-top: 20px;
}

.start-btn, .restart-btn {
  background: linear-gradient(135deg, #51cf66, #40c057);
  color: white;
  border: none;
  padding: 16px 32px;
  font-size: 1.4rem;
  font-weight: 600;
  border-radius: 50px;
  cursor: pointer;
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
  transition: all 0.3s ease;
  width: 100%;
  max-width: 300px;
}

.start-btn:hover, .restart-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0,0,0,0.3);
}

.running-controls {
  display: flex;
  gap: 15px;
  width: 100%;
  max-width: 300px;
  margin: 0 auto;
}

.pause-btn, .reset-btn {
  flex: 1;
  padding: 12px 20px;
  font-size: 1.1rem;
  font-weight: 600;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.pause-btn {
  background: linear-gradient(135deg, #ffd43b, #fab005);
  color: #333;
}

.reset-btn {
  background: linear-gradient(135deg, #ff6b6b, #fa5252);
  color: white;
}

.pause-btn:hover, .reset-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

.completion-message {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(0,0,0,0.9);
  padding: 30px;
  border-radius: 20px;
  text-align: center;
  animation: celebration 0.5s ease-out;
}

.completion-message h2 {
  font-size: 2.2rem;
  margin-bottom: 15px;
  color: #51cf66;
}

.completion-message p {
  font-size: 1.2rem;
  color: #ccc;
}

@keyframes celebration {
  0% { 
    opacity: 0;
    transform: translate(-50%, -50%) scale(0.8);
  }
  100% { 
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
}

/* Mobile responsive adjustments */
@media (max-width: 480px) {
  .header h1 {
    font-size: 2rem;
  }
  
  .exercise-name {
    font-size: 1.4rem;
  }
  
  .timer-display {
    font-size: 4.5rem;
  }
  
  .completion-message h2 {
    font-size: 1.8rem;
  }
}

/* Landscape orientation adjustments */
@media (orientation: landscape) and (max-height: 600px) {
  .workout-app {
    padding: 10px;
  }
  
  .header h1 {
    font-size: 1.8rem;
    margin-bottom: 5px;
  }
  
  .timer-display {
    font-size: 3.5rem;
    margin-bottom: 15px;
  }
  
  .exercise-name {
    font-size: 1.2rem;
    margin-bottom: 15px;
  }
}
</style> 