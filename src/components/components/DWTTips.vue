<script lang="jsx">
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    return () => (
      <>
      <div class="dwt-tips">
      </div>
      </>
    )
  },
})
</script>

<style lang="scss" scoped>
  .dwt-tips {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    height: 162px;
    margin-top: 28px;
    padding: 10px;
    font-size: 12px;
    line-height: 20px;
    text-align: left;
    color: #606060;
    border: 1px solid #ccc;
    background: #e7f2fd;
    box-sizing: border-box;

    a{
      color: #606060;
    }

    a:link, a:visited {
      color: #606060;
      text-decoration: none;
    }

    a:hover {
      color: #fe8e14;
    }
  }
</style>
