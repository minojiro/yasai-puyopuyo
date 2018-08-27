<template>
  <div id="app">
    <Game></Game>
  </div>
</template>

<script>
import Game from './components/Game'

export default {
  name: 'App',
  data() {
  },
  components: {
    Game,
  },
}
</script>

<style>
body {
  background: #D5D6D9;
  padding: 10px;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
