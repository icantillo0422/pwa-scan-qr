<template>
  <main class="max-w-screen-2xl w-11/12 mx-auto md:w-full flex flex-col gap-y-4 py-5">
    <h1 class="font-bold text-2xl text-center uppercase">Escanear codigo QR</h1>
    <section class="flex justify-center">
      <QrScan @resolve="onResolve" />
    </section>
    <section v-show="result" class="flex flex-col justify-center text-center gap-y-5">
      <p><strong>Codigo:</strong> {{ result }}</p>
      <button @click="() => {result = ''}" class="w-8/12 md:w-5/12 xl:w-3/12 mx-auto rounded-2xl bg-black text-white py-2">Reiniciar</button>
    </section>
  </main>
</template>

<script setup lang="ts">
  import QrScan from './components/QrScan.vue';
  import { ref } from 'vue';

  const result = ref<string>();

  const onResolve = ({ error, data }: { error: boolean, data: string }) => {
    if (error) {
      return
    }

    result.value = data;
  }
</script>