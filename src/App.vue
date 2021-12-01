<template>
  <div id="app">
    <video-bg></video-bg>
    <div class="container">
      <div class="row">
        <div class="col-md-12 content">
          <main-header></main-header>
          <h5 class="my-3">The Best JavaScript Grid in the World</h5>
          <div class="row">
            <div class="col-md-12 col-sm-12">
              <grid></grid>
            </div>
          </div>
          <small class="footer">Version : Vue2</small>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Grid from "./components/Grid.vue";
import MainHeader from "./components/Header.vue";
import VideoBg from "./components/VideoBg.vue";
export default {
  components: {
    Grid,
    MainHeader,
    VideoBg
  },
  data() {
    return {
      msg: "Welcome to Your Vue.js App"
    };
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
