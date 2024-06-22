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

const props = defineProps<{
  drawFilePath: string
  darkMode?: boolean
  background?: boolean
}>()

const svg = ref<string | null>(null)

onMounted(() => {
  loadJsonAndExport(props)
})

const loadJsonAndExport = async ({ drawFilePath: path, darkMode = false, background = false }: { drawFilePath: string; darkMode: boolean; background: boolean }) => {
  try {
    // Dynamically import the JSON file using import()
    /* @vite-ignore */
    const { default: json } = await import(path)

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
