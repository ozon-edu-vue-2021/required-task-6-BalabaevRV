<template>
  <div id="app">
    <mega-table />
  </div>
</template>

<script>
import megaTable from '@/components/megaTable';

export default {
  name: 'App',
  components: {
    megaTable
  }
};
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
  }
</style>
