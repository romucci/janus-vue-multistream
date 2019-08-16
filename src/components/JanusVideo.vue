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
    ></video>
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
  created () {
    setTimeout(() => this.initJanus(), 4000)
  },
  methods: {
    // Init Janus
    initJanus () {
      const vm = this
      for (let i = 0; i < this.cameras.length; i++) {
        this.janus.attach(
          {
            opaqueId: 'test-' + i,
            plugin: 'janus.plugin.streaming',
            success: function (pluginHandle) {
              console.log('catched plugin handle ', pluginHandle)
              if (pluginHandle) {
                console.log('plugin handle found', pluginHandle)
                vm.streaming.push({ id: i, plugin: pluginHandle })
                let body = { 'request': 'watch', id: parseInt('1') }
                console.log('index in plugin handle success', i)
                vm.streaming[i].plugin.send({ 'message': body })
                console.log(vm.streaming)
              }
            },
            error: function (error) { console.log(error) },
            onmessage: function (msg, jsep) {
              Janus.log('message', msg)
              console.log('jsep', jsep)
              if (jsep !== undefined && jsep !== null) {
                // Offer from the plugin, let's answer
                console.log('index in on message', i)
                console.log('array of streams', vm.streaming)
                const foundStream = vm.streaming.find(s => s.id === i)
                console.log('foundStream', foundStream)
                if (jsep.type === 'offer') {
                  foundStream.plugin.createAnswer(
                    {
                      jsep,
                      media: { audioSend: false, videoSend: false },
                      success: function (jsep) {
                        const body = { 'request': 'start' }
                        console.log('start', jsep)
                        console.log('index in on message success', i)

                        foundStream.plugin.send({ 'message': body, 'jsep': jsep })
                      },
                      error: function (error) {
                        Janus.error('WebRTC error:', error)
                      }
                    }
                  )
                } else {
                  foundStream.plugin.handleRemoteJsep({ jsep: jsep })
                }
              } else {
                let body = { 'request': 'watch', id: parseInt('1') }
                console.log('index when we are retrying', i)
                vm.streaming[i].plugin.send({ 'message': body })
              }
            },
            onremotestream: function (stream) {
              console.log('on remote stream being called')
              const element = document.getElementById(`janusVideo${i}`)
              console.log('this is the element', element)
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
