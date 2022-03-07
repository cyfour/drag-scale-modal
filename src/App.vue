<template>
  <div id="app" style="background: white;width: 100vw;height: 100vh;">
    <drag-scale-modal v-model="visible">
      <div slot="content" style="height: 100%;overflow: auto">
        <div style="height: 500px;">
          test
        </div>
      </div>
    </drag-scale-modal>
  </div>
</template>

<script>



import DragScaleModal from "./components/drag-scale-modal";
export default {
  name: 'App',
  components: {
    DragScaleModal,
  },
  data(){
    return{
      visible:true,

    }
  },
  created() {

  }
}
</script>

<style>
  .test{
    background: white;
    width: 500px;
    height: 300px;
  }
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

  padding: 0;
  margin: 0;
}
div{
  padding: 0;
  margin: 0;
}
  body{
    padding: 0;
    margin: 0;
  }
</style>
