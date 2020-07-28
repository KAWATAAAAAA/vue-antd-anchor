<template>
  <div class="wrapper">
    Footer
  </div>
</template>
<script>
export default {
  name: "Footer",
  mounted() {
    console.log("Footer 挂载完毕");
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
