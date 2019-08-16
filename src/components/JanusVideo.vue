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
    ></video>
  </div>
</template>

<script>
import Janus from '../janus'
let streaming = []

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
  mounted () {
    this.initJanus()
  },
  created () {
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
            success: function (pluginHandle) {
              streaming.push(pluginHandle)
              let body = { 'request': 'watch', id: parseInt('1') }
              console.log('index in plugin handle success', i)
              streaming[i].send({ 'message': body })
            },
            error: function (error) { console.log(error) },
            onmessage: function (msg, jsep) {
              Janus.log('message', msg)
              if (jsep !== undefined && jsep !== null) {
                // Offer from the plugin, let's answer
                console.log('index in on message', i)
                streaming[i].createAnswer(
                  {
                    jsep,
                    media: { audioSend: false, videoSend: false },
                    success: function (jsep) {
                      const body = { 'request': 'start' }
                      console.log('start', jsep)
                      console.log('index in on message success', i)
                      streaming[i].send({ 'message': body, 'jsep': jsep })
                    },
                    error: function (error) {
                      Janus.error('WebRTC error:', error)
                    }
                  })
              }
            },
            onremotestream: function (stream) {
              console.log('on remote stream being called')
              const element = document.getElementById(`janusVideo${i}`)
              console.log(element)
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
