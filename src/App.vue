<template>
  <div id="app" class="bg-gray-200 h-screen">
    <Hash msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import Hash from './components/Hash.vue'

export default {
  name: 'app',
  components: {
    Hash
  }
}
</script>