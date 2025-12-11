<script lang="jsx">
import { defineComponent, ref, reactive, watch, provide } from 'vue';
import DWTViewer from './components/DWTViewer.vue';
import DWTController from './components/DWTController.vue';
import DWTMessage from './components/DWTMessage.vue';


export default defineComponent({
  props: {
    Dynamsoft: Object,
    features: Number,
    containerId: String,
    startTime: Number,
    dwt: Object,
    status: Number,
    buffer: Object,
    selected: Array,
    zones: Array,
    runtimeInfo: Object,
    handleBufferChange: Function,
  },
  setup(props) {

    const statusChangeText = (_status, _statusChange) => {
      let text = "Initializing...";
      if(_statusChange) {
        text = [];
        (_statusChange & 1) && text.push("Core module ");
        (_statusChange & 2) && text.push("Webcam module ");
        (_statusChange & 32) && text.push("Barcode Reader module ");
        (_statusChange & 64) && text.push("File Uploader module ");
        if(text.length > 1) {
          text = text.join(" & ");
        }
        text += "ready...";
      }
      if(_status === props.features) {
        if(_statusChange) {
          text = "_ALLDONE_" + text;
        }else {
          text = "Ready...";
        }
      }
      return text;
    }

    // status
    const bNoScroll = ref(false);
    const bNoNavigating = ref(false);
    let barcodeRects = ref([]);
    let messages = reactive([{
      time: (new Date()).getTime(),
      text: statusChangeText(props.status),
      type: "info"
    }]);

    const handleBarcodeResults = (results) => {
      if(results === "clear") {
        barcodeRects.value = [];
      } else {
        let _oldBR = [...barcodeRects.value];
        if(results.value.length > 0) {
          let zoom;
          if(props.runtimeInfo.showAbleWidth >= props.runtimeInfo.ImageWidth && props.runtimeInfo.showAbleHeight >= props.runtimeInfo.ImageHeight) {
            zoom = 1;
          } else if (props.runtimeInfo.showAbleWidth / props.runtimeInfo.showAbleHeight >= props.runtimeInfo.ImageWidth / props.runtimeInfo.ImageHeight) {
              zoom = props.runtimeInfo.showAbleHeight / props.runtimeInfo.ImageHeight;
          } else {
            zoom = props.runtimeInfo.showAbleWidth / props.runtimeInfo.ImageWidth;
          }
          for (let i = 0; i < results.value.length; ++i) {
            let result = results.value[i];
            let loc = result.localizationResult;
            let left = Math.min(loc.x1, loc.x2, loc.x3, loc.x4);
            let top = Math.min(loc.y1, loc.y2, loc.y3, loc.y4);
            let right = Math.max(loc.x1, loc.x2, loc.x3, loc.x4);
            let bottom = Math.max(loc.y1, loc.y2, loc.y3, loc.y4);
            let leftBase = 1+props.runtimeInfo.showAbleWidth / 2 - props.runtimeInfo.ImageWidth / 2 * zoom ;
            let topBase = 2 + props.runtimeInfo.showAbleHeight / 2 - props.runtimeInfo.ImageHeight / 2 * zoom;
            let width = (right - left) * zoom;
            let height = (bottom - top) * zoom;
            left = leftBase + left * zoom;
            top = topBase + top * zoom;
            _oldBR.push({ x: left, y: top, w: width, h: height });
          }

          barcodeRects.value = _oldBR;
        }
      }
    }

    const handleOutPutMessage = (message, type, bReset, _bNoScroll) => {
      let _noScroll = false, _type = "info";
      if(type) {
        _type = type;
      }
      if(_type === "httpResponse") {
        let msgWindow = window.open("", "Response from server", "height=500,width=750,top=0,left=0,toolbar=no,menubar=no,scrollbars=no, resizable=no,location=no, status=no");
        msgWindow.document.writeln(message);
      } else {
        if(_bNoScroll) {
          _noScroll = true;
        }
        if(bReset) {
          messages.length = 0
          messages.push({
            time: (new Date()).getTime(),
            text: statusChangeText(props.status),
            type: "info"
          })
          bNoScroll.value = false;
         } else {
          messages.push({
            time: (new Date()).getTime(),
            text: message,
              type: _type
          })
          bNoScroll.value = _noScroll;
        }
      }
    }

    const handleException = (ex) => {
      handleOutPutMessage(ex.message, "error");
    }

    const handleNavigating = (bAllow) => {
      bNoNavigating.value = !bAllow;
    }

    const handleEvent = (event) => {
      switch(event) {
        case "doubleClick": handleOutPutMessage("", "", true); break;
        case "delete": handleOutPutMessage("", "", true); break;
        default: break;
      }
    }

    watch(() => props.status,
      (newStatus, prevStatus) => {
        let _statusChange = newStatus - prevStatus;
        let _text = statusChangeText(newStatus, _statusChange);
        if (_text.indexOf("_ALLDONE_") !== -1) {
          handleOutPutMessage(_text.substr(9));
          handleOutPutMessage("All ready... <initialization took " + ((new Date()).getTime() - props.startTime) + " milliseconds>", "important");
        } else {
          handleOutPutMessage(_text);
        }
      }
    )

    watch(props.buffer,
      (newBuffer, oldBuffer) => {
        if(barcodeRects.value.length > 0){
          handleBarcodeResults("clear")
        }
        if(newBuffer.updated) {
          props.buffer.updated && props.handleBufferChange();
        }
      }
    )

    provide("handleBarcodeResults", handleBarcodeResults);
    provide("handleOutPutMessage", handleOutPutMessage);
    provide("handleException", handleException);
    provide("handleNavigating", handleNavigating);

    return () => (
      <>
        <div class="dwt-content-inner">
          <div class="left-side-bar">
            <DWTViewer
              blocks = { 0b11 }
              dwt = { props.dwt }
              buffer = { props.buffer }
              zones = { props.zones }
              containerId = { props.containerId }
              runtimeInfo = { props.runtimeInfo }
              bNoNavigating = { bNoNavigating.value }
              barcodeRects = { barcodeRects.value }
              handleBufferChange = { () => props.handleBufferChange() }
            ></DWTViewer>
            <DWTMessage
              handleEvent = { (event) => handleEvent(event) }
              messages = { messages }
              bNoScroll = { bNoScroll.value }
            ></DWTMessage>
          </div>
          <div class="right-side-bar">
            <DWTController
              Dynamsoft = { props.Dynamsoft }
              features = { props.features }
              dwt = { props.dwt }
              buffer = { props.buffer }
              selected = { props.selected }
              zones = { props.zones }
              runtimeInfo = { props.runtimeInfo }
              barcodeRects = { barcodeRects.value }
            ></DWTController>
          </div>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  .dwt-content-inner {
    display: flex;
    justify-content: center;
    width: 100%;
    height: 100%;

    .left-side-bar {
      width: 650px;
    }

    .right-side-bar {
      width: 30%;
      max-width: 315px;
      margin-left: 30px;
    }
  }
</style>
