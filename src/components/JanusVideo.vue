<template>
  <!-- Janus Video -->
  <video
    ref="janusVideo"
    class="janus-video"
    playsinline
    autoplay
  />
</template>

<script>
import Janus from '../janus'
let streaming = null

export default {
  name: 'JanusVideo',
  data () {
    return {
      janus: {}
    }
  },
  mounted () {
    this.initJanus()
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
                    plugin: 'janus.plugin.streaming',
                    opaqueId: 'test-' + Janus.randomString(12),
                    success: (pluginHandle) => {
                      streaming = pluginHandle
                      let body = { 'request': 'watch', id: parseInt('1') }
                      streaming.send({ 'message': body })
                    },
                    error: (error) => { console.log(error) },
                    onmessage: (msg, jsep) => {
                      Janus.log('message', msg)
                      if (jsep !== undefined && jsep !== null) {
                        // Offer from the plugin, let's answer
                        streaming.createAnswer(
                          {
                            jsep,
                            media: { audioSend: false, videoSend: false },
                            success: (jsep) => {
                              const body = { 'request': 'start' }
                              streaming.send({ 'message': body, 'jsep': jsep })
                            },
                            error: (error) => {
                              Janus.error('WebRTC error:', error)
                            }
                          })
                      }
                    },
                    onremotestream: (stream) => {
                      console.log('got stream:' + stream)
                      Janus.attachMediaStream(this.$refs.janusVideo, stream)
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
