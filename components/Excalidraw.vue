<template>
  <p v-if="loading">Loading Excalidraw...</p>
  <div :class="$attrs.class" v-if="svg" v-html="svg"></div>
</template>

<script setup lang="ts">
import { onMounted, ref, useAttrs } from 'vue'
const attrs = useAttrs()
const loading = ref(false)
const svg = ref<string | null>(null)

interface AppState {
  exportWithDarkMode: boolean;
  exportBackground: boolean;
}

const props = withDefaults(defineProps<{
  drawFilePath: string
  darkMode?: true
  background?: boolean
}>(), {
  darkMode: false,
  background: false,
})


onMounted(() => {
  loading.value = true
  loadScriptsSimultaneously([
    'https://cdn.jsdelivr.net/npm/react@18.2.0/umd/react.production.min.js',
    'https://cdn.jsdelivr.net/npm/react-dom@18.2.0/umd/react-dom.production.min.js',
    'https://cdn.jsdelivr.net/npm/@excalidraw/excalidraw/dist/excalidraw.production.min.js',
  ]).then(() => {
    loadJsonAndExport(props)
  }).finally(() => {
    loading.value = false
  })
})


const loadJsonAndExport = async ({ drawFilePath: path, darkMode = false, background = false }: { drawFilePath: string; darkMode: boolean; background: boolean }) => {
  try {
    const url = new URL(path, window.location.origin + import.meta.env.BASE_URL).href
    const json = await (await fetch(url)).json()

    const svgElement = await ExcalidrawLib.exportToSvg({
      ...json,
      appState: {
        ...(json.appState as any),
        exportWithDarkMode: darkMode,
        exportBackground: background,
      }
    })
    svgElement.style.maxWidth = '100%'
    svgElement.style.height = 'auto'

    svg.value = svgElement.outerHTML
  } catch (error) {
    console.error('Failed to load JSON or export to SVG', error)
  }
}

function loadScript(src) {
  return new Promise((resolve, reject) => {
    // 如果该 src 的 script 已经加载过,直接返回
    if (document.querySelector(`script[src="${src}"]`)) {
      resolve('success')
      return
    }
    const script = document.createElement('script')
    script.src = src
    script.onload = resolve
    script.onerror = reject
    document.head.appendChild(script)
  })
}
function loadScriptsSimultaneously(srcList: string[]) {
  const promises = srcList.map(src => loadScript(src))
  return Promise.all(promises)
}
</script>
