<template>
  <div id="app">
<!--    <img alt="Vue logo" src="./assets/logo.png">-->
    <Load :msg="msg"></Load>
  </div>
</template>

<script>
import Load from './components/Load.vue'

export default {
  name: 'App',
  components: {
    Load
  },
  data: () => {
    return {
      msg: "Loading a file to redmine",
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
