<script setup lang="ts">
import Header from "./components/Header.vue"
import Canvas from "./components/Canvas.vue"
import Footer from "./components/Footer.vue"
</script>

<template >
      <div id="app">
      <Header />
      <Canvas />
      <Footer />
      </div>
</template>

<style>
*{
  margin:0;
}
body{
  overflow: hidden;
}
</style>
