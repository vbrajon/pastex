<script setup lang="ts">
// import { appWindow } from "@tauri-apps/api/window"
// import { register } from "@tauri-apps/api/globalShortcut"
import { readText, writeText } from "@tauri-apps/api/clipboard"
import { reactive } from "vue"
const state: {
  history: string[]
  selection: number
  prompts: string[]
} = reactive({
  history: [],
  selection: 0,
  prompts: [""],
})
setInterval(async () => {
  const text = await readText()
  if (!text || text === state.history[0]) return
  state.history = [text].concat(state.history).filter((v, i, a) => a.indexOf(v) === i)
})
async function swap() {
  await writeText(state.history[state.selection])
  remove()
  state.selection = 0
}
async function remove() {
  if (state.history.length === 1) {
    await writeText("")
    state.history = []
    return
  }
  state.history.splice(state.selection, 1)
  state.selection = Math.min(state.history.length - 1, state.selection)
}
// prettier-ignore
async function exec() {
  let s, e, x
  s = e = x = state.history[state.selection]
  try { e = eval(`(${x})`) } catch (e) {}
  try { x = JSON.parse(x) } catch (e) {}
  try { x = eval(state.prompts[0]) } catch(e) {}
  try { if (x instanceof Function) x = x() } catch(e) {}
  if (typeof x !== "string") x = JSON.stringify(x, null, 2)
  if (x !== s) await writeText(x)
  state.prompts = [""].concat(state.prompts)
}
addEventListener("keydown", async (e) => {
  console.log(e.key)
  // if (e.key === "Escape") appWindow.hide()
  if (document.activeElement !== document.body) {
    if (e.key === "ArrowUp") state.selection = Math.max(0, state.selection - 1)
    if (e.key === "ArrowDown") state.selection = Math.min(state.history.length - 1, state.selection + 1)
    return
  }
  if (e.key === "Enter") swap()
  if (e.key === "Backspace") remove()
  if (e.key === "ArrowUp") state.selection = Math.max(0, state.selection - 1)
  if (e.key === "ArrowDown") state.selection = Math.min(state.history.length - 1, state.selection + 1)
})
// if (e.key === "Escape") appWindow.hide()
</script>

<template>
  <!-- <img src="/logo.svg" alt="PasteX logo" /> -->
  <input type="text" v-model="state.prompts[0]" @change="exec" autofocus />
  <ul>
    <li :style="{ background: state.selection === i ? 'hotpink' : 'white' }" @click="state.selection = i" v-for="(item, i) in state.history" :key="i">
      {{ item }}
    </li>
  </ul>
</template>
