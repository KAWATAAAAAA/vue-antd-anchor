<template>
  <div id="app">
    <Anchor
      :scroll-view="false"
      :label-name="labelName"
      :lazy="true"
      @onClick="handleViewNodeChange"
      @onAllDone="handleLazyLoadAllDone"
      @onScrolling="handleScrolling"
      @onScrolled="handleScrolled"
    >
      <Header></Header>
      <Body></Body>
      <Footer></Footer>
    </Anchor>
  </div>
</template>

<script>
import Anchor from "./components/anchor";

import Header from "./components/Header";
import Body from "./components/Body";
import Footer from "./components/Footer";

import prettierLog from "./utils/log"
console.log(prettierLog)
export default {
  name: "App",
  components: {
    Anchor,
    Header,
    Body,
    Footer
  },
  data() {
    return {
      labelName: ["Header", "Body", "Footer"]
    };
  },
  methods:{
    handleViewNodeChange(node,index){
      console.log(node,index)
    },
    handleLazyLoadAllDone(){
      prettierLog.default("all done")
    },
    handleScrolling(){
      prettierLog.danger("scrolling...")
    },
    handleScrolled(){
      prettierLog.success("scrolled")
    },
  }
};
</script>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
