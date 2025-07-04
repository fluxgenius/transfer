<template>
  <div class="min-h-screen bg-gradient-to-br from-gray-50 to-gray-100 p-4 md:p-8">
    <div class="max-w-6xl mx-auto">
      <!-- Header -->
      <header class="mb-8">
        <h1 class="text-3xl md:text-4xl font-bold text-gray-800 flex items-center">
          <span class="bg-blue-600 text-white p-3 rounded-lg mr-3 shadow-md">📁</span>
          <span>FileDrop</span>
        </h1>
        <p class="text-gray-500 mt-2">Secure peer-to-peer file transfers</p>
      </header>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <!-- Left Column (Connection & Peers) -->
        <div class="lg:col-span-1 space-y-6">
          <!-- Connection Card -->
          <div class="bg-white rounded-xl shadow-sm p-5 border border-gray-100">
            <h2 class="text-lg font-semibold text-gray-800 mb-4 flex items-center">
              <span class="text-blue-600 mr-2">🔗</span>
              Connect Manually
            </h2>
            <div class="flex space-x-2">
              <input
                v-model="manualId"
                placeholder="Enter peer ID"
                class="flex-1 border border-gray-200 p-3 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition"
              />
              <button
                @click="() => {
                  if (manualId && !peers.find(p => p.id === manualId)) {
                    peers.push({ id: manualId, name: manualId })
                    manualId = ''
                  }
                }"
                class="px-5 py-3 bg-blue-600 hover:bg-blue-700 text-white rounded-lg font-medium transition-all shadow-md hover:shadow-lg active:scale-95"
              >
                Connect
              </button>
            </div>
          </div>

          <!-- Peer List -->
          <PeerList :peers="peers" @send="sendToPeer" class="bg-white rounded-xl shadow-sm p-5 border border-gray-100" />

          <!-- File Upload -->
          <FileUpload @uploaded="fetchFiles" class="bg-white rounded-xl shadow-sm p-5 border border-gray-100" />
        </div>

        <!-- Middle Column (Files) -->
        <div class="lg:col-span-1">
          <FileList :files="files" @deleted="fetchFiles" class="bg-white rounded-xl shadow-sm p-5 border border-gray-100 h-full" />
        </div>

        <!-- Right Column (Transfers) -->
        <div class="lg:col-span-1">
          <div class="bg-white rounded-xl shadow-sm p-5 border border-gray-100 h-full">
            <h2 class="text-lg font-semibold text-gray-800 mb-4 flex items-center">
              <span class="text-blue-600 mr-2">⏳</span>
              Active Transfers
            </h2>
            
            <div v-if="transfers.length === 0" class="text-center py-10 text-gray-400">
              <div class="text-4xl mb-3">📤</div>
              <p>No active transfers</p>
            </div>

            <ul class="space-y-3">
              <li
                v-for="t in transfers"
                :key="t.key"
                class="bg-gray-50 p-4 rounded-lg border border-gray-100 hover:border-blue-200 transition"
              >
                <div class="flex justify-between items-center mb-3">
                  <div class="flex items-center">
                    <span class="bg-blue-100 text-blue-600 p-2 rounded-full mr-3">
                      {{ t.from === 'me' ? '📤' : '📥' }}
                    </span>
                    <div>
                      <p class="font-medium text-gray-800 truncate max-w-[180px]">{{ t.filename }}</p>
                      <p class="text-xs text-gray-500">
                        {{ t.from === 'me' ? 'Sending' : 'Receiving' }}
                      </p>
                    </div>
                  </div>
                  <span v-if="t.status === 'uploading'" class="text-sm text-gray-500">
                    Preparing…
                  </span>
                  <span v-else-if="t.status === 'pendingTransfer'" class="text-sm text-gray-500">
                    Waiting…
                  </span>
                  <span v-else class="font-medium text-gray-700">{{ t.progress }}%</span>
                </div>

                <!-- Progress bar / spinner -->
                <div class="mb-4">
                  <div
                    v-if="t.status === 'uploading' || t.status === 'pendingTransfer'"
                    class="flex justify-center"
                  >
                    <svg
                      class="w-6 h-6 animate-spin text-blue-600"
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
                  </div>
                  <div v-else class="w-full h-2 bg-gray-200 rounded-full overflow-hidden">
                    <div
                      class="bg-gradient-to-r from-blue-500 to-blue-600 h-2 rounded-full"
                      :style="{ width: t.progress + '%' }"
                    ></div>
                  </div>
                </div>

                <!-- File preview -->
                <div v-if="getFileType(t.cloudinaryUrl) === 'image'" class="mt-3">
                  <img
                    :src="t.cloudinaryUrl"
                    alt="preview"
                    class="max-w-full rounded-lg shadow-sm border border-gray-200"
                  />
                </div>
                <div v-else-if="getFileType(t.cloudinaryUrl) === 'pdf'" class="mt-3">
                  <iframe
                    :src="t.cloudinaryUrl"
                    class="w-full h-48 rounded-lg border border-gray-200"
                  ></iframe>
                </div>
                <div v-else class="mt-3">
                  <a
                    :href="t.cloudinaryUrl"
                    target="_blank"
                    class="inline-flex items-center text-blue-600 hover:text-blue-800 transition"
                  >
                    <span class="mr-1">🔗</span>
                    Open {{ t.filename }}
                  </a>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import PeerList from '../components/PeerList.vue'
import FileUpload from '../components/FileUpload.vue'
import FileList from '../components/FileList.vue'
import axios from 'axios'
import socket from '../socket'
import { getFileType } from '../utils/filePreview'

const peers     = ref([])
const files     = ref([])
const transfers = ref([])
const manualId  = ref('')

const fetchFiles = async () => {
  const { data } = await axios.get('http://localhost:3000/api/files')
  files.value = data.files
}

// Utility to create unique keys
const makeKey = () => Date.now() + Math.random().toString(36).slice(2)

onMounted(() => {
  fetchFiles()

  socket.on('connect', () => {
    socket.emit('announce', { name: 'Web Client' })
    socket.on('peers', list => {
      peers.value = list.filter(p => p.id !== socket.id)
    })
  })

  // Incoming file — go straight to "pendingTransfer"
  socket.on('incoming-file', ({ from, file }) => {
    transfers.value.push({
      key: makeKey(),
      from,
      filename: file.filename,
      cloudinaryUrl: file.cloudinaryUrl,
      progress: 0,
      status: 'pendingTransfer'
    })
  })

  // Progress update — switch to "inProgress" on first event
  socket.on('transfer-progress', ({ from, percent }) => {
    const tr = transfers.value.find(t => t.from === from && t.status !== 'inProgress')
    if (tr) tr.status = 'inProgress'
    // update matching in-progress item
    const inProg = transfers.value.find(t => t.from === from && t.status === 'inProgress')
    if (inProg) inProg.progress = percent
  })

  // Complete — mark 100% and "complete"
  socket.on('transfer-complete', ({ from, file }) => {
    const tr = transfers.value.find(t => t.from === from && t.status === 'inProgress')
    if (tr) {
      tr.progress = 100
      tr.status = 'complete'
    }
    fetchFiles()
  })
})

// When we send, we kick off a simulated transfer
const sendToPeer = (peerId, file) => {
  const key = makeKey();
  // 1️⃣ Initialize transfer entry in "uploading" → "pendingTransfer"
  transfers.value.push({
    key,
    from: 'me',
    filename: file.filename,
    cloudinaryUrl: file.cloudinaryUrl,
    progress: 0,
    status: 'pendingTransfer'
  });

  // 2️⃣ Tell the peer we're sending
  socket.emit('start-transfer', { targetId: peerId, file });

  // 3️⃣ Simulate progress
  let percent = 0;
  const interval = setInterval(() => {
    percent += 10;
    // Emit to peer
    socket.emit('transfer-progress', { targetId: peerId, percent });

    // Update our own UI
    const myTx = transfers.value.find(t => t.key === key);
    if (myTx) {
      myTx.status = 'inProgress';
      myTx.progress = percent;
    }

    if (percent >= 100) {
      clearInterval(interval);
      // 4️⃣ Finalize
      socket.emit('transfer-complete', { targetId: peerId, file });
      if (myTx) {
        myTx.status = 'complete';
        myTx.progress = 100;
      }
    }
  }, 200);
};
</script>