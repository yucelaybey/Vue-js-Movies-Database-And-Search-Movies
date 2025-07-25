<template>
  <div class="min-vh-100 bg-light">
    <Home />
  </div>
</template>

<script>
import Home from './views/Home.vue'
import axios from 'axios'
export default {
  name: 'App',
  components: { Home }
}
</script> 