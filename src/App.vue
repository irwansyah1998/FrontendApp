<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <CrudPage msg="Welcome to Your CRUD App"/>
  </div>
</template>

<script>
import CrudPage from './components/CrudPage.vue'

export default {
  name: 'App',
  components: {
    CrudPage
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
