# slidev-addon-excalidraw

show excalidraw in [slidev](https://sli.dev/)

![example](./example-export/1.png)

```md
---
layout: center
---
<div class="flex flex-col items-center">

# slidev-addon-excalidraw

<Excalidraw
  drawFilePath="./example.excalidraw.json"
  class="w-[600px]"
  :darkMode="false"
  :background="false"
/>

</div>
```

## Installation

```bash
pnpm add slidev-addon-excalidraw
```

### Usage

-   Define this addon in `frontmatter`

```yaml
addons:
    - slidev-addon-excalidraw
```

-   or in `package.json`

```json
 "slidev": {
    "addons": [
      "slidev-addon-excalidraw"
    ]
  },
```

## Components

### Excalidraw

> [!NOTE]
> excalidraw file must be in `public`, and drawFilePath must be relative to your [Public Base Path](https://vitejs.dev/guide/build.html#public-base-path).

```vue
<Excalidraw
  drawFilePath="./example.excalidraw.json"
  class="w-[600px]"
  :darkMode="false"
  :background="false"
/>
```

### Options

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `drawFilePath` | `string` | `undefined` | The path to the excalidraw json file. It must be relative to your [Public Base Path](https://vitejs.dev/guide/build.html#public-base-path). |
| `darkMode` | `boolean` | `false` | Whether to use dark mode. |
| `background` | `boolean` | `false` | Whether to show the background. |