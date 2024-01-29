<script lang="jsx">
import { defineComponent, reactive, ref } from 'vue';
import Header from './components/Header.vue';
import Footer from './components/Footer.vue';
import DynamsoftSDK from './components/DynamsoftSDK.vue';

export default defineComponent({
  setup() {

    return () => (
      <>
        <Header></Header>
        <br style="height:17px"/>
        <DynamsoftSDK
          features = {[
            "scan",
            "camera",
            "load",
            "save",
            "upload",
            "uploader",
            "barcode",
          ]}
        ></DynamsoftSDK>
        <Footer></Footer>
      </>
    )
  }
});
</script>

<style lang="scss">
#app {
  height: 100%;
  text-align: center;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
