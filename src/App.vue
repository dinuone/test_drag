<template>
  <Home/>
</template>

<script>

import Home from '@/components/Home'

export default {
  name: 'App',

  components: {
    Home
  },

  data: () => ({
    //
  }),
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;


  overflow: auto;
  position: relative;
}
</style>
