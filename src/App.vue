<template>
  <div id="app">
    <JanusVideo :key="i" v-for="i in 8" />
  </div>
</template>

<script>
import JanusVideo from './components/JanusVideo'

export default {
  components: { JanusVideo }
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
