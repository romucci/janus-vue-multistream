<template>
  <!-- Janus Video -->
  <video
    :ref="`janusVideo${camera}`"
    class="janus-video"
    playsinline
    autoplay
  />
</template>

<script>
import Janus from '../janus'
let streaming = []

export default {
  name: 'JanusVideo',
  props: {
    camera: {
      type: Number,
      required: true
    }
  },
  data () {
    return {
      janus: {}
    }
  },
  mounted () {
    this.$nextTick(() => this.initJanus())
  },
  beforeDestroy () {
    this.janus.destroy()
  },
  methods: {
    // Restart Stream
    restartStream () {
      const body = { 'request': 'restart' }
      streaming.send({ 'message': body })
    },
    // Pause Stream
    pauseStream () {
      const body = { 'request': 'pause' }
      streaming.send({ 'message': body })
    },
    // Switch Stream
    switchStream (id) {
      const body = { 'request': 'switch', 'id': id }
      streaming.send({ 'message': body })
    },
    // Init Janus
    initJanus () {
      let server = 'https://janus.conf.meetecho.com/janus'
      Janus.init({
        debug: 'all',
        callback: () => {
          this.janus = new Janus(
            {
              server,
              success: () => {
                this.janus.attach(
                  {
                    opaqueId: 'test-' + Janus.randomString(12),
                    plugin: 'janus.plugin.streaming',
                    success: (pluginHandle) => {
                      streaming.push(pluginHandle)
                      console.log(streaming, 'jjjjjjjjjjjjjjjjj')
                      let body = { 'request': 'watch', id: parseInt('1') }
                      streaming[streaming.length - 1].send({ 'message': body })
                    },
                    error: (error) => { console.log(error) },
                    onmessage: (msg, jsep) => {
                      Janus.log('message', msg)
                      if (jsep !== undefined && jsep !== null) {
                        // Offer from the plugin, let's answer
                        for (let i = 0; i < streaming.length; i++) {
                          streaming[i].createAnswer(
                            {
                              jsep,
                              media: { audioSend: false, videoSend: false },
                              success: (jsep) => {
                                const body = { 'request': 'start' }
                                console.log('start', jsep)
                                streaming[i].send({ 'message': body, 'jsep': jsep })
                              },
                              error: (error) => {
                                Janus.error('WebRTC error:', error)
                              }
                            })
                        }
                      }
                    },
                    onremotestream: (stream) => {
                      console.log('Attached', stream)
                      this.$nextTick(() => Janus.attachMediaStream(this.$refs[`janusVideo${this.camera}`], stream))
                    }
                  })
              },
              error: (cause) => {
                console.log(cause)
              },
              destroyed: () => {
                console.log('Destroyed from inside janus handle')
              }
            })
        }
      })
    }
  }
}
</script>

<style scoped lang="sass">
.janus-video
  width: 350px
  height: 200px
  background: black
  border: 5px solid rgba(35, 177, 104, 0.83)
  margin: 30px 10px
</style>
