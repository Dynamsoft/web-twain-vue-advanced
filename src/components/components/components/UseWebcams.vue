<script lang="jsx">
import { defineComponent, ref, reactive, watch, inject, onUpdated } from 'vue';
import WebcamOption from './WebcamOption.vue'
import RangePicker from './RangePicker.vue'


export default defineComponent({
  props: {
    Dynamsoft: Object,
    dwt: Object,
    features: Number,
    showUseWebcams: Boolean
  },
  setup(props) {
    let handleStatusChange = inject("handleStatusChange", Function);
    let handleException = inject("handleException", Function);
    const cameras = ref([]);
    const cameraReady = ref(false);
    const cameraSettings = reactive([]);

    let deviceSetup = reactive({
      currentCamera: "Looking for devices..",
      nPixelType: "0",
      nResolution: "100",
      isVideoOn: false
    });

    let bShowRangePicker = ref(false);
    let rangePicker = ref({
        bMutable: false,
        bCamera: false,
        val: 0,
        min: 0,
        max: 0,
        defaultVal: 0,
        step: 0,
    });
    let rangePickerTitle = ref({
      prop:"",
      value:""
    })

    const onCameraChange = (value) => {
      deviceSetup.currentCamera = value;

      if(value === "noCamera") {
        if(!cameraReady.value) {
          cameraReady.value = true;
          handleStatusChange(2);
        }
        return;
      }

      // props.dwt.Addon.Webcam.StopVideo();

      if(props.dwt.Addon.Webcam.SelectSource(value)) {
        let mediaTypes = props.dwt.Addon.Webcam.GetMediaType(),
            _mediaTypes = [],
            _currentMediaType = mediaTypes.GetCurrent();
        let frameRates = props.dwt.Addon.Webcam.GetFrameRate(),
            _frameRates = [],
            _currentFrameRate = frameRates.GetCurrent();
        let resolutions = props.dwt.Addon.Webcam.GetResolution(),
            _resolutions = [],
            _currentResolution = resolutions.GetCurrent();
        let _advancedSettings = [],
            _advancedCameraSettings = [];

        mediaTypes = mediaTypes._resultlist;
        frameRates = frameRates._resultlist;
        resolutions = resolutions._resultlist;

        for(let i=0; i<mediaTypes.length-1; i++) {
          mediaTypes[i] === _currentMediaType
            ? _mediaTypes[i] = { value: mediaTypes[i].toString(), checked: true }
            : _mediaTypes[i] = { value: mediaTypes[i].toString(), checked: false }
        }
        for(let i=0; i<frameRates.length-1; i++) {
          frameRates[i] === _currentFrameRate
            ? _frameRates[i] = { value: frameRates[i].toString(), checked: true  }
            : _frameRates[i] = { value: frameRates[i].toString(), checked: false };
        }
        for(let i=0; i<resolutions.length-1; i++) {
          resolutions[i] === _currentResolution
            ? _resolutions[i] = { value: resolutions[i].toString(), checked: true }
            : _resolutions[i] = { value: resolutions[i].toString(), checked: false };
        }
        _advancedSettings = Object.keys(props.Dynamsoft.DWT.EnumDWT_VideoProperty).map((_value) => { return { value: _value.substr(3) } });
        _advancedCameraSettings = Object.keys(props.Dynamsoft.DWT.EnumDWT_CameraControlProperty).map((_value) => { return { value: _value.substr(4) } });
        cameraSettings.length = 0;
        cameraSettings.push(
          {
            name: "Media Type",
            value: '1',
            items: _mediaTypes
          },
          {
            name: "Frame Rate",
            value: '2',
            items: _frameRates
          },
          {
            name: "Resolution",
            value: '3',
            items: _resolutions
          },
          {
            name: "Video Setup",
            value: '4',
            items: _advancedSettings
          },
          {
            name: "Camera Setup",
            value: '5',
            items: _advancedCameraSettings
          }
        );
        if(!cameraReady.value) {
          cameraReady.value = true;
          handleStatusChange(2);
        }
      } else {
        handleException({
          code: -2,
          message: "Can't use the Webcam" + value + ", please make sure it's not in use!"
        });
        if(!cameraReady.value) {
          cameraReady.value = true;
          handleStatusChange(2);
        }
      }
    }

    const toggleShowVideo = () => {
      if(deviceSetup.isVideoOn === false) {
        toggleCameraVideo(true);
      } else {
        toggleCameraVideo(false);
      }
    }

    const toggleCameraVideo = (bShow) => {
      if(props.dwt) {
        if(bShow) {
          playVideo();
          deviceSetup.isVideoOn = true;
        } else {
          props.dwt.Addon.Webcam.StopVideo();
          deviceSetup.isVideoOn = false;
        }
      }
    }

    const handleRangeChange = (event) => {
        let value = event.target.value ? event.target.value : event.target.getAttribute("value");
        if (value === "reset-range") {
            let prop = event.target.getAttribute("prop");
            let _type = event.target.getAttribute("_type");
            let _default = event.target.getAttribute("_default");
            rangePicker.value.val = _default;
            _type === "camera"
                ? DWObject.Addon.Webcam.SetCameraControlPropertySetting(Dynamsoft.DWT.EnumDWT_CameraControlProperty["CCP_" + prop], _default, false)
                : DWObject.Addon.Webcam.SetVideoPropertySetting(Dynamsoft.DWT.EnumDWT_VideoProperty["VP_" + prop], _default, false);
            bShowRangePicker.value = false;
        } else if (value === "close-picker") {
            bShowRangePicker.value = false;
        } else {
            let _type = event.target.getAttribute("_type");
            let prop = event.target.getAttribute("prop");
            rangePicker.value.val = value;
            _type === "camera"
                ? DWObject.Addon.Webcam.SetCameraControlPropertySetting(Dynamsoft.DWT.EnumDWT_CameraControlProperty["CCP_" + prop], value, false)
                : DWObject.Addon.Webcam.SetVideoPropertySetting(Dynamsoft.DWT.EnumDWT_VideoProperty["VP_" + prop], value, false);
        }
    }

    const playVideo = (config) => {
      let basicSetting, moreSetting;
      if(config) {
        if(config.prop === "Video Setup" || config.prop === "Camera Setup") {
          let bCamera = true;
          if(config.prop === "Video Setup") {
            bCamera = false;
            basicSetting = props.dwt.Addon.Webcam.GetVideoPropertySetting(props.Dynamsoft.DWT.EnumDWT_VideoProperty["VP_" + config.value]);
            moreSetting = props.dwt.Addon.Webcam.GetVideoPropertyMoreSetting(props.Dynamsoft.DWT.EnumDWT_VideoProperty["VP_" + config.value]);
          } else {
            basicSetting = props.dwt.Addon.Webcam.GetCameraControlPropertySetting(props.Dynamsoft.DWT.EnumDWT_CameraControlProperty["CCP_" + config.value]);
            moreSetting = props.dwt.Addon.Webcam.GetCameraControlPropertyMoreSetting(props.Dynamsoft.DWT.EnumDWT_CameraControlProperty["CCP_" + config.value]);
          }
          let value = basicSetting.GetValue();
          let min = moreSetting.GetMinValue();
          let max = moreSetting.GetMaxValue();
          let defaultvalue = moreSetting.GetDefaultValue();
          let bMutable = true;
          if(min === max && value === defaultvalue && min === value) {
            bMutable = false;
          };

          let newRangePicker = {
            bMutable: bMutable,
            bCamera: bCamera, 
            val: value,
            min: min,
            max: max,
            defaultVal: defaultvalue,
            step: moreSetting.GetSteppingDelta(),
          }

          bShowRangePicker.value = true;
          rangePicker.value = newRangePicker;
          rangePickerTitle.value = config;

          return;

        } else {
          props.dwt.Addon.Webcam.StopVideo();
          switch (config.prop) {
            case "Frame Rate": props.dwt.Addon.Webcam.SetFrameRate(config.value); break;
            case "Media Type": props.dwt.Addon.Webcam.SetMediaType(config.value); break;
            case "Resolution": props.dwt.Addon.Webcam.SetResolution(config.value); break;
            default: break;
          }
        }
      }
      /**
       * NOTE: The video playing is not smooth, there is a zoom-out effect (unwanted)
       */
      if((config && deviceSetup.isVideoOn) || !config) {
        props.dwt.Addon.Webcam.PlayVideo(props.dwt, 80, () => { });
      }
    }

    const captureImage = () => {
      if(props.dwt) {
        let funCaptureImage = () => {
          setTimeout(() => {
            toggleCameraVideo(false);
          }, 50);
        }
        props.dwt.Addon.Webcam.CaptureImage(funCaptureImage, funCaptureImage);
      }
    }

    watch(() => props.dwt, () => {
      if(props.dwt) {
        if(props.features & 0b10) {
          let cameraNames = props.dwt.Addon.Webcam.GetSourceList();
          cameras.value = cameraNames;
          if(cameraNames.length > 0) {
            onCameraChange(cameraNames[0]);
          }
        }
      }
    },{
      immediate: true
    })

    onUpdated(() => {
      if(!props.showUseWebcams){
        deviceSetup.isVideoOn = false;
        props.dwt.Addon.Webcam.StopVideo();
      }
    })

    return () => (
      <>
        <div class="dwt-use-webcams-content">
          <div>
            <select onChange={ (e) => onCameraChange(e.target.value) }>
              {
                cameras.value.length > 0 ?
                  cameras.value.map((_name, _index) => {
                    return <option value={ _index } key={ _index } >{ _name }</option>
                  })
                  :
                  <option value="noCamera">Looking for devices..</option>
              }
            </select>
          </div>
          <div class="dwt-webcams-option">
            {
              cameraSettings.length > 0 ? (
                <WebcamOption
                  targetObject = { deviceSetup.currentCamera }
                  valuePacks = { cameraSettings }
                  current = { "Resolution" }
                  handleValuePicking = { (valuePair) => playVideo(valuePair) } 
                />
              ) : ""
            }
          </div>
          <div class="dwt-webcams-scan">
            <button class="btn-showVideo" onClick={ toggleShowVideo }>
              <span>{ deviceSetup.isVideoOn ? "Hide Video" : "Show Video" }</span>
            </button>
            <button 
              class={ deviceSetup.isVideoOn ? "btn-capture" : "btn-capture btn-disabled" }
              disabled={ deviceSetup.isVideoOn ? false : true } 
              onClick={ captureImage }>
              <span>Capture</span>
            </button>
          </div>
        </div>
        {bShowRangePicker.value ? (
          <RangePicker
            rangePicker = { rangePicker.value }
            rangePickerTitle = {rangePickerTitle.value}
            handleRangeChange = { (event) => handleRangeChange(event) }
          />
        ) : ""
        }
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  .dwt-use-webcams-content {
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      padding: 12px 20px 15px;
      border-bottom: solid 1px #ccc;

      select{
        width: 280px;
        height: 26px;
        margin-top: 3px;
        margin-bottom: 6px;
        border: solid 1px #ccc;
        border-radius: 3px;
        -webkit-border-radius: 3px;
        -moz-border-radius: 3px;
      }

      .dwt-webcams-scan {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 100%;
        margin-top: 15px;
        user-select: none;

        .btn-showVideo {
          display: flex;
          justify-content: center;
          align-items: center;
          width: 40%;
          height: 30px;
          font-size: 14px;
          color: white;
          border:none;
          border-radius: 5px;
          background-color: rgb(80, 168, 225);
        }

        .btn-showVideo:hover {
          background-color: #61c2ec;
          cursor: pointer;
        }
        .btn-capture {
          display: flex;
          justify-content: center;
          align-items: center;
          width: 40%;
          height: 30px;
          font-size: 14px;
          color: white;
          border:none;
          border-radius: 5px;
          background-color: rgb(80, 168, 225);
        }
        .btn-capture:hover {
          cursor: pointer;
        }
      }
    }
  .btn-disabled{
    background-color: #aaaaaa !important;
  }
</style>
