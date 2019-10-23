<template>
  <div :style="inputStyles" > </div>
</template>

<script>
export default {
  name: "Pixel",
  props: {
    color: String
  },
  computed: {
    inputStyles() {
      return {
        background: this.color
      }
    }
  },
  created(){
    //   console.log(this.color)
  }
};


</script>

<style scoped>
</style>