<script lang="jsx">
import { defineComponent, ref, reactive, toRaw, watch, onMounted } from 'vue';
import CustomScan from './components/CustomScan.vue';
import UseWebcams from './components/UseWebcams.vue';
import LoadFiles from './components/LoadFiles.vue';
import SaveDocuments from './components/SaveDocuments.vue';
import Recognize from './components/Recognize.vue';

export default defineComponent({
  props: {
    Dynamsoft: Object,
    dwt: Object,
    buffer: Object,
    runtimeInfo: Object,
    features: Number,
    zones: Array,
    selected: Array,
    barcodeRects: Array,
  },
  setup(props) {

    const arrowDown = require('@/assets/arrow-down.png');
    const bWin = ref(false);
    let TabShow = reactive({
      showCustomScan:true,
      showUseWebcams:false,
      showLoadFiles:false,
      showSaveDocuments:true,
      showRecognize:false
    })

    const changeTab = (i) => {
      switch (i) {
        case 0 : 
          TabShow.showCustomScan ? TabShow.showCustomScan = false : (TabShow.showCustomScan = true , TabShow.showUseWebcams = false , TabShow.showLoadFiles = false);
          break;
        case 1 :
          TabShow.showUseWebcams ? TabShow.showUseWebcams = false : (TabShow.showUseWebcams = true , TabShow.showCustomScan = false, TabShow.showLoadFiles = false);
          break;
        case 2 :
          TabShow.showLoadFiles ? TabShow.showLoadFiles = false : (TabShow.showLoadFiles = true , TabShow.showUseWebcams = false, TabShow.showCustomScan = false);
          break;
        case 3 :
          TabShow.showSaveDocuments ? TabShow.showSaveDocuments = false : (TabShow.showSaveDocuments = true , TabShow.showRecognize = false);
          break;
        case 4 :
          TabShow.showRecognize ? TabShow.showRecognize = false : (TabShow.showRecognize = true , TabShow.showSaveDocuments = false);
          break;
      }
    }

    watch(() => props.dwt, () => {
      if(props.dwt) {
        bWin.value = Dynamsoft.Lib.env.bWin;
      }
    })

    return () => (
      <>
        <div class="dwt-setting">
          <div style="border: solid 1px #ccc; border-bottom: none;">
            <div class="custom-scan">
              <div class="custom-scan-title" onClick={ () => changeTab(0) }>
                <label for="customScan">
                  <span>Custom Scan</span>
                  <img src={ arrowDown } alt="arrow-down" />
                </label>
              </div>
              <div  style={ TabShow.showCustomScan ? "display:block" : "display:none" }>
                <CustomScan
                  Dynamsoft = { props.Dynamsoft }
                  dwt = { props.dwt }
                  features = { props.features }
                ></CustomScan>
              </div>
            </div>

            <div class="dwt-use-webcams" style={ bWin.value ? "display:block" : "display:none" }>
              <div class="dwt-use-webcams-title" onClick={ () => changeTab(1) }>
                <label for="useWebcams">
                  <span>Use Webcams</span>
                  <img src={ arrowDown } alt="arrow-down" />
                </label>
              </div>
              <div  style={ TabShow.showUseWebcams ? "display:block" : "display:none" }>
                <UseWebcams
                  Dynamsoft = { props.Dynamsoft }
                  dwt={ props.dwt }
                  features={ props.features }
                  showUseWebcams = { TabShow.showUseWebcams }
                ></UseWebcams>
              </div>
            </div>

            <div class="dwt-load">
              <div class="dwt-load-title" onClick={() => changeTab(2)}>
                <label for="loadFiles">
                  <span>Load Images or PDFs</span>
                  <img src={ arrowDown } alt="arrow-down" />
                </label>
              </div>
              <div  style={ TabShow.showLoadFiles ? "display:block" : "display:none" }>
                <LoadFiles
                  dwt = { props.dwt }
                ></LoadFiles>
              </div>
            </div>

            <div class="dwt-save">
              <div class="dwt-save-title" onClick={() => changeTab(3)}>
                <label for="saveFiles">
                  <span>Save Documents</span>
                  <img src={ arrowDown } alt="arrow-down" />
                </label>
              </div>
              <div style={ TabShow.showSaveDocuments ? "display:block" : "display:none" }>
                <SaveDocuments
                  Dynamsoft = { props.Dynamsoft }
                  dwt = { props.dwt }
                  selected = { props.selected }
                  buffer = { props.buffer }
                  features = { props.features }
                ></SaveDocuments>
              </div>
            </div>

            <div class="dwt-recognize" style={ bWin.value ? "display:block" : "display:none" }>
              <div class="dwt-recognize-title" onClick={ () => changeTab(4) }>
                <label for="dwt-recognize">
                  <span>Recognize</span>
                  <img src={ arrowDown } alt="arrow-down" />
                </label>
              </div>
              <div  style={ TabShow.showRecognize ? "display:block" : "display:none" }>
                <Recognize
                  Dynamsoft = { props.Dynamsoft }
                  dwt = { props.dwt }
                  buffer = { props.buffer }
                  zones = { props.zones }
                  runtimeInfo = { props.runtimeInfo }
                  features = {props.features}
                  bWin = { bWin }
                  barcodeRects = {props.barcodeRects}
                ></Recognize>
              </div>
            </div>
          </div>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  @mixin setting-title-content {
    width: 100%;
    height: 40px;
    border-bottom: 1px solid #ccc;
    background-color: rgb(233, 233, 233);
    user-select: none;

    label {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      height: 40px;

      span {
        margin-left: 15px;
      }

      img {
        width: 20px;
        height: 20px;
        margin-right: 15px;
      }
    }

    label:hover {
      cursor: pointer;
    }
  }

  @mixin flex-layout {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .dwt-setting {
    width: 320px;
    height: 580px;
    font-size: 15px;
  }

  .custom-scan {

    .custom-scan-title {
      @include setting-title-content;
    }

    #customScan {
      display: none;
    }

    #customScan:checked+.if-showCustomScan {
      display: none !important;
    }

    #customScan:checked~.if-showUseWebcams {
      display: block !important;
    }
  }

  .dwt-use-webcams {

    .dwt-use-webcams-title {
      @include setting-title-content;
    }

    #useWebcams {
      display: none;
    }

    #useWebcams:checked+.if-showUseWebcams {
      display: none;
    }
  }

  .dwt-load {

    .dwt-load-title {
      @include setting-title-content;
    }

    #loadFiles {
      display: none;
    }

    #loadFiles:checked+.if-showLoadFiles {
      display: none;
    }
  }

  .dwt-save {

    .dwt-save-title {
      @include setting-title-content;
    }

    #saveFiles {
      display: none;
    }

    #saveFiles:checked+.if-showSaveDocuments {
      display: none;
    }
  }

  .dwt-recognize {

    .dwt-recognize-title {
      @include setting-title-content;
    }

    #dwt-recognize {
      display: none;
    }

    #dwt-recognize:checked+.if-showRecognize {
      display: none;
    }
  }
</style>
