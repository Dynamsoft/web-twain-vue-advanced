<script lang="jsx">
import { defineComponent, onUpdated, ref, watch, reactive, onMounted } from 'vue';

export default defineComponent({
  props : {
    targetObject : String,
    valuePacks : Array,
    current : String,
    handleValuePicking : Function
  },
  
  setup(props) {
    
    const valuesList = ref(null);
    const current = ref(props.current);
    const currentValues = reactive([]);

    const changeScrollTop = () => {
        let _valueLIs = valuesList.value.children, _scrollTop = 0, _height = 0;
        if (_valueLIs && _valueLIs.length > 0) {
            for (let i = 0; i < currentValues.length; i++) {
                if (currentValues[i].checked) {
                    _scrollTop = _valueLIs[i].offsetTop;
                    _height = _valueLIs[i].offsetHeight;
                }
            }
        }
        if (valuesList.value.scrollTop - _scrollTop > 0)
            valuesList.value.scrollTop = _scrollTop;
        else if (valuesList.value.scrollTop + valuesList.value.offsetHeight < _scrollTop + _height) {
            valuesList.value.scrollTop = _scrollTop + _height - valuesList.value.offsetHeight;
        }
    }

    const handlePackChange = (event) => {
        let packValue = event.target.getAttribute("value");
        for (let i = 0; i < props.valuePacks.length; i++) {
            if (props.valuePacks[i].value === packValue) {
                currentValues.length = 0;
                currentValues.push(...props.valuePacks[i].items);
                current.value = props.valuePacks[i].name;
                break;
            }
        }
    }

    const handleValuePicked = (event) => {
        if (event.keyCode && event.keyCode !== 32) return;
        if (event.keyCode && event.keyCode === 32) event.preventDefault();
        let value = event.target.getAttribute("data-info");
        let old_Values = [...currentValues];
        for (let i = 0; i < old_Values.length; i++) {
            if (old_Values[i].value === value)
                old_Values[i].checked = true;
            else
                old_Values[i].checked = false;
        }
        currentValues.length = 0;
        currentValues.push(...old_Values);
        props.handleValuePicking({prop: current.value, value: value});
    }

    watch(
        () => props.targetObject,
        (newTargetObject,prevTargetObject) => {
            let _valuePacks = props.valuePacks;
            if(currentValues.length === 0 || newTargetObject !== prevTargetObject){
                for(let i = 0; i < _valuePacks.length; i++) {
                    if (_valuePacks[i].name === current.value) {
                        currentValues.length = 0;
                        currentValues.push(..._valuePacks[i].items);
                        return;
                    }
                }
            }
        }
    )

    onMounted(() => {
        if(currentValues.length === 0 ){
            for(let i = 0; i < props.valuePacks.length; i++) {
                if (props.valuePacks[i].name === props.current) {
                    currentValues.length = 0;
                    currentValues.push(...props.valuePacks[i].items);
                    return;
                }
            }
        }
    })
    onUpdated(() => {
        changeScrollTop()
    })

    return () => (
      <>
        <div className="dwt-valuePicker">
            <ul>
                {
                    props.valuePacks.map((topItem, _key) =>
                        <li 
                            className={ (topItem.name === current.value) ? "current" : "" }
                            value={ topItem.value }
                            key={ Math.floor(Math.random() * 10000000) }
                            onClick={ (event) => handlePackChange(event) }
                        >{ topItem.name }</li>
                    )
                }
            </ul>
            <ul ref={ valuesList }>
                {
                    currentValues.map((values, _key) => (
                        <li className={ values.checked ? "current" : "" }
                            data-info={ values.value }
                            key={ Math.floor(Math.random() * 10000000) }
                            onClick={ (event) => handleValuePicked(event) }
                        >{ values.value }</li>
                    ))
                }
            </ul>
        </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
    .dwt-valuePicker {
        position: relative;
        width: 280px;
        height: 90px;
        margin-top: 10px;
        font-size: 14px;
        border: 1px #ccc solid;
        border-radius: 3px;
        box-sizing: border-box;
    }

    .dwt-valuePicker ul {
        position: relative;
        float: left;
        width: 48%;
        height: 100%;
        text-align: center;
        overflow-y: scroll;
        overflow-x: hidden;
        cursor: pointer;
    }

    .dwt-valuePicker ul:first-of-type {
        margin-right: 3%;
    }
    .dwt-valuePicker ul li {
        margin: 3px 10px;
        padding: 3px;
        background-color: #fff;
        border: 1px #ddd solid;
        border-radius: 3px;
        overflow: hidden;
    }

    .dwt-valuePicker ul li.current {
        color: white;
        background-color: #61c2ec;
        border: 1px #fff solid;
    }

</style>
