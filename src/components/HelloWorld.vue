<template>
  <div class="hello">
    <div class="bg-red-500 p-20 text-white">
        <div text-3xl>
          SHA-1 algorithm
        </div> 
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="username">
            text input
          </label>
          <input class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" id="username" type="text" placeholder="Username">
        </div>
        <div class="block">{{ sha }}</div>
      </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      sha: 'your sha-1'
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->