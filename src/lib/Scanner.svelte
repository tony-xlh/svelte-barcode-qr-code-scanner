<script>
  import { onMount,onDestroy,createEventDispatcher } from 'svelte';
  import { CameraEnhancer } from 'dynamsoft-camera-enhancer';
  import { BarcodeReader } from "dynamsoft-javascript-barcode";

  BarcodeReader.engineResourcePath = "https://cdn.jsdelivr.net/npm/dynamsoft-javascript-barcode@9.3.0/dist/";

	const dispatch = createEventDispatcher();

  let enhancer;
  let reader;
  let cameraContainer;
  let interval;
  let decoding = false;
  export let isActive = false;

  onMount(() => {
    console.log("mounted");
    init();
	});

  onDestroy(()=>{
    stopDecoding();
    enhancer?.dispose(true);
    reader?.destroyContext();
    enhancer = null;
    reader = null;
  });

  async function init(){
    if (!enhancer) {
      BarcodeReader.license = "DLS2eyJoYW5kc2hha2VDb2RlIjoiMjAwMDAxLTE2NDk4Mjk3OTI2MzUiLCJvcmdhbml6YXRpb25JRCI6IjIwMDAwMSIsInNlc3Npb25QYXNzd29yZCI6IndTcGR6Vm05WDJrcEQ5YUoifQ==";
      enhancer = await CameraEnhancer.createInstance();
      reader = await BarcodeReader.createInstance();
      await enhancer.setUIElement(cameraContainer);
    }
    dispatch('initialization', true);
  }

  async function isActiveUpdated(newValue) {
    if (enhancer) {
      if (newValue == true) {
        await enhancer.open(true);
        startDecoding();
      }else{
        await enhancer.close(true);
        stopDecoding();
      }
    }
  }
  $: isActiveUpdated(isActive);

  function startDecoding() {
    if (interval) {
      clearInterval(interval);
    }
    decoding = false;
    interval = setInterval(captureAndDecode,100);
  }

  function stopDecoding(){
    if (interval) {
      clearInterval(interval);
    }
    decoding = false;
  }

  async function captureAndDecode() {
    if (!enhancer || !reader) {
      return
    }
    if (enhancer.isOpen() === false) {
      return;
    }
    if (decoding == true) {
      return;
    }
    let frame = enhancer.getFrame();
    if (frame) {
      decoding = true;
      let results = await reader.decode(frame);
      decoding = false;
      dispatch('barcodeResults',results);
    }
  }

</script>

<div class="container" bind:this={cameraContainer} style="display:none;">
  <svg class="dce-bg-loading" viewBox="0 0 1792 1792"><path d="M1760 896q0 176-68.5 336t-184 275.5-275.5 184-336 68.5-336-68.5-275.5-184-184-275.5-68.5-336q0-213 97-398.5t265-305.5 374-151v228q-221 45-366.5 221t-145.5 406q0 130 51 248.5t136.5 204 204 136.5 248.5 51 248.5-51 204-136.5 136.5-204 51-248.5q0-230-145.5-406t-366.5-221v-228q206 31 374 151t265 305.5 97 398.5z"/></svg>
  <svg class="dce-bg-camera" viewBox="0 0 2048 1792"><path d="M1024 672q119 0 203.5 84.5t84.5 203.5-84.5 203.5-203.5 84.5-203.5-84.5-84.5-203.5 84.5-203.5 203.5-84.5zm704-416q106 0 181 75t75 181v896q0 106-75 181t-181 75h-1408q-106 0-181-75t-75-181v-896q0-106 75-181t181-75h224l51-136q19-49 69.5-84.5t103.5-35.5h512q53 0 103.5 35.5t69.5 84.5l51 136h224zm-704 1152q185 0 316.5-131.5t131.5-316.5-131.5-316.5-316.5-131.5-316.5 131.5-131.5 316.5 131.5 316.5 316.5 131.5z"/></svg>
  <div class="dce-video-container"></div>
  <div class="dce-scanarea">
      <div class="dce-scanlight"></div>
  </div>
  <div class="sel-container">
      <select class="dce-sel-camera"></select>
      <select class="dce-sel-resolution"></select>
  </div>
</div>

<style>
@keyframes dce-rotate{from{transform:rotate(0turn);}to{transform:rotate(1turn);}}
@keyframes dce-scanlight{from{top:0;}to{top:97%;}}
.container{width:100%;height:100%;min-width:100px;min-height:100px;background:#eee;position:absolute;left:0;top:0;}
.dce-bg-loading{animation:1s linear infinite dce-rotate;width:40%;height:40%;position:absolute;margin:auto;left:0;top:0;right:0;bottom:0;fill:#aaa;}
.dce-bg-camera{display:none;width:40%;height:40%;position:absolute;margin:auto;left:0;top:0;right:0;bottom:0;fill:#aaa;}
.dce-video-container{position:absolute;left:0;top:0;width:100%;height:100%;}
.dce-scanarea{position:absolute;left:0;top:0;width:100%;height:100%;pointer-events:none;}
.dce-scanarea .dce-scanlight{display:none;position:absolute;width:100%;height:3%;border-radius:50%;box-shadow:0px 0px 2vw 1px #00e5ff;background:#fff;animation:3s infinite dce-scanlight;user-select:none;}
.sel-container{position: absolute;left: 0;top: 0;}
.sel-container .dce-sel-camera{display:block;}
.sel-container .dce-sel-resolution{display:block;margin-top:5px;}
.sel-container {display:block;margin-top:5px;}
</style>