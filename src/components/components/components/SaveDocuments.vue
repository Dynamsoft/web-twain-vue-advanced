<script lang="jsx">
import { defineComponent, ref, reactive, watch, inject } from 'vue';

export default defineComponent({
  props: {
    Dynamsoft: Object,
    dwt: Object,
    buffer: Object,
    selected: Array,
    features: Number,
  },
  setup(props) {
    let handleOutPutMessage = inject("handleOutPutMessage", Function);
    let handleException = inject("handleException", Function);
    let handleStatusChange = inject("handleStatusChange", Function);

    const saveFileName = ref((new Date()).getTime().toString());
    const saveFileFormat = ref("jpg");

    const bMulti = ref(false);
    const bUseFileUploader = ref(false);
    const fileUploaderReady = ref(false);

    let fileUploaderManager = reactive({});

    const handleFileNameChange = (event) => {
      saveFileName.value = event.target.value;
    }

    const handleSaveConfigChange = (event) => {
      let format = event.target.value;
      switch(format) {
        case "multiPage":
          bMulti.value = event.target.checked;
          break;
        case "tif":
          saveFileFormat.value = event.target.value;
          bMulti.value = true;
          break;
        case "pdf":
          saveFileFormat.value = event.target.value;
          bMulti.value = true;
          break;
        case "bmp":
          saveFileFormat.value = event.target.value;
          bMulti.value = false;
          break;
        case "jpg":
          saveFileFormat.value = event.target.value;
          bMulti.value = false;
          break;
        case "png":
          saveFileFormat.value = event.target.value;
          bMulti.value = false;
          break;
        default:
          break;
      }
    }

    const toggleUseUploade = (event) => {
      bUseFileUploader.value = event.target.value;
    }

    const saveOrUploadImage = (_type) => {
      if(_type !== "local" && _type !== "server") return;
      let fileName = saveFileName.value + "." + saveFileFormat.value;
      let imagesToUpload = [];
      let fileType = 0;
      let onSuccess = () => {
        saveFileName.value = (new Date()).getTime().toString();
        _type === "local"
        ? handleOutPutMessage(fileName + " saved successfully!", "important")
        : handleOutPutMessage(fileName + " uploaded successfully!", "important");
      }
      let onFailure = (errorCode, errorString, httpResponse) => {
        (httpResponse && httpResponse !== "")
        ? handleOutPutMessage(httpResponse, "httpResponse")
        : handleException({
          code: errorCode,
          message: errorString
        })
      }
      if(bMulti.value) {
        if(props.selected.length === 1 || props.selected.length === props.buffer.count) {
          if(_type === "local") {
            switch(saveFileFormat.value) {
              case "tif":
                props.dwt.SaveAllAsMultiPageTIFF(fileName, onSuccess, onFailure);
                break;
              case "pdf":
                props.dwt.SaveAllAsPDF(fileName, onSuccess, onFailure);
                break;
              default:
                break;
            }
          } else {
            for(let i=0; i<props.buffer.count; i++) {
              imagesToUpload.push(i);
            }
          }
        } else {
          if(_type === "local") {
            switch(saveFileFormat.value) {
              case "tif":
                props.dwt.SaveSelectedImagesAsMultiPageTIFF(fileName, onSuccess, onFailure);
                break;
              case "pdf":
                props.dwt.SaveSelectedImagesAsMultiPagePDF(fileName, onSuccess, onFailure);
                break;
              default:
                break;
            }
          } else {
            imagesToUpload = props.selected;
          }
        }
      } else {
        if(_type === "local") {
          switch(saveFileFormat.value) {
            case "bmp":
              props.dwt.SaveAsBMP(fileName, props.buffer.current, onSuccess, onFailure);
              break;
            case "jpg":
              props.dwt.SaveAsJPEG(fileName, props.buffer.current, onSuccess, onFailure);
              break;
            case "tif":
              props.dwt.SaveAsTIFF(fileName, props.buffer.current, onSuccess, onFailure);
              break;
            case "png":
              props.dwt.SaveAsPNG(fileName, props.buffer.current, onSuccess, onFailure);
              break;
            case "pdf":
              props.dwt.SaveAsPDF(fileName, props.buffer.current, onSuccess, onFailure);
              break;
            default:
              break;
          }
        } else {
          imagesToUpload.push(props.buffer.current);
        }
      }
      if(_type === "server") {
        switch(saveFileFormat.value) {
            case "bmp":
              fileType = 0; break;
            case "jpg":
              fileType = 1; break;
            case "tif":
              fileType = 2; break;
            case "png":
              fileType = 3; break;
            case "pdf":
              fileType = 4; break;
          }
        let protocol = Dynamsoft.Lib.detect.ssl ? "https://" : "http://"
        let _strPort = 2020;//for testing
            // window.location.port === "" ? 80 : window.location.port;
            if (Dynamsoft.Lib.detect.ssl === true)
                _strPort = window.location.port === "" ? 443 : window.location.port;

        let strActionPage = "/upload";
        let serverUrl = protocol + window.location.hostname + ":" + _strPort + strActionPage;
        if(bUseFileUploader.value) {
          var job = fileUploaderManager.CreateJob();
          job.ServerUrl = serverUrl;
          job.FileName = fileName;
          job.ImageType = fileType;
          props.dwt.GenerateURLForUploadData(imagesToUpload, fileType, (resultURL, newIndices, enumImageType) => {
            job.SourceValue.Add(resultURL, fileName);
            job.OnUploadTransferPercentage = (job, sPercentage) => {
              handleOutPutMessage("Uploading...(" + sPercentage + "%)");
            }
            job.OnRunSuccess = (job) => {
              onSuccess()
            };
            job.OnRunFailure = (job, errorCode, errorString) => {
              onFailure(errorCode, errorString);
            }
            fileUploaderManager.Run(job);
          }, (errorCode, errorString, strHTTPPostResponseString, newIndices, enumImageType) => {
              handleException({ code: errorCode, message: errorString });
          });
        } else {
          props.dwt.HTTPUpload(serverUrl, imagesToUpload, fileType, props.Dynamsoft.DWT.EnumDWT_UploadDataFormat.Binary, fileName, onSuccess, onFailure);
        }
      }
    }

    watch(() => props.dwt, () => {
      if(props.dwt) {
        if(props.features & 0b10000000) {
          props.Dynamsoft.FileUploader.Init("", (objFileUploader) => {
            fileUploaderManager = objFileUploader;
            if(!fileUploaderReady.value) {
              fileUploaderReady.value = true;
              handleStatusChange(128);
            }
            }, (errorCode, errorString) => {
              console.log(errorString);
              // handleException({
              //   code: errorCode,
              //   message: "Initializing FileUploader failed: " + errorString
              // });
          })
        }
      }
    })

    return () => (
      <>
        <div class="dwt-save-content">
          <div class="dwt-save-fileName">
            <span>File Name:</span>
            <input type="text"
                   value={ saveFileName.value }
                   onChange={ (event) => handleFileNameChange(event) }
            />
          </div>
          <div class="dwt-save-fileType">
            <label><input type="radio" value="bmp" name="ImageType" onClick={ (event) => handleSaveConfigChange(event) } />BMP</label>
            <label><input type="radio" value="jpg" name="ImageType" onClick={ (event) => handleSaveConfigChange(event) } defaultChecked />JPEG</label>
            <label><input type="radio" value="tif" name="ImageType" onClick={ (event) => handleSaveConfigChange(event) } />TIFF</label>
            <label><input type="radio" value="png" name="ImageType" onClick={ (event) => handleSaveConfigChange(event) } />PNG</label>
            <label><input type="radio" value="pdf" name="ImageType" onClick={ (event) => handleSaveConfigChange(event) } />PDF</label>
          </div>
          <div class="dwt-save-checkBox">
            <label>
              <input type="checkbox"
                     value="multiPage"
                     disabled={ (saveFileFormat.value === "pdf" || saveFileFormat.value === "tif") ? false : true }
                     checked={ (saveFileFormat.value === "pdf" || saveFileFormat.value === "tif") && (bMulti.value ? "checked" : "") }
                     onChange={ (event) => handleSaveConfigChange(event) }
            />Upload Multiple Pages</label>
            <label><input type="checkbox" onChange={ (e) => toggleUseUploade(e) } />Use File Uploader</label>
          </div>
          <div class="dwt-save-scan">
            <button class={ props.buffer.count === 0 ? "btn-saveToLocal btn-disabled" : "btn-saveToLocal" }
                    onClick={ () => saveOrUploadImage("local") }
                    disabled = { props.buffer.count === 0 ? true : false }
            >
              Save to Local
            </button>
            <button class={ props.buffer.count === 0 ? "btn-saveToLocal btn-disabled" : "btn-saveToLocal" }
                    onClick={ () => saveOrUploadImage("server") }
                    disabled = { props.buffer.count === 0 ? true : false }
            >
              Upload to Server
            </button>
          </div>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  .dwt-save-content {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    padding: 12px 20px 15px;
    border-bottom: solid 1px #ccc;

    .dwt-save-fileName{
      width: 100%;
      margin-top: 5px;
      span{
        font-size: 12px;
      }

      input{
        width: 75%;
        height: 26px;
        margin-left: 2%;
        padding-left: 3px;
        border: solid 1px #ccc;
        border-radius: 3px;
        -webkit-border-radius: 3px;
      }
    }

    .dwt-save-fileType{
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      width: 100%;
      margin-top: 12px;
      font-size: 12px;
      
      label{
        display: flex;
        align-items: center
      }

      input{
        margin-right: 2px;
      }
    }

    .dwt-save-checkBox {
      display: flex;
      justify-content: space-between;
      width: 100%;
      margin-top: 12px;
      font-size: 13px;

      label{
        display: flex;
        align-items: center;
        height: 20px;
      }

      input{
        margin-right: 2px;
      }
    }

    .dwt-save-scan {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      margin-top: 15px;
      .btn-saveToLocal {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 48%;
        height: 30px;
        font-size: 14px;
        color: white;
        border: none;
        border-radius: 5px;
        background-color: rgb(80, 168, 225);
      }
      .btn-saveToLocal:hover {
        background-color: #61c2ec;
        cursor: pointer;
      }
      .btn-uploadToServer {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 48%;
        height: 30px;
        border: none;
        border-radius: 5px;
        font-size: 14px;
        color: white;
        background-color: rgb(80, 168, 225);
      }
      .btn-uploadToServer:hover {
        background-color: #61c2ec;
        cursor: pointer;
      }
    }
  }

  .btn-disabled {
    background-color: #aaaaaa !important;
  }
</style>
