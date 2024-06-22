<template>
  <div :class="$attrs.class" v-if="svg" v-html="svg"></div>
</template>

<script setup lang="ts">
import { exportToSvg } from '@excalidraw/utils'
import { onMounted, ref, useAttrs } from 'vue'
const attrs = useAttrs()

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

const svg = ref<string | null>(null)

onMounted(() => {
  loadJsonAndExport(props)
})

const loadJsonAndExport = async ({ drawFilePath: path, darkMode = false, background = false }: { drawFilePath: string; darkMode: boolean; background: boolean }) => {
  try {
    // Dynamically import the JSON file using import()
    // const { default: json } = await import(path)
    const url = new URL(path, window.location.origin + import.meta.env.BASE_URL).href
    const json = await (await fetch(url)).json()

    const svgElement = await exportToSvg({
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
</script>
