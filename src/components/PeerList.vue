<template>
  <div class="mb-8">
    <h2 class="text-xl font-semibold text-gray-800 mb-4 flex items-center">
      <span class="bg-purple-100 text-purple-600 p-2 rounded-lg mr-3">📡</span>
      Available Connections
    </h2>
    
    <div v-if="peers.length === 0" class="bg-gray-50 rounded-xl p-6 text-center border border-dashed border-gray-200">
      <div class="text-4xl mb-3 text-gray-300">🔍</div>
      <p class="text-gray-500">No active connections</p>
      <p class="text-sm text-gray-400 mt-1">Connect with peers using the ID above</p>
    </div>

    <ul class="space-y-3">
      <li
        v-for="peer in peers"
        :key="peer.id"
        class="bg-white p-4 rounded-xl border border-gray-100 hover:border-purple-200 transition-all shadow-sm hover:shadow-md"
      >
        <div class="flex items-start justify-between gap-4">
          <div class="flex-1 min-w-0">
            <div class="flex items-center mb-1">
              <span class="font-medium text-gray-800 truncate">{{ peer.name }}</span>
              <span class="ml-2 bg-purple-100 text-purple-800 text-xs px-2 py-0.5 rounded-full">Online</span>
            </div>
            <div class="text-xs text-gray-500 font-mono truncate" :title="peer.id">{{ peer.id }}</div>
          </div>

          <!-- File upload section -->
          <div class="flex flex-col items-end">
            <label class="relative cursor-pointer">
              <input
                type="file"
                @change="e => handleSendFile(peer.id, e)"
                :disabled="uploading[peer.id]"
                class="hidden"
              />
              <div class="flex items-center space-x-2">
                <span v-if="!uploading[peer.id]" class="px-4 py-2 bg-gradient-to-r from-purple-600 to-blue-600 hover:from-purple-700 hover:to-blue-700 text-white rounded-lg font-medium text-sm transition-all shadow-sm hover:shadow-md active:scale-95 flex items-center">
                  <span class="mr-2">📤</span>
                  Send File
                </span>
                <span v-else class="px-4 py-2 bg-gray-100 text-gray-500 rounded-lg font-medium text-sm flex items-center">
                  <svg
                    class="w-4 h-4 animate-spin text-purple-600 mr-2"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                  >
                    <circle
                      class="opacity-25"
                      cx="12"
                      cy="12"
                      r="10"
                      stroke="currentColor"
                      stroke-width="4"
                    ></circle>
                    <path
                      class="opacity-75"
                      fill="currentColor"
                      d="M4 12a8 8 0 018-8v8H4z"
                    ></path>
                  </svg>
                  Uploading...
                </span>
              </div>
            </label>
            <div v-if="uploading[peer.id]" class="text-xs text-gray-500 mt-1">Preparing transfer...</div>
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const props = defineProps({ peers: Array })
const emit = defineEmits(['send'])

// track uploading state per peer
const uploading = ref({})

const handleSendFile = async (peerId, event) => {
  const file = event.target.files[0]
  if (!file) return

  uploading.value[peerId] = true
  const formData = new FormData()
  formData.append('file', file)

  try {
    // 1️⃣ upload to backend
    const res = await axios.post('https://trasferserver.onrender.com/api/files/upload', formData)
    const fileData = res.data.file

    // 2️⃣ emit real metadata for socket transfer
    emit('send', peerId, {
      filename: fileData.filename,
      cloudinaryUrl: fileData.cloudinaryUrl,
      size: fileData.size
    })
  } catch (err) {
    console.error(err)
    alert('File upload failed. Please try again.')
  } finally {
    // re-enable input
    uploading.value[peerId] = false
    // clear the input so the same file can be selected again if needed
    event.target.value = ''
  }
}
</script>

<style scoped>
/* Smooth transitions for hover effects */
label, button {
  transition: all 0.2s ease;
}

/* Better truncation for long peer IDs */
.truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>