<template>
  <div class="wrapper">
    <h1>XML → JSON Parser</h1>

    <input type="file" accept=".xml" @change="onFileChange" />

    <p v-if="outputFileName" class="filename">
      Файл для сохранения: <strong>{{ outputFileName }}</strong>
    </p>

    <div v-if="jsonText" class="actions">
      <button @click="downloadJson">
        Скачать JSON
      </button>
    </div>

    <pre v-if="jsonText">{{ jsonText }}</pre>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { XMLParser } from 'fast-xml-parser'
import '../styles/XmlParser.css'

const jsonText = ref('')
const outputFileName = ref('')

const parser = new XMLParser({
  ignoreAttributes: false,
  attributeNamePrefix: '@',
  textNodeName: '#text',
  trimValues: true
})

const onFileChange = (event) => {
  const file = event.target.files[0]
  if (!file) return

  outputFileName.value = file.name.replace(/\.xml$/i, '.json')

  const reader = new FileReader()

  reader.onload = () => {
    try {
      const xml = reader.result
      const json = parser.parse(xml)
      jsonText.value = JSON.stringify(json, null, 2)
    } catch (err) {
      console.error(err)
      alert('Ошибка при разборе XML файла')
    }
  }

  reader.readAsText(file, 'utf-8')
}

const downloadJson = () => {
  const blob = new Blob([jsonText.value], {
    type: 'application/json'
  })

  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')

  a.href = url
  a.download = outputFileName.value || 'result.json'

  document.body.appendChild(a)
  a.click()

  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}
</script>
