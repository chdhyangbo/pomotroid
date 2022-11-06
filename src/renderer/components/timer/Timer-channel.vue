<template>
  <div class="Dial-wrapper">
    <slot></slot>
    <p class="Dial-label">{{ currentRoundDisplay }}</p>
    <flipclock :options="options" class="flip-clock" ref="flipClock"></flipclock>
  </div>
</template>

<script>
import anime from 'animejs'
import { EventBus } from '@/utils/EventBus'
import { ipcRenderer } from 'electron'
import flipclock from '../flipclock/flipclock.vue'
console.log(flipclock);
export default {
  props: {
    currentTime: {
      type: Number,
      required: true,
    },
    minutes: {
      type: Number,
      required: true,
    },
    timerActive: {
      type: Boolean,
      required: true,
    },
  },

  data() {
    return {
      dial: null,
      options: {
        digit: 25*60,
        countdown: true,
        clockFace: 'DailyCounter',
        autoStart: false,
        callbacks: {
          stop: function () {
            // alert('The clock has stopped!')
            this.$emit('stop')
          },
        },
      },
    }
  },

  components:{
    flipclock,
  },

  computed: {
    // store getters
    currentRound() {
      return this.$store.getters.currentRound
    },

    timeLongBreak() {
      return this.$store.getters.timeLongBreak * 60 * 1000
    },

    timeShortBreak() {
      return this.$store.getters.timeShortBreak * 60 * 1000
    },

    timeWork() {
      return this.$store.getters.timeWork * 60 * 1000
    },

    currentRoundDisplay() {
      if (this.currentRound === 'work') {
        return 'Focus'
      } else if (this.currentRound === 'short-break') {
        return 'Short Break'
      } else if (this.currentRound === 'long-break') {
        return 'Long Break'
      }
    },

    dialClass() {
      if (this.currentRound === 'work') {
        return 'Dial-fill--work'
      } else if (this.currentRound === 'short-break') {
        return 'Dial-fill--shortBreak'
      } else if (this.currentRound === 'long-break') {
        return 'Dial-fill--longBreak'
      }
    },
  },

  methods: {
    /**
     * Set the time dial animation using a given duration in milliseconds.
     * If a dial animation already exists, removes it and recreates it.
     *
     * @param {number} duration - The current round duration in milliseconds.
     */
    dialAnimation(duration) {
      if (this.dial !== null) {
        this.dial = null
        anime.remove('.Dial-fill path')
        this.dialAnimation(duration)
      }
      this.dial = anime({
        targets: '.Dial-fill path',
        strokeDashoffset: [anime.setDashoffset, 0],
        easing: function () {
          return function (t) {
            return t.toFixed(3)
          }
        },
        duration: duration,
        direction: 'reverse',
        autoplay: false,
      })
      this.dial.seek(this.dial.duration)
    },

    /**
     * Reset timer animation on window focus.
     * Required due to RequestAnimationFrame not running in blurred windows.
     */
    handleFocus() {
      if (this.timerActive) {
        const duration = this.dial.duration
        const position = this.dial.duration - this.currentTime * 1000
        this.dial.pause()
        this.dialAnimation(duration)
        this.dial.seek(position)
        this.dial.play()
      }
    },
  },

  mounted() {
    // register listener for window-restore events
    ipcRenderer.on('win-restore', (event, arg) => {
      this.handleFocus()
    })
    // register listener for window-show events
    ipcRenderer.on('win-show', (event, arg) => {
      this.handleFocus()
    })

    // set timer to initial work time
    this.dialAnimation(this.timeWork)

    EventBus.$on('timer-started', () => {
      this.$refs.flipClock.start();
    })
    EventBus.$on('timer-paused', () => {
      this.$refs.flipClock.stop()
    })
    EventBus.$on('timer-resumed', () => {
      this.$refs.flipClock.start()
    })
    EventBus.$on('timer-reset', () => {
      this.dial.pause()
      this.dial.seek(this.dial.duration)
    })
    EventBus.$on('timer-init', () => {
      this.dial.pause()
      if (this.currentRound === 'work') {
        this.dialAnimation(this.timeWork)
      } else if (this.currentRound === 'short-break') {
        this.dialAnimation(this.timeShortBreak)
      } else if (this.currentRound === 'long-break') {
        this.dialAnimation(this.timeLongBreak)
      }
    })
  },
}
</script>

<style lang="scss" scoped>
.Dial-wrapper {
  display: flex;
  justify-content: center;
  margin-top: 35px;
  position: relative;
  width: 100%;
  height: 100%;
}

.Dial-label {
  letter-spacing: 0.1em;
  position: absolute;
  top: 66%;
  text-transform: uppercase;
}

.Dial-bg {
  stroke: var(--color-background-lightest);
}

.Dial-fill {
  position: absolute;
  transform-origin: center;
  -webkit-app-region: no-drag;
}

.Dial-fill--work {
  stroke: var(--color-focus-round);
}

.Dial-fill--shortBreak {
  stroke: var(--color-short-round);
}

.Dial-fill--longBreak {
  stroke: var(--color-long-round);
}

.flip-clock {
  // position: absolute;
  // top: 0;
  // left: 0;
  // width: 100px;
  // height: 100px;
  // background: black;
}
</style>
