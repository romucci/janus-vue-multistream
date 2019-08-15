<template>
  <video
    :ref="`janusVideo${camera.id}`"
    class="bg-dark"
    playsinline
    autoplay
    @playing="$emit('started')"
  />
</template>

<script>
import Janus from '../../utils/janus'
let streaming = []

export default {
  name: 'JanusVideo',
  props: {
    camera: {
      type: Object,
      required: false,
      default: () => {}
    },
    quality: {
      type: Number,
      required: true
    },
    'cid': {
      type: Number,
      required: true
    }
  },
  data () {
    return {
      janus: {},
      isWaiting: false
    }
  },
  watch: {
    quality (newValue) {
      if (newValue) {
        let cameraID = null
        switch (newValue) {
          case 1:
            cameraID = 21
            break
          case 2:
            cameraID = 31
            break
          case 3:
            cameraID = 32
            break
          default:
            cameraID = 99
        }
        this.switchStream(cameraID)
      }
    }
  },
  mounted () {
    this.$emit('isWaiting')
    this.initJanus()
  },
  beforeDestroy () {
    console.log('destroyed janus stuff')
    this.janus.destroy()
  },
  methods: {
    restartStream () {
      this.$emit('isWaiting')
      const body = { 'request': 'start' }
      streaming.forEach(s => {
        s.send({ 'message': body })
      })
    },
    pauseStream () {
      const body = { 'request': 'pause' }
      streaming.forEach(s => {
        s.send({ 'message': body })
      })
    },
    switchStream (id) {
      const body = { 'request': 'switch', 'id': id }
      streaming.forEach(s => {
        s.send({ 'message': body })
      })
    },
    initJanus () {
      let server = 'https://janus.conf.meetecho.com/janus'

      if (!this.camera.id) {
        // server = `https://ifu-${this.ifuId}.proxy.agricamera.co.uk/janus`
      }

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
                    success: (pluginHandle) => {
                      streaming.push(pluginHandle)
                      console.log(streaming, 'jjjjjjjjjjjjjjjjj')
                      let body = { 'request': 'watch', id: parseInt('1') }
                      streaming[streaming.length - 1].send({ 'message': body })
                      this.$emit('isWaiting')
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
                      console.log('Attached', stream, this.camera.id)
                      this.$nextTick(() => Janus.attachMediaStream(this.$refs[`janusVideo${this.camera.id}`], stream))
                      // Janus.attachMediaStream(this.$refs.janusVideo2, stream)
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

<style scoped>

</style>
