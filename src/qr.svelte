<script>
export let contents
export let label
let canvas
import QRCode from 'qrcode/build/qrcode.min.js'
import {onMount} from 'svelte'

onMount(()=> {
  QRCode.toCanvas(canvas, contents,(e)=> {
    if (e) {console.log("qr error",e)}
    else { console.log("success")}
  })
})

</script>
<div id="qrholder">
  <p>{label}</p>
</div>
<canvas bind:this={canvas}>
</canvas>
