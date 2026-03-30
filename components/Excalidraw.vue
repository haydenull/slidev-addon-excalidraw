<template>
  <p v-if="loading">Loading Excalidraw...</p>
  <p v-else-if="errorMessage">{{ errorMessage }}</p>
  <div v-else-if="svg" :class="$attrs.class" v-html="svg"></div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'

interface ExcalidrawAppState {
  exportWithDarkMode?: boolean
  exportBackground?: boolean
  viewBackgroundColor?: string
  [key: string]: unknown
}

interface ExcalidrawScene {
  elements?: unknown[]
  appState?: ExcalidrawAppState
  files?: Record<string, unknown> | null
}

interface ExcalidrawModule {
  exportToSvg: (scene: {
    elements: readonly unknown[]
    appState: ExcalidrawAppState
    files: Record<string, unknown> | null
  }) => Promise<SVGSVGElement>
}

declare global {
  interface Window {
    EXCALIDRAW_ASSET_PATH?: string | string[]
  }
}

const EXCALIDRAW_VERSION = '0.18.0'
const EXCALIDRAW_MODULE_URL = `https://esm.sh/@excalidraw/excalidraw@${EXCALIDRAW_VERSION}?bundle&exports=exportToSvg`
const EXCALIDRAW_ASSET_PATH = `https://esm.sh/@excalidraw/excalidraw@${EXCALIDRAW_VERSION}/dist/prod/`

let excalidrawModulePromise: Promise<ExcalidrawModule> | null = null

const loading = ref(false)
const errorMessage = ref<string | null>(null)
const svg = ref<string | null>(null)

const props = withDefaults(defineProps<{
  drawFilePath: string
  darkMode?: boolean
  background?: boolean
}>(), {
  darkMode: false,
  background: false,
})

onMounted(async () => {
  loading.value = true
  errorMessage.value = null

  try {
    const { exportToSvg } = await loadExcalidrawModule()
    const drawData = await loadDrawFile(props.drawFilePath)
    const svgElement = await exportToSvg({
      elements: drawData.elements ?? [],
      appState: {
        ...(drawData.appState ?? {}),
        exportWithDarkMode: props.darkMode,
        exportBackground: props.background,
        viewBackgroundColor: drawData.appState?.viewBackgroundColor ?? '#ffffff',
      },
      files: drawData.files ?? null,
    })

    svgElement.style.maxWidth = '100%'
    svgElement.style.height = 'auto'
    svg.value = svgElement.outerHTML
  } catch (error) {
    console.error('Failed to load JSON or export to SVG', error)
    errorMessage.value = 'Failed to render Excalidraw.'
  } finally {
    loading.value = false
  }
})

async function loadExcalidrawModule() {
  if (!window.EXCALIDRAW_ASSET_PATH)
    window.EXCALIDRAW_ASSET_PATH = EXCALIDRAW_ASSET_PATH

  if (!excalidrawModulePromise) {
    excalidrawModulePromise = import(
      /* @vite-ignore */
      EXCALIDRAW_MODULE_URL
    ) as Promise<ExcalidrawModule>

    excalidrawModulePromise = excalidrawModulePromise.catch((error) => {
      excalidrawModulePromise = null
      throw error
    })
  }

  return excalidrawModulePromise
}

async function loadDrawFile(path: string): Promise<ExcalidrawScene> {
  const url = new URL(path, window.location.origin + import.meta.env.BASE_URL).href
  const response = await fetch(url)

  if (!response.ok)
    throw new Error(`Failed to fetch ${url}: ${response.status} ${response.statusText}`)

  return response.json()
}
</script>
