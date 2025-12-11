<script lang="jsx">
import { defineComponent, ref, reactive, watch, inject } from 'vue';

export default defineComponent({
  props: {
    Dynamsoft: Object,
    dwt: Object,
    features: Number
  },
  setup(props) {
    let handleOutPutMessage = inject("handleOutPutMessage", Function);
    const scanners = ref([]);

    let deviceSetup = reactive({
      currentScanner: "Looking for devices..",
      currentCamera: "Looking for devices..",
      bShowUI: false,
      bADF: false,
      bDuplex: false,
      nPixelType: "0",
      nResolution: "100",
      isVideoOn: false
    });

    const onSourceChange = (value) => {
      deviceSetup.currentScanner = value;
      if(value === "noScanner") return;
      if(Dynamsoft.Lib.env.bMac) {
        if(value.indexOf("ICA") === 0) {
          deviceSetup.noUI = true;
        } else {
          deviceSetup.noUI = false;
        }
      }
    }

    const _onScannerSetupChange = (option, value) => {
      let newDeviceSetup = deviceSetup;
      switch(option) {
        case "bShowUI":
          newDeviceSetup.bShowUI = value;
          break;
        case "bADF":
          newDeviceSetup.bADF = value;
          break;
        case "bDuplex":
          newDeviceSetup.bDuplex = value;
          break;
        case "nPixelType":
          newDeviceSetup.nPixelType = value;
          break;
        case "nResolution":
          newDeviceSetup.nResolution = value;
          break;
        default:
          break;
      }
      deviceSetup = newDeviceSetup;
    }

    const handleScannerSetupChange = (e, option) => {
      switch(option.substr(0, 1)) {
        case "b":
          _onScannerSetupChange(option, e.target.checked);
          break;
        case "n":
          _onScannerSetupChange(option, e.target.value);
          break;
        default:
          break;
      }
    }

    const acquireImage = () => {
      if(props.dwt){
        props.dwt.GetDevicesAsync().then((devices)=>{
            for (var i = 0; i < devices.length; i++) { // Get how many sources are installed in the system
                if (devices[i].displayName === deviceSetup.currentScanner) {
                    return devices[i];
                }
            }
        }).then((device) =>{
            return props.dwt.SelectDeviceAsync(device);
        }).then(()=>{
            return props.dwt.AcquireImageAsync({
                IfShowUI: deviceSetup.bShowUI,
                PixelType: deviceSetup.nPixelType,
                Resolution: deviceSetup.nResolution,
                IfFeederEnabled: deviceSetup.bADF,
                IfDuplexEnabled: deviceSetup.bDuplex,
                IfDisableSourceAfterAcquire: true,
                IfGetImageInfo: true,
                IfGetExtImageInfo: true,
                extendedImageInfoQueryLevel: 0
                /**
                 * NOTE: No errors are being logged!!
                 */
            });
        }).then(()=>{
            handleOutPutMessage("Acquire success!", "important")
        }).catch(function (exp) {
            handleOutPutMessage("Acquire failure!", "error")
        });
      }
    }

    watch(() => props.dwt, () => {
      if(props.dwt) {
        if(props.features & 0b1) {
          props.dwt.GetDevicesAsync().then((devices)=>{
            let sourceNames = [];
            for (var i = 0; i < devices.length; i++) { // Get how many sources are installed in the system
                sourceNames.push(devices[i].displayName);
            }
            scanners.value = sourceNames;
            if (sourceNames.length > 0) onSourceChange(sourceNames[0]);

          }).catch(function (exp) {
              alert(exp.message);
          });
        }
      }
    },{
      immediate: true
    })

    return () => (
      <>
        <div class="dwt-custom-scan-content">
          <div>
            <select style="width:280px" onChange={ (e) => onSourceChange(e.target.value) }>
              {
                scanners.value.length > 0 ?
                  scanners.value.map((_name, _index) => {
                    return <option value={ _name } key={ _index }>{ _name }</option>
                  })
                  :
                  <option value="noScanner">Looking for devices..</option>
              }
            </select>
          </div>
          <div style="display:flex; margin-top:5px">
            {
              deviceSetup.noUI ?
              ""
              :
              <label for="showUI">
                <input type="checkbox"
                       id="showUI"
                       checked={ deviceSetup.bShowUI }
                       onChange={ (e) => handleScannerSetupChange(e, "bShowUI") }
                />Show UI</label>
            }
            <label for="pageFeeder">
              <input type="checkbox"
                     id="pageFeeder"
                     checked={ deviceSetup.bADF }
                     onChange={ (e) => handleScannerSetupChange(e, "bADF") }
              />Page Feeder</label>
            <label for="duplex">
              <input type="checkbox"
                     id="duplex"
                     checked={ deviceSetup.duplex }
                     onChange={ (e) => handleScannerSetupChange(e, "bDuplex") }
              />Duplex</label>
          </div>
          <div class="custom-scan-setup">
            <select value={ deviceSetup.nPixelType }
                    onChange={ (e) => handleScannerSetupChange(e, "nPixelType") }>
              <option value="0">B{'&'}W</option>
              <option value="1">Gray</option>
              <option value="2">Color</option>
            </select>
            <select value={ deviceSetup.nResolution }
                    onChange={ (e) => handleScannerSetupChange(e, "nResolution") }>
              <option value="100">100 DPI</option>
              <option value="200">200 DPI</option>
              <option value="300">300 DPI</option>
              <option value="600">600 DPI</option>
            </select>
          </div>
          <button
            class = {scanners.value.length > 0 ?  "btn-scan" : "btn-scan btn-disable"}
            disabled = {scanners.value.length > 0 ? false : true}
            onClick = { acquireImage }>
            <span>Scan</span>
          </button>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  .dwt-custom-scan-content {
      display: flex;
      flex-direction: column;
      justify-content: center;
      padding: 12px 20px 15px;
      border-bottom: solid 1px #ccc;
      align-items: flex-start;

      select{
        height: 26px;
        margin-top: 3px;
        margin-bottom: 6px;
        border: solid 1px #ccc;
        border-radius: 3px;
        -webkit-border-radius: 3px;
        -moz-border-radius: 3px;
      }

      label{
        display: flex;
        align-items: center;
        margin-right: 15px;
      }

      .btn-scan {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
        height: 30px;
        margin-top: 5px;
        font-size: 14px;
        color: white;
        border: none;
        border-radius: 5px;
        background-color: rgb(80, 168, 225);
        user-select: none;
      }

      .btn-scan:hover {
        background-color: #61c2ec;
        cursor: pointer;
      }
    }

  .custom-scan-setup{
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    width: 100%;
    margin-top: 5px;
    select{
      width: 130px;
    }
  }

  .btn-disable{
    background: #aaa !important;
  }
</style>
