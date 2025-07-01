<template>
  <div class="min-h-screen bg-gray-100">
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