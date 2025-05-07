<script lang="jsx">
import { defineComponent, inject, ref, watch } from 'vue';

export default defineComponent({

  props: {
    Dynamsoft: Object,
    dwt: Object,
    buffer: Object,
    runtimeInfo: Object,
    zones: Array,
    features: Number,
    barcodeRects: Array,
  },
  setup(props, ctx) {
    let handleStatusChange = inject("handleStatusChange", Function);
    let handleException = inject("handleException", Function);
    let handleBarcodeResults = inject("handleBarcodeResults", Function);
    let handleOutPutMessage = inject("handleOutPutMessage", Function);
    let handleNavigating = inject("handleNavigating", Function);

    const barcodeReady = ref(false);
    const readingBarcode = ref(false);

    const dbrResults = ref([]);
    const parent_handleCloseVideo = inject('handleCloseVideo');

    const initBarcodeReader = (_features) => {

      props.dwt.Viewer.on('wheel', clearBarcodeRects);
      props.dwt.Viewer.on('scroll', clearBarcodeRects);

      props.dwt.Addon.BarcodeReader.getRuntimeSettings().then((settings) => {
        if(!barcodeReady.value) {
          barcodeReady.value = true;
          handleStatusChange(32);
        }
      },
       (exp) => {
         console.log(exp);
        // handleException({
        //   code: -6,
        //   message: "Initializing Barcode Reader failed: " + (ex.message || ex)
        // })
      });
    }

    const readBarcode = () => {

      let previewModeEl = document.querySelector('.previewMode');
      if (previewModeEl && previewModeEl.value !== "1") {
          handleOutPutMessage("Cannot read barcode in this view mode. Please switch to 1x1 view mode.", "error");
          return;
      }

      parent_handleCloseVideo();
      props.Dynamsoft.Lib.showMask();
      handleBarcodeResults("clear");
      readingBarcode.value = true;
      handleNavigating(false);
      
      if(props.dwt.Viewer) {
        props.dwt.Viewer.gotoPage(props.dwt.CurrentImageIndexInBuffer);
      }

      props.dwt.Addon.BarcodeReader.getRuntimeSettings().then((settings) => {
        if(props.dwt.GetImageBitDepth(props.buffer.current) === 1) {
          settings.scaleDownThreshold = 214748347;
        } else {
          settings.scaleDownThreshold = 2300;
        }
        settings.barcodeFormatIds = props.Dynamsoft.DBR.EnumBarcodeFormat.BF_ALL;
        settings.region.measuredByPercentage = 0;
        if(props.zones.length > 0) {
          let i = 0;
          let readBarcodeFromRect = () => {
            i++;
            settings.region.left = props.zones[i].x;
            settings.region.top = props.zones[i].y;
            settings.region.right = props.zones[i].x + props.zones[i].width;
            settings.region.bottom = props.zones[i].y + props.zones[i].height;
            if(i === props.zones.length - 1) {
              doReadBarcode(settings);
            } else {
              doReadBarcode(settings, readBarcodeFromRect);
            }
          }
          settings.region.left = props.zones[0].x;
          settings.region.top = props.zones[0].y;
          settings.region.right = props.zones[0].x + props.zones[0].width;
          settings.region.bottom = props.zones[0].y + props.zones[0].height;
          if(props.zones.length === 1) {
            doReadBarcode(settings);
          } else {
            doReadBarcode(settings, readBarcodeFromRect);
          }
        } else {
          settings.region.left = 0;
          settings.region.top = 0;
          settings.region.right = 0;
          settings.region.bottom = 0;
          doReadBarcode(settings);
        }
      })
    }

    const doReadBarcode = (settings, callback) => {
      let bHasCallback = props.Dynamsoft.Lib.isFunction(callback);
      props.dwt.Addon.BarcodeReader.updateRuntimeSettings(settings).then((settings) => {
        let userData = props.runtimeInfo.curImageTimeStamp;
        let outputResults = () => {
          if(dbrResults.value.length === 0) {
            handleOutPutMessage("--------------------------", "seperator");
            handleOutPutMessage("Nothing found on the image!", "important", false, false);
            doneReadingBarcode();
          } else {
            handleOutPutMessage("--------------------------", "seperator");
            handleOutPutMessage("Total barcode(s) found: " + dbrResults.value.length, "important");
            for(let i=0; i<dbrResults.value.length; i++) {
              let result = dbrResults.value[i];
              handleOutPutMessage("------------------", "seperator");
              handleOutPutMessage("Barcode " + (i + 1).toString());
              handleOutPutMessage("Type: " + result.BarcodeFormatString);
              handleOutPutMessage("Value: " + result.BarcodeText, "important");
            }
            if(props.runtimeInfo.curImageTimeStamp === userData) {
              handleBarcodeResults("clear");
              handleBarcodeResults(dbrResults);
            }
            doneReadingBarcode();
          }
        };
        let onDbrReadSuccess = (results) => {
          dbrResults.value = dbrResults.value.concat(results);
          bHasCallback ? callback() : outputResults();
        };
        let onDbrReadFail = (_code, _msg) => {
          handleException({
            code: _code,
            message: _msg
          });
          bHasCallback ? callback() :outputResults();
        };
        props.dwt.Addon.BarcodeReader.decode(props.buffer.current).then(onDbrReadSuccess, onDbrReadFail);
      });
    }

    const doneReadingBarcode = () => {
      handleNavigating(true);
      readingBarcode.value = false;
      dbrResults.value = [];
      props.Dynamsoft.Lib.hideMask();
    }

    const clearBarcodeRects = () => {
      handleBarcodeResults("clear");
    }

    watch(
      () => props.dwt, () => {
      if(props.dwt) {
        if (props.features & 0b100000) {
            initBarcodeReader(props.features);
        }
      }
    },{
      immediate: true
    })

    return () => (
      <>
        <div class="dwt-recognize-content">
          <div class="dwt-recognize-scan">
            <button 
              class={ props.buffer.count === 0 ? "btn-readBarcode btn-disabled" : "btn-readBarcode" }
              disabled={ (props.buffer.count === 0) ? true : false }
              onClick={ readBarcode }>
              <span>Read Barcode</span>
            </button>
          </div>

          <div class="dwt-recognize-scan" style={ props.barcodeRects.length > 0 ? "display:block; margin-top:10px" : "display:none" }>
            <button 
              class={ props.barcodeRects.length > 0 ?  "btn-clear-barcode" : "btn-clear-barcode btn-display-none" }
              onClick={ clearBarcodeRects }>
              <span>Clear Barcode Rects</span>
            </button>
          </div>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  .dwt-recognize-content {
    padding: 20px 20px;
    border-bottom: solid 1px #ccc;

    .dwt-recognize-scan {
      display: flex;
      justify-content: space-around;
      align-items: center;

      .btn-readBarcode {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 48%;
        height: 30px;
        border-radius: 5px;
        border:none;
        background-color: rgb(80, 168, 225);
        color: white;
        font-size: 14px;
      }
      .btn-readBarcode:hover {
        background-color: #61c2ec;
        cursor: pointer;
      }
    }
  }

  .btn-clear-barcode{
    width: 98%;
    height: 30px;
    border-radius: 5px;
    border:none;
    background-color: rgb(80, 168, 225);
    color: white;
    font-size: 14px;
  }
  .btn-disabled {
    background: #aaa !important;
  }
  .btn-display-none {
    display: none;
  }
</style>
