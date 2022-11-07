<template>
  <div class="Dial-wrapper">
    <slot></slot>
    <!-- <p class="Dial-label">{{ currentRoundDisplay }}</p> -->
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
        digit: 25*60,// 秒数
        countdown: true,// 倒计时
        clockFace: 'MinuteCounter',// 时间模式
        autoStart: false, // 自动开始
        callbacks: {
          stop: function () {
            // alert('The clock has stopped!')
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
      return this.$store.getters.timeLongBreak * 60
    },

    timeShortBreak() {
      return this.$store.getters.timeShortBreak * 60
    },

    timeWork() {
      return this.$store.getters.timeWork * 60
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
     * Reset timer animation on window focus.
     * Required due to RequestAnimationFrame not running in blurred windows.
     */
    handleFocus() {
      if (this.timerActive) {
       
      }
    },
    reset() {
      if (this.currentRound === 'work') {
        this.options.digit = this.timeWork;
      } else if (this.currentRound === 'short-break') {
        this.options.digit = this.timeShortBreak;
      } else if (this.currentRound === 'long-break') {
        this.options.digit = this.timeLongBreak;
      }
      this.$refs.flipClock.reset()
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
    this.options.digit = this.timeWork;
    // this.dialAnimation(this.timeWork)

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
      this.$refs.flipClock.stop()
      this.reset()
      this.$refs.flipClock.start()
    })
    EventBus.$on('timer-init', () => {
      this.reset();
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
  height: 75%;
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
