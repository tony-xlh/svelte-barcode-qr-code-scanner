<script>
  import Scanner from './lib/Scanner.svelte'
  let isActive = false;
  let initialized = false;
  function startScanning(){
    isActive = true;
  }
  function handleBarcodeResults(event){
    let results = event.detail;
    console.log(results);
    if (results.length>0) {
      isActive = false;
    }
  }
  function handleInitialization(event){
    console.log(event.detail);
    initialized = event.detail;
  }
</script>

<main>
  <div>
    <h1>Svelte Barcode Scanner Demo</h1>
    {#if initialized}
      <button on:click={startScanning}>Start Scanning</button>
    {:else}
      <p>Initializing...</p>
    {/if}
    
    <div>
      <Scanner isActive={isActive} on:barcodeResults={handleBarcodeResults} on:initialization={handleInitialization}></Scanner>
    </div>
  </div>
</main>

<style>
  
</style>
