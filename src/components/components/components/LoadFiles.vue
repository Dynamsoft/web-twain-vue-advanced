<script lang="jsx">
import { defineComponent, inject } from 'vue';

export default defineComponent({
  props: {
    dwt: Object
  },
  setup(props) {
    let handleOutPutMessage = inject("handleOutPutMessage", Function);
    let handleException = inject("handleException", Function);

    const loadImagesOrPDFs = () => {
      props.dwt.IfShowFileDialog = true;
      props.dwt.Addon.PDF.SetResolution(200);
      props.dwt.Addon.PDF.SetConvertMode(1);
      props.dwt.LoadImageEx("", 5, () => {
        handleOutPutMessage("Loaded an image successfully!");
      }, (errorCode, errorString) => {
        handleException({
          code: errorCode,
          message: errorString
        })
      })
    }

    return () => (
      <>
        <div class="dwt-load-content">
          <button class="btn-load" onClick={ () => loadImagesOrPDFs() }>
            <span>Load</span>
          </button>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  .dwt-load-content {
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 20px 20px;
      border-bottom: solid 1px #ccc;
      user-select: none;

      .btn-load {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 80%;
        height: 30px;
        font-size: 14px;
        color: white;
        border:none;
        border-radius: 5px;
        background-color: rgb(80, 168, 225);
      }
      
      .btn-load:hover {
        background-color: #61c2ec;
        cursor: pointer;
      }
    }
</style>
