<template>
  <div id="app">
    <Anchor
      :nav-list="navList"
      :scroll-view="false"
      :label-name="labelName"
      :lazy="true"
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
      navList: [],
      labelName: ["Header", "Body", "Footer"]
    };
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
