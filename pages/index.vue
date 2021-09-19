<template>
	<div>
    	<p></p>
    	<canvas id="qrcode" ref="qrcode"></canvas>
	</div>
</template>

<script lang="ts">
import QRCode from 'qrcode';
// import FingerprintJS from '@fingerprintjs/fingerprintjs'

export default {
    async asyncData({ $axios}) {
        const test = await $axios.$get('https://api.ipify.org/')
        return { test }
    },
    mounted() {
        const canvas = this.$refs.qrcode as any;
        QRCode.toCanvas(canvas, 'supersecretkey', (err) => {
            if (err) console.error(err);
            console.log('rendered qrcode')
        });
    }
}

/*
import Vue from 'vue'

if (typeof(window) !== 'undefined') {
  const fpPromise = FingerprintJS.load();
  (async()=>{
      const fp = await fpPromise;
      const result = await fp.get();

      console.log(result);
  })();
}

export default Vue.extend({})
*/
</script>
