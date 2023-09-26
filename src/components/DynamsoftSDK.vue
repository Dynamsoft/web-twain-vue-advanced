<script lang="jsx">
import { defineComponent, ref, reactive, onMounted, provide, watch } from 'vue';
import Dynamsoft from 'dwt';
import DWTInterface from './DWTInterface.vue';


export default defineComponent({
  props: {
    features: Array
  },
  setup(props) {

    let featureSet = { scan: 0b1, camera: 0b10, load: 0b100, save: 0b1000, upload: 0b10000, barcode: 0b100000, ocr: 0b1000000, uploader: 0b10000000 };
    let features = 0b11111111;
    let initialStatus = 0;
    let containerId = "dwtcontrolContainer";
    let DWObject = null;
    let width = 590;
    let height = 522;

    // change features and initialStatus
    if(props.features) {
      features = 0;
      props.features.map((value) => {
        if(featureSet[value]) {
           features += featureSet[value];
        }
        return features;
      });
      initialStatus = features - (features & 0b11100011);
    }

    // status
    let startTime  = ref((new Date()).getTime());
    let unSupportedEnv = ref(false);
    let dwt = null;//ref();
    let status = ref(initialStatus);
    let selected = ref([]);
    let zones = ref([]);
    let buffer = reactive({
      updated: false,
      count: 0,
      current: -1
    });
    let runtimeInfo = reactive({
      curImageTimeStamp: 0,
      showAbleWidth: 0,
      showAbleHeight: 0,
      ImageWidth: 0,
      ImageHeight: 0
    });

    // create DWObject, register event
    const loadDWT = (UseService) => {
      Dynamsoft.DWT.Containers = [{ ContainerId: 'dwtcontrolContainer', Width: 0, Height: 0 }];
      Dynamsoft.DWT.ResourcesPath = "/dwt-resources";
		  Dynamsoft.DWT.ProductKey = 't0107KwEAAFqHOzuiG0pkbu8UHkUGeOSoaezN9f0x+c7plsWGSCA28VNFZFgj71h2GachA/emFbGgAT3emZ8dQmmIu1Wg2A5UAHk4fwDOvVYEC34dWi6egGFy6ICtAAn9z6svMviWPNJ4ABlIPoA=';
      Dynamsoft.DWT.AutoLoad = true;
      Dynamsoft.DWT.RegisterEvent('OnWebTwainReady',() => webTwain_OnReady())
    }

    const webTwain_OnReady = () => {
      DWObject = Dynamsoft.DWT.GetWebTwain('dwtcontrolContainer');
      if(DWObject) {
        DWObject.Viewer.width = width;
        DWObject.Viewer.height = height;
        DWObject.Viewer.show();
        handleStatusChange(1);
       // dwt.value = DWObject;
        dwt = DWObject;
        DWObject.RegisterEvent("OnBitmapChanged", (changedIndex, changeType) => handleBufferChange(changedIndex, changeType));
        DWObject.RegisterEvent("OnPostTransfer", () => handleBufferChange());
        DWObject.RegisterEvent("OnPostLoad", () => handleBufferChange());
        DWObject.RegisterEvent("OnPostAllTransfers", () => DWObject.CloseSource());
        DWObject.RegisterEvent("OnBufferChanged", (e) => {
          if(e.action === 'shift' && e.currentId !==  -1){
            handleBufferChange()
          }
        });
        DWObject.Viewer.on('pageAreaSelected', (nImageIndex, rect) => {
          if (rect.length > 0) {
            let currentRect = rect[rect.length - 1];
            let newZones = [...zones.value];
            if(rect.length === 1)
              newZones = [];
            newZones.push({ x: currentRect.x, y: currentRect.y, width: currentRect.width, height: currentRect.height });
            zones.value = newZones;
          }
        });
        DWObject.Viewer.on('pageAreaUnselected', () => zones.value = []);
        DWObject.Viewer.on("click", () => handleBufferChange());
        if(Dynamsoft.Lib.env.bWin)
          DWObject.MouseShape = false;
        handleBufferChange();
      }
    }

    // handle viewer size change
    const handleViewerSizeChange = (viewSize) => {
      width = viewSize.width;
      height = viewSize.height;
    }

    // handle status change
    const handleStatusChange = (value) => {
      status.value = status.value + value;
    }

    //handle buffer change
    const handleBufferChange = (changedIndex, changeType) => {
      let _updated = false;
      // Modified
      if (changeType === 4) {
          _updated = true;
      }
      let selection = DWObject.SelectedImagesIndices;
      selected.value = selection;
      buffer.updated = _updated;
      buffer.current = DWObject.CurrentImageIndexInBuffer;
      buffer.count = DWObject.HowManyImagesInBuffer;
    }

    onMounted(() => {
      Dynamsoft.Ready(function() {
        if(!Dynamsoft.Lib.env.bWin || !Dynamsoft.Lib.product.bChromeEdition) {
          //unSupportedEnv.value = true;
          featureSet = { scan: 0b1, load: 0b100, save: 0b1000, upload: 0b10000, uploader: 0b10000000 };
          features = 0b10011101;
          initialStatus = 0;
        } 
        if(DWObject === null) 
          loadDWT(true);
        
      });
    })
    
    watch(buffer,() => {
      if (buffer.count > 0) {
          runtimeInfo.curImageTimeStamp = (new Date()).getTime();
          runtimeInfo.showAbleWidth = (DWObject.HowManyImagesInBuffer > 1 ? width - 17 : width)-4;
          runtimeInfo.showAbleHeight = height-4;
          runtimeInfo.ImageWidth = DWObject.GetImageWidth(buffer.current);
          runtimeInfo.ImageHeight = DWObject.GetImageHeight(buffer.current);
      }
    })

    provide("handleViewerSizeChange",handleViewerSizeChange);
    provide("handleStatusChange",handleStatusChange);

    return () => (
      <>
        <div class="dwt-content">
          <DWTInterface
            Dynamsoft = { Dynamsoft }
            features = { features }
            containerId = { containerId }
            startTime = { startTime.value }
            //dwt = { dwt.value }
            dwt = { dwt }
            status = { status.value }
            selected = { selected.value }
            zones = { zones.value }
            buffer = { buffer }
            runtimeInfo = { runtimeInfo }
            handleBufferChange = { () => handleBufferChange() }
          ></DWTInterface>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  .dwt-content {
    display: flex;
    width: 100%;
    height: 100%;
  }
</style>
