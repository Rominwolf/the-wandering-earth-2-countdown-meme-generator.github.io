<template>
  <router-view />
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'App'
})
</script>

<style>
@font-face {
  font-family: "Alte DIN 1451 Mittelschrift gepraegt Regular";
  src: url("fonts/Alte DIN 1451 Mittelschrift gepraegt Regular.ttf") format("truetype");;
}

@font-face {
  font-family: "字魂59号-创粗黑";
  src: url("fonts/字魂59号-创粗黑.ttf") format("truetype");;
}

.font-zihun {
  font-family: "字魂59号-创粗黑";
}

body {
  overflow-y: hidden;
}
</style>
