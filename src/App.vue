<script setup lang="ts">
import { appWindow } from "@tauri-apps/api/window"
import { register } from "@tauri-apps/api/globalShortcut"
import { readText, writeText } from "@tauri-apps/api/clipboard"
import { reactive } from "vue"
const history: string[] = reactive([])
setInterval(async () => {
  const text = await readText()
  if (!text || text === history[0]) return
  history.unshift(text)
})
register("CommandOrControl+Shift+V", async () => {
  const visible = await appWindow.isVisible()
  if (!visible) await appWindow.show()
  else await appWindow.hide()
})
</script>

<template>
  <img src="/logo.svg" alt="PasteX logo" />
  <ul>
    <li @click="writeText(item)" v-for="(item, i) in history" :key="i">
      {{ item }}
    </li>
  </ul>
</template>
