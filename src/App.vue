<template>
  <div id="app">
    <i-map v-model="location" autoLocation></i-map>
    <router-view/>
  </div>
</template>

<script>
import IMap from '@/components/map'
export default {
  name: 'App',
  data () {
    return {
      location: {}
    }
  },
  components: {
    IMap
  },
  watch: {
    location (location) {
      console.log(location)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
