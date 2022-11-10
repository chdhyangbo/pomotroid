<template></template>

<script>
import { EventBus } from '@/utils/EventBus'
import { logger } from '@/utils/logger'

const notifier = require('node-notifier')
const path = require('path')

export default {
  name: 'Notification-win',

  computed: {
    // store getters
    timeLongBreak() {
      return this.$store.getters.timeLongBreak
    },
    timeShortBreak() {
      return this.$store.getters.timeShortBreak
    },
    timeWork() {
      return this.$store.getters.timeWork
    }
  },

  methods: {
    callNotification(opts) {
      notifier.notify(
        {
          appName: 'flipclock番茄钟',
          title: opts.title || '已完成此轮专注',
          message: opts.message,
          icon: opts.icon || path.join(__static, 'icon.png'),
          sound: false
        },
        (err, res) => {
          if (err) {
            logger.error(err)
          }
        }
      )
    },

    notifyLongBreak() {
      this.callNotification({
        message: `开始 ${this.timeLongBreak} 分钟的长时间休息.`,
        icon: path.join(__static, 'icon--blue.png')
      })
    },

    notifyShortBreak() {
      this.callNotification({
        message: `开始 ${this.timeShortBreak} 分钟的短时间休息.`,
        icon: path.join(__static, 'icon--green.png')
      })
    },

    notifyWork() {
      this.callNotification({
        title: '休息结束',
        message: `开始 ${this.timeWork} 分钟的专注.`
      })
    }
  },

  mounted() {
    EventBus.$on('ready-long-break', this.notifyLongBreak)
    EventBus.$on('ready-short-break', this.notifyShortBreak)
    EventBus.$on('ready-work', this.notifyWork)
  },

  beforeDestroy() {
    EventBus.$off('ready-long-break', this.notifyLongBreak)
    EventBus.$off('ready-short-break', this.notifyShortBreak)
    EventBus.$off('ready-work', this.notifyWork)
  }
}
</script>
