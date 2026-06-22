<script setup>
import { ref } from 'vue'

const isDragging = ref(false)
const isUploading = ref(false)
const uploadComplete = ref(false)
const uploadProgress = ref(0)
const fileInput = ref(null)
const showSecret = ref(false)

const fileData = ref({
  link: '',
  secret: ''
})

const processFile = (file) => {
  if (!file) return
  
  isDragging.value = false
  isUploading.value = true
  uploadComplete.value = false
  uploadProgress.value = 0

  const interval = setInterval(() => {
    uploadProgress.value += 10
    if (uploadProgress.value >= 100) {
      clearInterval(interval)
      isUploading.value = false
      uploadComplete.value = true
      
      const randomId = Math.random().toString(36).substring(2, 10)
      const randomSecret = Math.random().toString(36).substring(2, 12).toUpperCase()
      
      fileData.value = {
        link: `https://mein-prototyp.app/download/${randomId}`,
        secret: randomSecret
      }
    }
  }, 200) 
}

const handleDrop = (event) => {
  isDragging.value = false
  const files = event.dataTransfer.files
  if (files.length > 0) {
    processFile(files[0])
  }
}

const handleFileInput = (event) => {
  const files = event.target.files
  if (files.length > 0) {
    processFile(files[0])
  }
}

const copyToClipboard = async (text) => {
  try {
    await navigator.clipboard.writeText(text)
    alert('Erfolgreich kopiert!')
  } catch (err) {
    console.error('Fehler beim Kopieren', err)
  }
}
</script>

<template>
  <div class="min-h-screen bg-gray-50 text-gray-900 font-sans">
    
    <header class="bg-white shadow-sm px-6 py-4 flex justify-between items-center">
      <h1 class="text-xl font-bold text-gray-800">FileShare Prototyp</h1>
      <div class="flex items-center gap-3">
        <span class="text-sm font-medium text-gray-500">Eingeloggt als Admin</span>
        <img 
          src="https://i.pravatar.cc/150?img=11" 
          alt="Profilbild" 
          class="w-10 h-10 rounded-full border-2 border-gray-200" 
        />
      </div>
    </header>

    <main class="max-w-2xl mx-auto mt-16 px-4">
      
      <div
        @dragover.prevent="isDragging = true"
        @dragleave.prevent="isDragging = false"
        @drop.prevent="handleDrop"
        @click="$refs.fileInput.click()"
        :class="[
          'border-2 border-dashed rounded-xl p-16 text-center transition-all duration-200 cursor-pointer',
          isDragging 
            ? 'border-blue-500 bg-blue-50 scale-[1.02]' 
            : 'border-gray-300 bg-white hover:bg-gray-50 hover:border-gray-400'
        ]"
      >
        <input type="file" ref="fileInput" class="hidden" @change="handleFileInput" />
        <div class="text-6xl mb-4 pointer-events-none">☁️</div>
        <h2 class="text-2xl font-semibold mb-2 pointer-events-none">Dateien hier ablegen</h2>
        <p class="text-gray-500 pointer-events-none">oder klicken, um eine Datei von deinem Computer auszuwählen</p>
      </div>

      <div v-if="isUploading" class="mt-8 bg-white p-6 rounded-xl shadow-sm border border-gray-200">
        <div class="flex justify-between text-sm mb-2 text-gray-700">
          <span class="font-medium">Datei wird verarbeitet...</span>
          <span>{{ uploadProgress }}%</span>
        </div>
        <div class="w-full bg-gray-200 rounded-full h-3">
          <div 
            class="bg-blue-500 h-3 rounded-full transition-all duration-300" 
            :style="{ width: uploadProgress + '%' }"
          ></div>
        </div>
      </div>

      <div v-if="uploadComplete" class="mt-8 bg-white p-6 rounded-xl shadow-sm border border-gray-200">
        <h3 class="text-lg font-bold mb-5 text-green-600 flex items-center gap-2">
          <span>✓</span> Upload erfolgreich abgeschlossen
        </h3>
        
        <div class="space-y-5">
          <div>
            <label class="block text-sm font-semibold text-gray-700 mb-1">Teilbarer Link</label>
            <div class="flex gap-2">
              <input 
                type="text" 
                readonly 
                :value="fileData.link" 
                class="flex-1 bg-gray-50 border border-gray-300 text-gray-800 text-sm rounded-lg px-4 py-2.5 focus:outline-none" 
              />
              <button 
                @click="copyToClipboard(fileData.link)" 
                class="bg-gray-800 hover:bg-gray-700 text-white px-5 py-2.5 rounded-lg text-sm font-medium transition-colors"
              >
                Kopieren
              </button>
            </div>
          </div>

          <div>
            <label class="block text-sm font-semibold text-gray-700 mb-1">Geheimes Passwort (Secret)</label>
            <div class="flex gap-2">
              <input 
                :type="showSecret ? 'text' : 'password'" 
                readonly 
                :value="fileData.secret" 
                class="flex-1 bg-gray-50 border border-gray-300 text-gray-800 text-sm rounded-lg px-4 py-2.5 focus:outline-none font-mono" 
              />
              <button 
                @click="showSecret = !showSecret" 
                class="bg-gray-200 hover:bg-gray-300 text-gray-800 px-4 py-2.5 rounded-lg text-sm font-medium transition-colors"
              >
                {{ showSecret ? 'Verbergen' : 'Anzeigen' }}
              </button>
              <button 
                @click="copyToClipboard(fileData.secret)" 
                class="bg-gray-800 hover:bg-gray-700 text-white px-5 py-2.5 rounded-lg text-sm font-medium transition-colors"
              >
                Kopieren
              </button>
            </div>
          </div>
        </div>
      </div>

    </main>
  </div>
</template>