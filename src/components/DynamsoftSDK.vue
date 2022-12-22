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
      initialStatus = 255 - (features & 0b11100011);
    }

    // status
    let startTime  = ref((new Date()).getTime());
    let unSupportedEnv = ref(false);
    let dwt = ref();
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
		  Dynamsoft.DWT.ProductKey = 't01016QAAAEvOkEkSgAXzAjLcLgaQLc7rsPKTL/ka12h61qfgRA2s7NFM7wo17hlZeyCgZ/Hw8NjwD/Rg3Jl0u2pwAeHCmHpuULSMVT9cnJqjx3g7/uS02ofR1E1yk8nwncm2G4lfLto=';
      Dynamsoft.DWT.AutoLoad = true;
      Dynamsoft.DWT.RegisterEvent('OnWebTwainReady',() => webTwain_OnReady())
    }

    const webTwain_OnReady = () => {
      DWObject = Dynamsoft.DWT.GetWebTwain('dwtcontrolContainer');
      if(DWObject) {
        DWObject.Viewer.width = width;
        DWObject.Viewer.height = height;
        DWObject.Viewer.setViewMode(1, 1);
        DWObject.Viewer.show();
        handleStatusChange(1);
        dwt.value = DWObject;
        if(DWObject) {
          DWObject.RegisterEvent("OnBitmapChanged", (changedIndex, changeType) => handleBufferChange(changedIndex, changeType));
          DWObject.RegisterEvent("OnPostTransfer", () => handleBufferChange());
          DWObject.RegisterEvent("OnPostLoad", () => handleBufferChange());
          DWObject.RegisterEvent("OnPostAllTransfers", () => DWObject.CloseSource());
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
          unSupportedEnv.value = true;
          return;
        } else {
          if(DWObject === null) 
            loadDWT(true);
        }
      });
    })
    
    watch(buffer,() => {
      if (buffer.count > 0) {
          runtimeInfo.curImageTimeStamp = (new Date()).getTime();
          runtimeInfo.showAbleWidth = DWObject.HowManyImagesInBuffer > 1 ? width - 16 : width;
          runtimeInfo.showAbleHeight = height;
          runtimeInfo.ImageWidth = DWObject.GetImageWidth(buffer.current);
          runtimeInfo.ImageHeight = DWObject.GetImageHeight(buffer.current);
      }
    })

    provide("handleViewerSizeChange",handleViewerSizeChange);
    provide("handleStatusChange",handleStatusChange);

    return () => (
      unSupportedEnv.value ? <> <div>Please use Chrome, Firefox or Edge on Windows!</div> </>
      :
      <>
        <div class="dwt-content">
          <DWTInterface
            Dynamsoft = { Dynamsoft }
            features = { features }
            containerId = { containerId }
            startTime = { startTime.value }
            dwt = { dwt.value }
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
