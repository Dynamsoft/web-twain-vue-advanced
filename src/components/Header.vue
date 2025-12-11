<script lang="jsx">

import { defineComponent } from 'vue';
import DWTLogo from '@/assets/icon-dwt.svg';
import logo from '@/assets/logo.png';
import DynamsoftLogo from '@/assets/logo-dynamsoft-white.svg';

export default defineComponent({
  setup() {

    return () => (
      <>
          <div class="App-header">
            <a href="https://www.dynamsoft.com/Products/WebTWAIN_Overview.aspx" target="_blank" style="height:26px">
              <img src={DWTLogo} className="dwt-logo" alt="Dynamic Web TWAIN Logo" />
            </a>
            <div style={{ width: "10px" }}></div>
            <a href="https://vuejs.org/" target="_blank" style="height:35px"><img src={logo} className="App-logo" alt="logo" /></a>
            <div style={{ width: "770px" }}></div>
            <a href="https://www.dynamsoft.com" target="_blank"><img src={DynamsoftLogo} className="ds-logo" alt="Dynamsoft Logo" /></a>
          </div>
      </>
    );
  },
});
</script>

<style lang="scss" scoped>
.App {
  text-align: center;
}

.App-logo {
  height: 4vmin;
  pointer-events: none;
}

.dwt-logo {
  height: 3vmin;
  pointer-events: none;
}

.ds-logo {
  float: right;
  height: 3vmin;
  pointer-events: none;
}

.App-header {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  min-height: 5vh;
  font-size: calc(10px + 2vmin);
  color: white;
  background-color: #282c34;
}

.App-link {
  color: #61dafb;
}

@keyframes App-logo-spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>
