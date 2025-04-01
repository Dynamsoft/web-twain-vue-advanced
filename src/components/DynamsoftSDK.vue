<script lang="jsx">
import { defineComponent, ref, reactive, onMounted, provide, watch } from 'vue';
import Dynamsoft from 'dwt';
import DWTInterface from './DWTInterface.vue';


export default defineComponent({
  props: {
    features: Array
  },
  setup(props) {

    let featureSet = { scan: 0b1, camera: 0b10, load: 0b100, save: 0b1000, upload: 0b10000, barcode: 0b100000, uploader: 0b1000000 };
    let features = 0b1111111;
    let initialStatus = 0;
    let containerId = "dwtcontrolContainer";
    let DWTObject = null;
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
      initialStatus = features - (features & 0b1100011);
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

    // create DWTObject, register event
    const loadDWT = (UseService) => {
      Dynamsoft.OnLicenseError = function (message, errorCode) {
        if(errorCode == -2808)
          message = '<div style="padding:0">Sorry. Your product key has expired. You can purchase a full license at the <a target="_blank" href="https://www.dynamsoft.com/store/dynamic-web-twain/#DynamicWebTWAIN">online store</a>.</div><div style="padding:0">Or, you can try requesting a new product key at <a target="_blank" href="https://www.dynamsoft.com/customer/license/trialLicense?product=dwt&utm_source=in-product">this page</a>.</div><div style="padding:0">If you need any help, please <a target="_blank" href="https://www.dynamsoft.com/company/contact/">contact us</a>.</div>';
          Dynamsoft.DWT.ShowMessage(message, {
          width: 680,
          headerStyle: 2
        });
      };
      Dynamsoft.DWT.Containers = [{ ContainerId: 'dwtcontrolContainer', Width: 0, Height: 0 }];
      Dynamsoft.DWT.ResourcesPath = "/dwt-resources";
      Dynamsoft.DWT.ProductKey = 'DLS2eyJvcmdhbml6YXRpb25JRCI6IjIwMDAwMSJ9';
      Dynamsoft.DWT.AutoLoad = true;
      Dynamsoft.DWT.RegisterEvent('OnWebTwainReady',() => webTwain_OnReady())
    }

    const webTwain_OnReady = () => {
      DWTObject = Dynamsoft.DWT.GetWebTwain('dwtcontrolContainer');
      if(DWTObject) {
        DWTObject.Viewer.width = width;
        DWTObject.Viewer.height = height;
        DWTObject.Viewer.show();
        handleStatusChange(1);
       // dwt.value = DWTObject;
        dwt = DWTObject;
        DWTObject.RegisterEvent("OnBitmapChanged", (changedIndex, changeType) => handleBufferChange(changedIndex, changeType));
        DWTObject.RegisterEvent("OnPostTransfer", () => handleBufferChange());
        DWTObject.RegisterEvent("OnPostLoad", () => handleBufferChange());
        DWTObject.RegisterEvent("OnPostAllTransfers", () => DWTObject.CloseSource());
        DWTObject.RegisterEvent("OnBufferChanged", (e) => {
          if(e.action === 'shift' && e.currentId !==  -1){
            handleBufferChange()
          }
        });
        DWTObject.Viewer.on('pageAreaSelected', (nImageIndex, rect) => {
          if (rect.length > 0) {
            let currentRect = rect[rect.length - 1];
            let newZones = [...zones.value];
            if(rect.length === 1)
              newZones = [];
            newZones.push({ x: currentRect.x, y: currentRect.y, width: currentRect.width, height: currentRect.height });
            zones.value = newZones;
          }
        });
        DWTObject.Viewer.on('pageAreaUnselected', () => zones.value = []);
        DWTObject.Viewer.on("click", () => handleBufferChange());
        if(Dynamsoft.Lib.env.bWin)
          DWTObject.MouseShape = false;
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
      let selection = DWTObject.SelectedImagesIndices;
      selected.value = selection;
      buffer.updated = _updated;
      buffer.current = DWTObject.CurrentImageIndexInBuffer;
      buffer.count = DWTObject.HowManyImagesInBuffer;
    }

    onMounted(() => {
      Dynamsoft.Ready(function() {
        if(!Dynamsoft.Lib.env.bWin || !Dynamsoft.Lib.product.bChromeEdition) {
          //unSupportedEnv.value = true;
          featureSet = { scan: 0b1, load: 0b100, save: 0b1000, upload: 0b10000, barcode: 0b100000, uploader: 0b1000000 };
          features = 0b1111101;
          initialStatus = 0;
        } 
        if(DWTObject === null) 
          loadDWT(true);
        
      });
    })
    
    watch(buffer,() => {
      if (buffer.count > 0) {
          runtimeInfo.curImageTimeStamp = (new Date()).getTime();
          runtimeInfo.showAbleWidth = (DWTObject.HowManyImagesInBuffer > 1 ? width - 17 : width)-4;
          runtimeInfo.showAbleHeight = height-4;
          runtimeInfo.ImageWidth = DWTObject.GetImageWidth(buffer.current);
          runtimeInfo.ImageHeight = DWTObject.GetImageHeight(buffer.current);
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
