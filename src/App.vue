<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
import Game from './components/Game'
import Ranking from './components/Ranking'
import VueRouter from 'vue-router'
import Vuex from 'vuex'

const router = new VueRouter({
  routes: [
    { path: '/', component: Game },
    { path: '/ranking', component: Ranking, props: true },
  ]
})

export default {
  name: 'App',
  data() {
    return {
      score: false,
    }
  },
  router,
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
