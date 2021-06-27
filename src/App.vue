<template>
  <div id="app">
   <Meal></Meal>
  </div>
</template>

<script>
import Meal from "@/components/meal";

export default {
  name: 'App',
  components: {
    Meal
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
