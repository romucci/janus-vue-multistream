<template>
  <div id="app">
    <!-- Janus Videos List -->
    <div v-if="!loading">
      <JanusVideo :janus="janus" :cameras="['1', '2', '3' ]" />
    </div>
  </div>
</template>

<script>
import Janus from './janus'

export default {
  components: {
    JanusVideo: () => import('@/components/JanusVideo')
  },
  data () {
    return {
      janus: null,
      loading: false
    }
  },
  mounted () {
    this.initJanus()
  },
  beforeDestroy () {
    this.janus.destroy()
  },
  methods: {
    initJanus () {
      this.loading = true
      let server = 'https://janus.conf.meetecho.com/janus'
      Janus.init({
        debug: 'all',
        callback: () => {
          this.janus = new Janus(
            {
              server,
              success: () => {
                this.loading = false
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

<style lang="sass">
#app
  height: 90vh
  display: flex
  flex-wrap: wrap
  align-items: center
  justify-content: center
</style>
