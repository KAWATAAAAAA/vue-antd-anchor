<template>
    <div class="wrapper">
        Body
    </div>
</template>
<script>
export default {
    name:'Body',
    mounted(){
        console.log("Body 挂载完毕")
    }
}
</script>
<style scoped>
.wrapper{
    width: 100%;
    height: 100%;
    border: 1px solid red;
}
</style>