<script lang="jsx">
import { defineComponent, ref} from 'vue';

export default defineComponent({
  props : {
    rangePicker: Object,
    rangePickerTitle: Object,
    handleRangeChange: Function
  },

  setup(props) {
    let offsetX, offsetY, oldX, oldY;
    let rangePickerDiv = ref(null);
    let overlayDIV = ref(null);

    const move = (e) => {
        const parent = rangePickerDiv.value;
        parent.style.left = `${oldX + (e.clientX - offsetX)}px`;
        parent.style.top = `${oldY + (e.clientY - offsetY)}px`;
        oldX = parent.offsetLeft;
        offsetX = e.clientX;
        oldY = parent.offsetTop;
        offsetY = e.clientY;
    }

    const add = (e) => {
        const parent = rangePickerDiv.value;
        oldX = parent.offsetLeft;
        oldY = parent.offsetTop;
        offsetX = e.clientX;
        offsetY = e.clientY;
        parent.addEventListener('mousemove', move);
        overlayDIV.value.addEventListener('mousemove', move);
    }

    const remove = (e) => {
        rangePickerDiv.value.removeEventListener('mousemove', move);
        overlayDIV.value.removeEventListener('mousemove', move);
    }

    
    return () => (
      <>
        <div ref={ overlayDIV } 
             className="overlay" 
             onMouseup={(e) => remove(e)}></div>
        <div ref={ rangePickerDiv } 
             className="range-Picker"
             onMouseup={ (e) => remove(e)}>
            <div className="range-title" onMousedown={ (e) => add(e) }>
                <span className="range-title" >{props.rangePickerTitle.value}</span>
            </div>
            <div className="range-content">
                {props.rangePicker.bMutable
                    ? (<>
                        <span className="range-current" >{props.rangePicker.val} (Default: {props.rangePicker.defaultVal})</span>
                        <span>{props.rangePicker.min}</span>
                        <input type="range"
                            prop={props.rangePickerTitle.value} _type={props.rangePicker.bCamera ? "camera" : "video"}
                            min={props.rangePicker.min} max={props.rangePicker.max}
                            step={props.rangePicker.step} 
                            value={props.rangePicker.val}
                            onChange={(event) => props.handleRangeChange(event)} />
                        <span>{props.rangePicker.max}</span>
                    </>)
                    : (<span className="range-current" >Default and only value (immutable): {props.rangePicker.val}</span>)}
            </div>
            <div className="range-buttons">
                {props.rangePicker.bMutable ? <button prop={props.rangePickerTitle.value}
                    _type={props.rangePicker.bCamera ? "camera" : "video"}
                    _default={props.rangePicker.defaultVal}
                    value="reset-range" onClick={(event) => props.handleRangeChange(event)}>Reset &amp; Close</button> : ""}
                < button value="close-picker" onClick={(event) => props.handleRangeChange(event)}>Close Window</button>
            </div>
        </div>
      </>
    )

  }
})
</script>

<style lang="scss" scoped>
    .range-Picker {
    position: absolute;
    top: 400px;
    left: 620px;
    display: block;
    height: auto;
    width: 400px;
    z-index: 101;
    border: 1px solid #c5c5c5;
    border-radius: 3px;
    background: #ffffff;
    color: #333333;
    outline: 0;
    padding: .2em;
    user-select: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    -khtml-user-select: none;
    -ms-user-select: none;
}

    .range-title {
        border: 1px solid #dddddd;
        background: #e9e9e9;
        color: #333333;
        font-weight: bold;
        touch-action: none;
        cursor: move;
        margin: 0;
        border-radius: 3px;
        padding: .4em 1em;
        position: relative;
        line-height: 26px;
    }

    .range-content {
        text-align: center;
        min-width: 150px;
        width: auto;
        min-height: 24px;
        max-height: none;
        height: auto;
        position: relative;
        border: 0;
        padding: .5em 1em;
        background: none;
        overflow: auto;
        line-height: 26px;
    }

    .range-content span {
        margin: .1em 0;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        color: #333333;
        font-weight: bold;
        font-family: Arial, Helvetica, sans-serif;
        font-size: 1em;
        line-height: 26px;
    }

    .range-content span.range-current {
        float: left;
        width: 100%;
    }

    .range-buttons {
        margin: 0;
        border: solid #dddddd;
        text-align: center;
        border-width: 1px 0 0 0;
        background-image: none;
        margin-top: .5em;
        background: #ffffff;
        color: #333333;
    }

    .range-buttons button {
        margin: .5em .4em .5em 1em;
        cursor: pointer;
        font-size: 1em;
        font-family: Arial, Helvetica, sans-serif;
        border-radius: 3px;
        color: #454545;
        text-decoration: none;
        border: 1px solid #c5c5c5;
        background: #f6f6f6;
        font-weight: normal;
        padding: .4em 1em;
        position: relative;
        line-height: normal;
        margin-right: .1em;
        vertical-align: middle;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
        overflow: visible;
        -webkit-appearance: button;
        writing-mode: horizontal-tb !important;
        -webkit-writing-mode: horizontal-tb !important;
        text-rendering: auto;
        letter-spacing: normal;
        word-spacing: normal;
        text-transform: none;
        text-indent: 0px;
        text-shadow: none;
        display: inline-block;
        text-align: center;
        align-items: flex-start;
    }

    .range-buttons button:hover {
        color: white;
        background-color: #61c2ec;
    }

    .overlay {
        z-index: 100;
        background: #aaaaaa;
        opacity: .3;
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
    }
</style>
