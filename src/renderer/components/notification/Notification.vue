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
    },
    currentRound() {
      return this.$store.getters.currentRound
    }
  },

  methods: {
    callNotification(opts) {
      if (!opts.body.endsWith('break.')) {
        return
      }

      const child = cp.spawn(
        'dzen2',
        ['-p', '-bg', '#8bbe2c', '-m', '-e', 'button1=exit'],
        {
          detached: true,
          stdio: ['ignore', 'ignore', 'ignore']
        }
      )

      child.unref()
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
    },

    clearDzenNotification() {
      if (this.currentRound !== 'work') {
        return
      }

      const processList = cp
        .spawnSync('ps', ['-eo', 'pid,command'], { shell: true })
        .stdout.toString()
        .split('\n')
        .slice(1)

      for (const line of processList) {
        const parts = line.trim().split(' ')
        const pid = parts[0]

        if (!pid) {
          continue
        }

        for (const cmd of parts.slice(1)) {
          if (cmd.slice(0, 5) === 'dzen2') {
            cp.execFile('kill', [pid])
            break
          }
        }
      }
    }
  },

  mounted() {
    EventBus.$on('ready-long-break', this.notifyLongBreak)
    EventBus.$on('ready-short-break', this.notifyShortBreak)
    EventBus.$on('ready-work', this.notifyWork)
    EventBus.$on('timer-started', this.clearDzenNotification)
  },

  beforeDestroy() {
    EventBus.$off('ready-long-break', this.notifyLongBreak)
    EventBus.$off('ready-short-break', this.notifyShortBreak)
    EventBus.$off('ready-work', this.notifyWork)
    EventBus.$off('timer-started', this.clearDzenNotification)
  }
}
</script>
