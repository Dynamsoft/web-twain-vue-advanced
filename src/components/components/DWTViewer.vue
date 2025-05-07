<script lang="jsx">
import { defineComponent, ref, watch, inject, onUpdated } from 'vue';


export default defineComponent({
  props: {
    blocks: Number,
    dwt: Object,
    buffer: Object,
    zones: Array,
    containerId: String,
    runtimeInfo: Object,
    bNoNavigating: Boolean,
    barcodeRects: Array,
    handleBufferChange: Function,
    handleOutPutMessage: Function
  },

  setup(props) {

    let handleViewerSizeChange = inject("handleViewerSizeChange",Function);
    let handleOutPutMessage = inject("handleOutPutMessage",Function);

    // const
    const re = /^\d+$/;
    const width = "590px";
    const height = "522px";
    let imageEditor;

    if(props.blocks !== undefined) {
      switch (props.blocks) {
        case 0:
          width = "100%"; height = "100%";
          break;
        case 1:
          width = "100%";
          break;
        case 2:
          height = "100%";
          break;
        default:
          break;
      }
    }

    const viewReady = ref(false);
    const bShowChangeSizeUI = ref(false);
    const newWidth = ref(props.runtimeInfo.ImageHeight);
    const newHeight = ref(props.runtimeInfo.ImageWidth);
    const interpolationMethod = ref("1");
    const previewMode = ref("1");


    let toolList = [
      {
        value: "editor",
        src: require('@/assets/images/ShowEditor.png'),
        title: "Show Image Editor",
        alt: "Show Image Editor"
      },
      {
        value: "rotateL",
        src: require('@/assets/images/RotateLeft.png'),
        title: "Rotate Left",
        alt: "Rotate Left"
      },
      {
        value: "rotateR",
        src: require('@/assets/images/RotateRight.png'),
        title: "Rotate Right",
        alt: "Rotate Right"
      },
      {
        value: "rotate180",
        src: require('@/assets/images/Rotate180.png'),
        title: "Rotate 180",
        alt: "Rotate 180"
      },
      {
        value: "mirror",
        src: require('@/assets/images/Mirror.png'),
        title: "Mirror",
        alt: "Mirror"
      },
      {
        value: "flip",
        src: require('@/assets/images/Flip.png'),
        title: "Flip",
        alt: "Flip"
      },
      {
        value: "removeS",
        src: require('@/assets/images/RemoveSelectedImages.png'),
        title: "Remove Selected Images",
        alt: "Remove Selected Images"
      },
      {
        value: "removeA",
        src: require('@/assets/images/RemoveAllImages.png'),
        title: "Remove All Images",
        alt: "Remove All images"
      },
      {
        value: "changeSize",
        src: require('@/assets/images/ChangeSize.png'),
        title: "Change Image Size",
        alt: "Change Image Size"
      },
      {
        value: "crop",
        src: require('@/assets/images/Crop.png'),
        title: "Crop",
        alt: "Crop"
      },
    ]

    const crop = () => {
      if(props.zones.length === 0) {
        handleOutPutMessage("Please select where you want to crop first!", "error");
      }else if(props.zones.length > 1) {
        handleOutPutMessage("Please select only one rectangle to crop!", "error");
      }else {
        let _zone = props.zones[0];
        props.dwt.Crop(
          props.buffer.current,
          _zone.x, _zone.y, _zone.x + _zone.width, _zone.y + _zone.height
        );
      }
    }

    const changeImageSizeOK = () => {
      props.dwt.ChangeImageSize(props.buffer.current, newWidth.value, newHeight.value, parseInt(interpolationMethod.value));
      bShowChangeSizeUI.value = !bShowChangeSizeUI.value;
    }

    const handleInterpolationMethodChange = (event) => {
      interpolationMethod.value = event.target.value;
    }

    const handleQuickEdit = (event) => {
      if(props.buffer.count === 0) {
        handleOutPutMessage("There is no image in Buffer!", "error");
        return;
      }
      if(props.bNoNavigating) {
        handleOutPutMessage("Navigation not allowed", "error");
        return;
      }
      switch(event.target.getAttribute("value")) {
        case "editor":
          imageEditor = props.dwt.Viewer.createImageEditor();
          imageEditor.show();
          break;
        case "rotateL":
          props.dwt.RotateLeft(props.buffer.current);
          break;
        case "rotateR":
          props.dwt.RotateRight(props.buffer.current);
          break;
        case "rotate180":
          props.dwt.Rotate(props.buffer.current, 180, true);
          break;
        case "mirror":
          props.dwt.Mirror(props.buffer.current);
          break;
        case "flip":
          props.dwt.Flip(props.buffer.current);
          break;
        case "removeS":
          props.dwt.RemoveAllSelectedImages();
          break;
        case "removeA":
          props.dwt.RemoveAllImages();
          handleNavigation("removeAll");
          break;
        case "changeSize":
          bShowChangeSizeUI.value = !bShowChangeSizeUI.value;
          break;
        case "crop":
          crop();
          break;
        case "changeImageSizeOK":
          changeImageSizeOK();
          break;
      }
    }

    const handleNewSize = (event, bHeight) => {
        if (!re.test(event.target.value)) {
            return;
        } else {
            if (bHeight)
                newHeight.value = event.target.value;
            else
                newWidth.value = event.target.value;
        }
    }

    const handleNavigation = (action) => {
      switch(action) {
        case "first":
          props.dwt.CurrentImageIndexInBuffer = 0;
          break;
        case "previous":
          props.dwt.CurrentImageIndexInBuffer = (props.buffer.current > 0) && (props.buffer.current - 1);
          break;
        case "next":
          props.dwt.CurrentImageIndexInBuffer = (props.buffer.current < props.buffer.count) && (props.buffer.current + 1);
          break;
        case "last":
          props.dwt.CurrentImageIndexInBuffer = props.buffer.count - 1;
          break;
        default:
          break;
      }
      props.handleBufferChange();
    }

    const handlePreviewModeChange = (event) => {
      let _newMode = "";
      if(event && event.target) {
        _newMode = event.target.value;
      } else {
        if(parseInt(event) > 0 && (parseInt(event) < 6)) _newMode = parseInt(event).toString();
      }
      if(_newMode !== previewMode.value) {
        if(props.bNoNavigating) {
          handleOutPutMessage("Navigation not allowed!", "error");
          return;
        }
        if (previewMode.value === "1" && props.barcodeRects.length > 0) {
          handleOutPutMessage("Can't change view mode when barcode rects are on display!", "error");
          return;
        }
        previewMode.value = _newMode;
        props.dwt.Viewer.setViewMode(parseInt(_newMode), parseInt(_newMode));
        props.dwt.MouseShape = (parseInt(_newMode) > 1);
        handleNavigation("viewModeChange");
      }
    }

    onUpdated(() => {
      if(props.barcodeRects.length !== 0){
            !props.bNoNavigating && handlePreviewModeChange("1");
        }
        if (document.getElementById(props.containerId).offsetWidth !== 0) {
            handleViewerSizeChange({
                width: document.getElementById(props.containerId).offsetWidth,
                height: document.getElementById(props.containerId).offsetHeight
            });
        }
    })

    watch(() => props.dwt, 
    () => {
      viewReady.value = true;
    })

    watch(() => viewReady.value, 
    (newViewReady,prevViewReady) => {
      if(props.dwt !== null && newViewReady && !prevViewReady) {
        props.dwt.Viewer.width = width;
        props.dwt.Viewer.height = height;
      }
    })

    //Change ImageSize
    watch(() => props.runtimeInfo.ImageHeight,
    () => {
    newHeight.value = props.runtimeInfo.ImageHeight;
    }
    )

    watch(() => props.runtimeInfo.ImageWidth, 
    () => {
      newWidth.value = props.runtimeInfo.ImageWidth;
    }
    )

    return () => (
      <>
        <div class="dwt-viewer">
          <div class="tool-list">
            <ul onClick={ handleQuickEdit }>
              {
                toolList.map(item => {
                  return (<li>
                            <img src={ item.src } title={ item.title } alt={ item.alt } value={ item.value }/>
                          </li>)
                })
              }
            </ul>
            <div className="dwt-imageSizeEditor" style={ bShowChangeSizeUI.value ? { visibility: "visible" } : { visibility: "hidden" } }>
                <ul>
                    <li>
                        <label>New Height (pixel): 
                          <input tabIndex="6" type="text" value={ newHeight.value } className="width_48p floatR" onChange={ (event) => handleNewSize(event, true) } />
                        </label>
                    </li>
                    <li>
                        <label>New Width (pixel): 
                          <input tabIndex="6" type="text" value={ newWidth.value } className="width_48p floatR" onChange={ (event) => handleNewSize(event) } />
                        </label>
                    </li>
                    <li >Interpolation method:
                        <select value={ interpolationMethod.value } 
                                className="width_48p floatR" 
                                onChange={ (event) => handleInterpolationMethodChange(event) }>
                        <option value="1">NearestNeighbor</option>
                        <option value="2">Bilinear</option>
                        <option value="3">Bicubic</option>
                        </select>
                    </li>
                    <li style={{ textAlign: "center" }}>
                        <button value="changeImageSizeOK" onClick={ (event) => handleQuickEdit(event) } >OK</button>
                        <button value="changeSize" onClick={ (event) => handleQuickEdit(event) } >Cancel</button>
                    </li>
                </ul>
            </div>
          </div>
          <div class="viewer-content">
            <div style={{ position: "relative", float: "left", width: width, height: height }} id={ props.containerId } class="viewer-main">
            {props.barcodeRects.map((_rect, _index) => (
                  <div key={_index} className="barcodeInfoRect" style={{ left: _rect.x + "px", top: _rect.y + "px", width: _rect.w + "px", height: _rect.h + "px" }} >
                      <div className="spanContainer"><span>[{_index + 1}]</span>
                      </div>
                  </div>
              ))}
            </div>
            <div class="pagination">
              <div class="pagination-tool">
                <button value="first" 
                        onClick={ (event) => handleNavigation(event.target.value) }
                        disabled={ props.buffer.current < 1 ? true : false }>
                  { " |< " }
                </button>
                &nbsp;
                <button value="previous" 
                        onClick={ (event) => handleNavigation(event.target.value) }
                        disabled={ props.buffer.current < 1 ? true : false }>
                  { " < " }
                </button>
                <input style="text-align:right" type="text" value={ props.buffer.current > -1 ? props.buffer.current + 1 : 0 } readonly  disabled="disabled"/>
                  &nbsp;{ "/" }&nbsp;
                <input type="text" value={ props.buffer.count > 0 ? props.buffer.count : 0 } readonly  disabled="disabled"/>
                <button value="next" 
                        onClick={ (event) => handleNavigation(event.target.value) }
                        disabled={ props.buffer.count-1 === props.buffer.current ? true : false } >
                  { " > " }
                </button>
                &nbsp;
                <button value="last" 
                        disabled={ props.buffer.count-1 === props.buffer.current ? true : false }
                        onClick={ (event) => handleNavigation(event.target.value) }>
                  { " >| " }
                </button>
              </div>
              <div>
                <select class="previewMode" value={ previewMode.value } onChange={ (event) => handlePreviewModeChange(event) }>
                  <option value="1">1x1</option>
                  <option value="2">2x2</option>
                  <option value="3">3x3</option>
                  <option value="4">4x4</option>
                  <option value="5">5x5</option>
                </select>
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
  .dwt-viewer {
    display: flex;
    width: 100%;
    height: 580px;
    background-color: white;
  }
  .tool-list {
    width: 60px;

    ul {
      height: 100%;
      box-sizing: border-box;
      border-left: 1px solid gainsboro;
      border-right: 1px solid gainsboro;
      border-bottom: 1px solid gainsboro;
    }
    li {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 60px;
      height: 58px;
      border-top: 1px solid gainsboro;
      list-style-type: none;
      box-sizing: border-box;
      cursor: pointer;

      img {
        width: 32px;
        height: 32px;
      }
    }
  }
  .viewer-content {
    width: 100%;

    .viewer-main {
      width: 100%;
      height: 90%;
    }

    .pagination {
      display: flex;
      justify-content: flex-end;
      align-items: center;
      width: 100%;
      height: 10%;
      padding-right: 20px;
      background-color: #eaeeee;
      box-sizing: border-box;

      .pagination-tool {
        display: flex;
        justify-content: center;
        align-items: center;
        margin-right: 130px;

        button {
          padding: 1px 7px;
        }

        div {
          margin: 0 5px;
          padding: 2px 5px;
          border: 1px solid;
          border-radius: 2px;
          font-size: 13px;
          user-select: none;
        }

        div:hover {
          background-color: rgb(213, 214, 214);
          cursor: pointer;
        }

        input {
          width: 30px;
          margin: 0 5px;
          background: #eaeeee;
          border: none;
        }
      }
    }
  }

  .dwt-imageSizeEditor {
    position: absolute;
    top: 588px;
    width: 300px;
    height: auto;
    padding: 10px 15px;
    z-index: 1;
    font-size: 14px;
    text-align: left;
    color: #606060;
    background-color: #f5f5f5;
    border: 1px solid #ccc;
    border-collapse: collapse;
    box-shadow: 4px 4px 18px #ccc;
    -webkit-box-shadow: 4px 4px 18px #ccc;
    -moz-box-shadow: 4px 4px 18px #ccc;
  }

  .dwt-imageSizeEditor ul {
    border:none
  }

  .dwt-imageSizeEditor li {
      display: flex;
      justify-content: space-between;
      width: 100%;
      height: 18px;
      margin: 8px 0;
      line-height: 19px;
      text-align: center;
      border: none;

      label{
        display: flex;
        justify-content: space-between;
        width: 100%;
      }

  }

  .dwt-imageSizeEditor input {
      width: 48%;
      padding-left: 3px;
      background: #fff;
      border: solid 1px #ccc;
      box-sizing: border-box;
  }

  .dwt-imageSizeEditor select {
      width: 48%;
      border: solid 1px #ccc;
      background: #fff;
  }

  .dwt-imageSizeEditor input[type='text'] {
      padding-left: 3px;
  }

  .dwt-imageSizeEditor input[type='button'] {
      margin: 6px 5px 0;
      padding: 5px 5px;
  }

  .dwt-imageSizeEditor button {
    width: 48%;
    height: 20px;
  }

  .barcodeInfoRect {
    position: absolute;
    z-index: 10;
    border: 1px solid red;
  }

  .image-container * {
      box-sizing: content-box;
  }

  .spanContainer {
      position: absolute;
      left: 0px;
      top: -14px;
      color: blue;
      font-size: 12px;
      font-weight: bold;
  }
</style>
