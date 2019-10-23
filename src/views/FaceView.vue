<template>
  <div class="Face">
    <FaceComp msg="Decentralised Face"/>
  </div>
</template>

<script>
// @ is an alias to /src
import FaceComp from '@/components/Face.vue'

export default {
  name: 'Face',
  components: {
    FaceComp
  },
  created() {
    console.log("faceview created")
  }
}
</script>
