<script lang="ts">
  import { Options, Vue } from 'vue-class-component'
  import * as pdfjsLib from 'pdfjs-dist'
  export default class PdfViewer extends Vue {
    onChange(e: Event) {
      const input = e.target as HTMLInputElement
      const files = input.files
      if (files == null || files.length == 0) return
      const file = files[0]
      const reader = new FileReader()
      reader.onload = async () => {
        const arrayBuffer = reader.result as ArrayBuffer
        pdfjsLib.GlobalWorkerOptions.workerSrc =
          'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.7.107/pdf.worker.min.js'
        const loadingTask = pdfjsLib.getDocument(arrayBuffer)
        const pdf = await loadingTask.promise
        const viewerContainer = document.getElementById(
          'pdf-viewer'
        ) as HTMLDivElement

        const totalPage = pdf._pdfInfo.numPages
        for (let pageNum = 1; pageNum <= totalPage; pageNum++) {
          const page = await pdf.getPage(pageNum)
          let scale = 1.5
          let viewport = page.getViewport({ scale: scale })
          let canvas = document.createElement('canvas') as HTMLCanvasElement
          viewerContainer.appendChild(canvas)
          let canvasContext = canvas.getContext('2d')
          if (canvasContext === null) return
          canvas.height = viewport.height
          canvas.width = viewport.width
          await page.render({
            canvasContext,
            viewport,
          }).promise
        }
      }
      reader.readAsArrayBuffer(file)
    }
    mounted() {
      console.log('pdf viewer mounted')
    }
  }
</script>

<template>
  <div class="container">
    PDF VIEWER
    <div class="fileInput">
      <input type="file" @change="onChange" />
    </div>
    <div id="pdf-viewer"></div>
  </div>
</template>
<style scoped></style>
