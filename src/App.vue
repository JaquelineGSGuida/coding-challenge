<script setup>
import { RouterView } from 'vue-router'
</script>

<template>
  <router-view />
  <div class="site-author text-center text-muted py-3">
    Made by Jaqueline Guida <br />
    Recipes by <a href="https://spoonacular.com/food-api">Spoonacular</a> and Homepage Photo by
    <a
      href="https://unsplash.com/@tamanna_rumee?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
      >Tamanna Rumee</a
    >
    on
    <a
      href="https://unsplash.com/photos/sliced-tomato-and-sliced-lemon-on-white-ceramic-plate-cvq_8XHp2rM?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
      >Unsplash</a
    >
  </div>
</template>

<style scoped></style>
