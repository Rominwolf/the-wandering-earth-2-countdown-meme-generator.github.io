<template>
  <q-layout view="lHh Lpr lFf">

    <q-page-container>
      <FabricCanvas></FabricCanvas>
    </q-page-container>

    <CanvasConfigs></CanvasConfigs>

  </q-layout>
</template>

<script>
import { defineComponent, ref } from 'vue'
import FabricCanvas from "components/FabricCanvas";
import CanvasConfigs from "components/CanvasConfigs";

export default defineComponent({
  name: 'MainLayout',

  components: {
    CanvasConfigs,
    FabricCanvas,
  },
})
</script>
