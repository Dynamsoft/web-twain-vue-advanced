<script lang="jsx">
import { ref, defineComponent, watch, render, onMounted, inject, onUpdated} from 'vue';

export default defineComponent({
  props: {
    messages: Array,
    bNoScroll: Boolean,
    handleEvent: Function
  },
  setup(props) {

    const DWTOutPut_message = ref(null);

    onUpdated(
      () => {
        if(props.bNoScroll)
          DWTOutPut_message.value.scrollTop = 0;
        else
          DWTOutPut_message.value.scrollTop = DWTOutPut_message.value.scrollHeight;
      }
    )

    return () => (
      <>
        <div class="dwt-output">
          <span>Message:(Double click to clear!)</span>
          <div ref={ DWTOutPut_message } 
               class="message" 
               ondblclick={ () => props.handleEvent("doubleClick") }>
            <ul>
                {
                  props.messages.map((oneMsg) =>
                      <li key={ oneMsg.time + "_" + Math.floor(Math.random(1) * 10000000) } 
                          className={ oneMsg.type }>{ oneMsg.text }</li>
                  )
                }
            </ul>
          </div>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
.dwt-output {
    margin-top: 5px;
    padding: 5px 0 0;
    font-size: 14px;
    text-align: left;
    color: #444;
    user-select: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    -khtml-user-select: none;
    -ms-user-select: none;
}

.dwt-output .message {
    width: 638px;
    height: 160px;
    padding: 0px 5px;
    margin-top: 3px;
    font-size: 12px;
    line-height: 20px;
    background: #fff;
    border: solid 1px #ccc;
    overflow-y: auto;
    word-wrap: break-word;
    word-break: break-all;
}

.dwt-output .message .info {
    color: black;
}

.dwt-output .message .error {
    color: red;
}

.dwt-output .message .important {
    color: #cE5E04;
}
</style>
