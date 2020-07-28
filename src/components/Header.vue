<template>
  <div class="wrapper">
    Header
  </div>
</template>
<script>
export default {
  name: "Header",
  mounted() {
    console.log("Header 挂载完毕");
  }
};
</script>
<style scoped>
.wrapper {
  width: 100%;
  height: 100%;
  border: 1px solid red;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
