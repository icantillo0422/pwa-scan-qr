<template>
  <qrcode-stream 
    @init="onInit"
    @detect="onDetect"
    :paused="paused"
    :track="paintOutline"
    class="flex max-w-[450px] w-full h-[270px] md:h-[340px] aspect-[450/340] relative overflow-clip rounded-3xl"
  />
</template>

<script setup lang="ts">
  import { ref } from "vue";
  import { QrcodeStream } from "vue-qrcode-reader";

  const paused = ref<boolean>(false);
  const isValid = ref<boolean>();
  const result = ref<string>();

  const emit = defineEmits(['resolve'])

  const onDetect =  async([ firstDetectedCode ]: [any]) => {
    result.value = firstDetectedCode.rawValue
    paused.value = true

    // pretend it's taking really long
    await timeout(2000)
    isValid.value = Boolean(result.value);

    emit("resolve", {
      error: !isValid.value,
      data: result.value
    });

    // some more delay, so users have time to read the message
    paused.value = false
    result.value = undefined
    isValid.value = undefined
  }

  const paintOutline = (detectedCodes: any, ctx: any) => {
    for (const detectedCode of detectedCodes) {
      const [firstPoint, ...otherPoints] = detectedCode.cornerPoints

      ctx.strokeStyle = 'red'

      ctx.beginPath()
      ctx.moveTo(firstPoint.x, firstPoint.y)
      for (const { x, y } of otherPoints) {
        ctx.lineTo(x, y)
      }
      ctx.lineTo(firstPoint.x, firstPoint.y)
      ctx.closePath()
      ctx.stroke()
    }
  }

  const timeout = (ms: number) => new Promise((resolve) => {
    window.setTimeout(resolve, ms)
  });

  const onInit = async (promise: Promise<any>) => {
    try {
      const { capabilities } = await promise;
      console.log(capabilities)
    } catch (error: any) {
      if (error.name === "NotAllowedError") {
        error = "ERROR: you need to grant camera access permisson";
      } else if (error.name === "NotFoundError") {
        error = "ERROR: no camera on this device. Try upload option.";
      } else if (error.name === "NotSupportedError") {
        error = "ERROR: secure context required (HTTPS, localhost)";
      } else if (error.name === "NotReadableError") {
        error = "ERROR: is the camera already in use?";
      } else if (error.name === "OverconstrainedError") {
        error = "ERROR: installed cameras are not suitable";
      } else if (error.name === "StreamApiNotSupportedError") {
        error = "ERROR: Stream API is not supported in this browser";
      }
    }
  }
</script>

<style>

  .qrcode-stream-camera {
    @apply absolute top-0 left-0 max-w-[450px] w-full h-[270px] md:h-[340px] aspect-[450/340]
  }

  .qrcode-stream-overlay {
    @apply absolute top-0 left-0 max-w-[450px] w-full h-[270px] md:h-[340px] aspect-[450/340]
  }
</style>