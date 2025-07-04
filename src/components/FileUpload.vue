<template>
  <div class="mb-8">
    <h2 class="text-xl font-semibold text-gray-800 mb-4 flex items-center">
      <span class="bg-green-100 text-green-600 p-2 rounded-lg mr-3">📤</span>
      Upload Files
    </h2>
    
    <div 
      class="border-2 border-dashed border-gray-300 rounded-xl p-8 text-center transition-all hover:border-green-400 hover:bg-green-50/50"
      @dragover.prevent="dragOver = true"
      @dragleave="dragOver = false"
      @drop.prevent="handleDrop"
      :class="{ 'border-green-500 bg-green-50/50': dragOver }"
    >
      <div class="max-w-md mx-auto">
        <div class="text-4xl mb-3" :class="{ 'text-green-500': dragOver, 'text-gray-400': !dragOver }">
          <span v-if="dragOver">⬆️</span>
          <span v-else>📁</span>
        </div>
        <p class="text-gray-700 font-medium mb-1">
          <span v-if="dragOver">Drop your file here</span>
          <span v-else>Drag & drop files here</span>
        </p>
        <p class="text-sm text-gray-500 mb-4">or</p>
        
        <label class="cursor-pointer">
          <input 
            type="file" 
            @change="uploadFile" 
            class="hidden" 
          />
          <span class="inline-flex items-center px-5 py-3 bg-gradient-to-r from-green-600 to-blue-600 hover:from-green-700 hover:to-blue-700 text-white rounded-lg font-medium text-sm transition-all shadow-sm hover:shadow-md active:scale-95">
            <span class="mr-2">🔼</span>
            Select File
          </span>
        </label>
        
        <p class="text-xs text-gray-400 mt-4">
          Supports all file types (max 100MB)
        </p>
      </div>
    </div>

    <!-- Upload status -->
    <div v-if="uploadStatus" class="mt-4 p-4 rounded-lg" :class="statusClasses">
      <div class="flex items-center">
        <span v-if="uploadStatus === 'success'" class="text-xl mr-2">✅</span>
        <span v-else class="text-xl mr-2">❌</span>
        <span>{{ statusMessage }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'

const emit = defineEmits(['uploaded'])
const dragOver = ref(false)
const uploadStatus = ref(null) // 'success' or 'error'
const statusMessage = ref('')

const statusClasses = computed(() => {
  return {
    'bg-green-100 text-green-800': uploadStatus.value === 'success',
    'bg-red-100 text-red-800': uploadStatus.value === 'error'
  }
})

const handleDrop = (e) => {
  dragOver.value = false
  const file = e.dataTransfer.files[0]
  if (file) {
    processUpload(file)
  }
}

const uploadFile = (e) => {
  const file = e.target.files[0]
  if (file) {
    processUpload(file)
  }
}

const processUpload = async (file) => {
  // Check file size (example: 100MB limit)
  if (file.size > 100 * 1024 * 1024) {
    showStatus('error', 'File size exceeds 100MB limit')
    return
  }

  const formData = new FormData()
  formData.append('file', file)

  try {
    await axios.post('http://localhost:3000/api/files/upload', formData)
    showStatus('success', 'File uploaded successfully!')
    emit('uploaded')
  } catch (err) {
    console.error(err)
    showStatus('error', 'Upload failed. Please try again.')
  } finally {
    // Reset the input to allow uploading the same file again
    document.querySelector('input[type="file"]').value = ''
  }
}

const showStatus = (status, message) => {
  uploadStatus.value = status
  statusMessage.value = message
  
  // Auto-hide status after 5 seconds
  setTimeout(() => {
    uploadStatus.value = null
  }, 5000)
}
</script>

<style scoped>
/* Smooth transitions */
label, div[class*="border-"] {
  transition: all 0.2s ease;
}
</style>