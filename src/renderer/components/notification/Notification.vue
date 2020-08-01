<template> </template>

<script>
import { EventBus } from '@/utils/EventBus'
const path = require('path')
const cp = require('child_process')

export default {
  name: 'Notification',

  data() {
    return {
      notification: null
    }
  },

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
      if (opts.body.endsWith('break.')) {
        const child = cp.spawn('i3-nagbar', ['-m', opts.body], {
          detached: true,
          stdio: ['ignore', 'ignore', 'ignore']
        })
        child.unref()
      }
    },

    notifyLongBreak() {
      this.callNotification({
        title: 'Focus Round Complete',
        body: `Begin a ${this.timeLongBreak} minute long break.`,
        icon: path.join('static', 'icon--blue.png')
      })
    },

    notifyShortBreak() {
      this.callNotification({
        title: 'Focus Round Complete',
        body: `Begin a ${this.timeShortBreak} minute short break.`,
        icon: path.join('static', 'icon--green.png')
      })
    },

    notifyWork() {
      this.callNotification({
        title: 'Break Finished',
        body: `Begin focusing for ${this.timeWork} minutes.`
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
