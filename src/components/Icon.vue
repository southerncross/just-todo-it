<template>
  <span @click="click" :class="'icon-' + name"></span>
</template>

<script>
import '../assets/icon.css'
export default {
  name: 'Icon',

  props: {
    name: String
  },

  methods: {
    click(e) {
      this.$emit('click')
    }
  }
}
</script>

<style>
</style>
