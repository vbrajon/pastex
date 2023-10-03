<script setup lang="ts">
import { Command } from "@tauri-apps/api/shell"
// import { appWindow } from "@tauri-apps/api/window"
// import { register } from "@tauri-apps/api/globalShortcut"
import { readText, writeText } from "@tauri-apps/api/clipboard"
import { reactive } from "vue"
const state: {
  history: string[]
  h: number
  prompts: string[]
  p: number
} = reactive({
  history: [],
  h: 0,
  prompts: [""],
  p: 0,
})
setInterval(async () => {
  const text = await readText()
  if (!text || text === state.history[0]) return
  state.history = [text].concat(state.history).filter((v, i, a) => a.indexOf(v) === i)
})
async function swap() {
  await writeText(state.history[state.h])
  remove()
  state.h = 0
}
async function remove() {
  if (state.history.length === 1) {
    await writeText("")
    state.history = []
    return
  }
  state.history.splice(state.h, 1)
  state.h = Math.min(state.history.length - 1, state.h)
}
// prettier-ignore
async function exec($event) {
  state.prompts[state.p] = $event.target.value
  if (/^ls/.test(state.prompts[state.p])) {
    const [cmd, ...args] = state.prompts[state.p].split(" ")
    console.log(cmd, args)
    const output = await new Command(cmd, args).execute()
    await writeText(output.stdout)
    state.prompts = [""].concat(state.prompts)
    return
  }
  let s, e, x
  s = e = x = state.history[state.h]
  try { e = eval(`(${x})`) } catch (e) {}
  try { x = JSON.parse(x) } catch (e) {}
  try { x = eval(state.prompts[state.p]) } catch(e) {}
  try { if (x instanceof Function) x = x() } catch(e) {}
  if (typeof x !== "string") x = JSON.stringify(x, null, 2)
  if (x !== s) await writeText(x)
  state.prompts = [""].concat(state.prompts)
}
addEventListener("keydown", async (e) => {
  console.log(e.key)
  // if (e.key === "Escape") appWindow.hide()
  if (document.activeElement === document.body) {
    if (e.key === "Enter") swap()
    if (e.key === "Backspace") remove()
  }
  if (e.key === "ArrowUp") state.p = Math.min(state.prompts.length - 1, state.p + 1)
  if (e.key === "ArrowDown") state.p = Math.max(0, state.p - 1)
  if (e.key === "ArrowLeft") state.h = Math.max(0, state.h - 1)
  if (e.key === "ArrowRight") state.h = Math.min(state.history.length - 1, state.h + 1)
})
// if (e.key === "Escape") appWindow.hide()
</script>

<template>
  <!-- <img src="/logo.svg" alt="PasteX logo" /> -->
  <input type="text" autofocus :value="state.prompts[state.p]" @change="exec" />
  <div style="overflow: auto;max-width: 80vw;">
    <div style="display: flex;gap: 20px;min-width: fit-content;">
      <div :style="['width: 200px;height: 200px;', { background: state.h === i ? 'rgb(0,0,0,0.25)' : 'rgb(0,0,0,0.1)' }]" @click="state.h = i" v-for="(item, i) in state.history" :key="i">
        {{ item }}
      </div>
    </div>
  </div>
</template>
