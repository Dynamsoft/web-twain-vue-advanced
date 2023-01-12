<script lang="jsx">
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    return () => (
      <>
        <div class="footer">
          <span>© 2003–2023 Dynamsoft. All rights reserved.</span>
        </div>
      </>
    );
  },
});
</script>

<style lang="scss" scoped>
  .footer {
    position: absolute;
    bottom: 0;
    left: 0;
    z-index: -1;
    font-size: 14px;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    min-height: 45px;
    background-color: white;
    span {
      color: black;
    }
  }
</style>
