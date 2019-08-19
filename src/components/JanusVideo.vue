<template>
  <!-- Janus Video -->
  <div>
    <video
      :key="camera"
      v-for="(camera, i) in cameras"
      :id="`janusVideo${i}`"
      class="janus-video"
      playsinline
      autoplay
      muted
    />
  </div>
</template>

<script>
import Janus from '../janus'

export default {
  name: 'JanusVideo',
  props: {
    cameras: {
      type: Array,
      required: true
    },
    janus: {
      type: Object
    }
  },
  data () {
    return {
      streaming: []
    }
  },
  mounted () {
    this.initJanus()
  },
  methods: {
    // Init Janus
    initJanus () {
      for (let i = 0; i < this.cameras.length; i++) {
        this.janus.attach(
          {
            opaqueId: 'test-' + i,
            plugin: 'janus.plugin.streaming',
            success: (pluginHandle) => {
              this.streaming.push({ id: i, plugin: pluginHandle })
              let body = { 'request': 'watch', id: parseInt('1') }
              pluginHandle.send({ 'message': body })
            },
            error: (error) => {
              console.log(error)
            },
            onmessage: (msg, jsep) => {
              Janus.log('message', msg)
              if (jsep !== undefined && jsep !== null) {
                // Offer from the plugin, let's answer
                const foundStream = this.streaming.find(s => s.id === i)
                if (jsep.type === 'offer') {
                  foundStream.plugin.createAnswer(
                    {
                      jsep,
                      media: { audioSend: false, videoSend: false },
                      success: (jsep) => {
                        const body = { 'request': 'start' }
                        foundStream.plugin.send({ 'message': body, 'jsep': jsep })
                      },
                      error: (error) => {
                        Janus.error('WebRTC error:', error)
                      }
                    }
                  )
                } else {
                  foundStream.plugin.handleRemoteJsep({ jsep: jsep })
                }
              }
            },
            onremotestream: (stream) => {
              const element = document.getElementById(`janusVideo${i}`)
              Janus.attachMediaStream(element, stream)
            }
          })
      }
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
