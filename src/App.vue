<template>
  <div id="app">
    <Top/>
  </div>
</template>

<script>
import Top from './components/Top';

export default {
  name: 'App',
  components: {
    Top
  }
};
</script>

<style>
html,
body {
  margin: 0;
  overflow: hidden;
  background-color: #000000;
  width: 100%;
  height: 100%;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  position: relative;
  width: 100%;
  height: 100%;
}
</style>
