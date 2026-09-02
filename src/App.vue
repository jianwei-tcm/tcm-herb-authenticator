<script setup lang="ts">
import { onLaunch, onShow, onHide } from "@dcloudio/uni-app";
onLaunch(() => {
  console.log("App Launch");
});
onShow(() => {
  console.log("App Show");
});
onHide(() => {
  console.log("App Hide");
});
</script>
<style>
/* #ifdef H5 */
@media screen and (min-width: 376px) {
  html,
  body,
  #app {
    min-height: 100%;
    background: #f1f0ec;
  }
}
/* #endif */
</style>
