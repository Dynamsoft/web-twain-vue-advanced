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
    bWin: Object,
  },
  setup(props) {
    let handleStatusChange = inject("handleStatusChange", Function);
    let handleException = inject("handleException", Function);
    let handleBarcodeResults = inject("handleBarcodeResults", Function);
    let handleOutPutMessage = inject("handleOutPutMessage", Function);
    let handleNavigating = inject("handleNavigating", Function);

    const barcodeReady = ref(false);
    const readingBarcode = ref(false);
    const ocrReady = ref(false);
    const ocring = ref(false);

    const dbrResults = ref([]);

    const initBarcodeReader = (_features) => {
      props.dwt.Addon.BarcodeReader.getRuntimeSettings().then((settings) => {
        if(!barcodeReady.value) {
          barcodeReady.value = true;
          handleStatusChange(32);
        }
      },
       (ex) => {
         console.log(ex);
        // handleException({
        //   code: -6,
        //   message: "Initializing Barcode Reader failed: " + (ex.message || ex)
        // })
      });
    }

    const readBarcode = () => {
      props.Dynamsoft.Lib.showMask();
      readingBarcode.value = true;
      handleNavigating(false);
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
            handleOutPutMessage("Total barcode(s) found: " + dbrResults.length, "important");
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

    const initOCR = (_features) => {
      downloadOCRBasic(true, _features);
    }

    const downloadOCRBasic = (bDownloadDLL, _features) => {
      let strOCRPath = props.Dynamsoft.DWT.ResourcesPath + "/addon/OCRx64.zip";
      let strOCRLangPath = props.Dynamsoft.DWT.ResourcesPath + "/addon/OCRBasicLanguages/English.zip";
      if(bDownloadDLL) {
        if(props.dwt.Addon.OCR.IsModuleInstalled()) {
          downloadOCRBasic(false, _features);
        } else {
          props.dwt.Addon.OCR.Download(
            strOCRPath,
            () => {
              downloadOCRBasic(false);
            },
            (errorCode, errorString) => {
              handleException({
                code: errorCode,
                message: errorString
              })
            }
          )
        }
      } else {
        props.dwt.Addon.OCR.DownloadLangData(
          strOCRLangPath,
          () => {
            if(!ocrReady.value) {
              ocrReady.value = true;
              handleStatusChange(64);
            }
          },
          (errorCode, errorString) => {
            handleException({
              code: errorCode,
              message: errorString
            })
          }
        )
      }
    }

    const ocr = () => {
      props.dwt.Addon.OCR.SetLanguage('eng');
      props.dwt.Addon.OCR.SetOutputFormat(props.Dynamsoft.DWT.EnumDWT_OCROutputFormat.OCROF_TEXT);
      if(props.zones.length > 0) {
        ocrRect(props.zones);
      } else {
        props.dwt.Addon.OCR.Recognize(
          props.buffer.current,
          (imageId, result) => {
            if(result === null) {
              handleOutPutMessage("Nothing found!", "important");
              return;
            }
            handleOutPutMessage("", "", true);
            handleOutPutMessage("OCR result:", "important");
            handleOutPutMessage(props.Dynamsoft.Lib.base64.decode(result.Get()), "info", false, true);
          },
          (errorCode, errorString) => {
            handleException({
              code: errorCode,
              message: errorString
            })
          }
        )
      }
    }

    const ocrRect = (_zones) => {
      let doRectOCR = (_zone, _zoneId) => {
        props.dwt.Addon.OCR.RecognizeRect(
          props.buffer.current,
          _zone.x, _zone.y, _zone.x + _zone.width, _zone.y + _zone.height,
          (imageId, left, top, right, bottom, result) => {
            if(result === null) {
              handleOutPutMessage("Nothing found in the rect [" + left + ", " + top + ", " + right + ", " + bottom + "]", "important")
              return;
            }
            if(_zoneId === 0) {
              handleOutPutMessage("", "", true);
            }
            handleOutPutMessage("OCR result in the rect [" + left + ", " + top + ", " + right + ", " + bottom + "]", "important");
            handleOutPutMessage(props.Dynamsoft.Lib.base64.decode(result.Get()), "info", false, true);
            (++_zoneId < _zones.length) && doRectOCR(_zones[_zoneId], _zoneId);
          },
          (errorCode, errorString) => {
            handleException({
              code: errorCode,
              message: errorString
            });
            (++_zoneId < _zones.length) && doRectOCR(_zones[_zoneId], _zoneId);
          }
        )
      }
      doRectOCR(_zones[0], 0);
    }

    watch(
      () => props.dwt, () => {
      if(props.dwt) {
        if (props.features & 0b100000) {
            initBarcodeReader(props.features);
            }
        if (props.features & 0b1000000) {
            initOCR(props.features);
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
              style={ props.bWin.value ? "display:block" : "display:none" }
              class={ props.buffer.count === 0 ? "btn-readBarcode btn-disabled" : "btn-readBarcode" }
              disabled={ (props.buffer.count === 0) ? true : false }
              onClick={ readBarcode }>
              <span>Read Barcode</span>
            </button>
            <button 
              class={ props.buffer.count === 0 ?  "btn-ocr btn-disabled" : "btn-ocr" }
              disabled={ props.buffer.count === 0 ? true : false }
              onClick={ ocr }>
              <span>OCR(English)</span>
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
      .btn-ocr {
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
      .btn-ocr:hover {
        background-color: #61c2ec;
        cursor: pointer;
      }
    }
  }
  .btn-disabled {
    background: #aaa !important;
  }
</style>
