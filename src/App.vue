<template>
  <div id="app">
    <!-- Janus Videos List -->
    <JanusVideo :key="i" v-for="i in 8" :camera="i"/>
  </div>
</template>

<script>
export default {
  components: {
    JanusVideo: () => import('@/components/JanusVideo')
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
