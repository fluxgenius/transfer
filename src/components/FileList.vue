<template>
  <div>
    <h2 class="text-xl font-semibold text-gray-800 mb-4 flex items-center">
      <span class="bg-blue-100 text-blue-600 p-2 rounded-lg mr-3">📁</span>
      Your Files
    </h2>
    
    <div v-if="files.length === 0" class="text-center py-8 bg-gray-50 rounded-xl border border-dashed border-gray-200">
      <div class="text-4xl mb-3 text-gray-300">📂</div>
      <p class="text-gray-500">No files uploaded yet</p>
    </div>

    <ul class="space-y-3">
      <li
        v-for="file in files"
        :key="file._id"
        class="bg-white p-4 rounded-xl border border-gray-100 hover:border-blue-200 transition-all shadow-sm hover:shadow-md"
      >      
        <div class="flex justify-between items-start mb-3">
          <div class="flex-1 min-w-0">
            <p class="font-medium text-gray-800 truncate">{{ file.filename }}</p>
            <p class="text-sm text-gray-500">{{ formatFileSize(file.size) }}</p>
          </div>
          
          <div class="flex space-x-2">
            <a 
              :href="file.cloudinaryUrl" 
              target="_blank" 
              class="px-3 py-1.5 bg-blue-50 hover:bg-blue-100 text-blue-600 rounded-lg text-sm font-medium transition flex items-center"
            >
              <span class="mr-1">⬇️</span> Download
            </a>
            <button 
              @click="deleteFile(file._id)" 
              class="px-3 py-1.5 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg text-sm font-medium transition flex items-center"
            >
              <span class="mr-1">🗑️</span> Delete
            </button>
          </div>
        </div>

        <!-- File Preview Section -->
        <div class="mt-4">
          <div v-if="getFileType(file.cloudinaryUrl) === 'image'" class="bg-gray-50 rounded-lg overflow-hidden border border-gray-200">
            <img
              :src="file.cloudinaryUrl"
              alt="preview"
              class="w-full h-auto max-h-64 object-contain mx-auto"
            />
          </div>

          <div v-else-if="getFileType(file.cloudinaryUrl) === 'pdf'" class="bg-gray-50 rounded-lg overflow-hidden border border-gray-200">
            <iframe
              :src="file.cloudinaryUrl"
              class="w-full h-64"
            ></iframe>
          </div>

          <div v-else class="bg-gray-50 rounded-lg p-4 text-center border border-gray-200">
            <div class="text-3xl mb-2 text-gray-400">🗂</div>
            <p class="text-gray-500 text-sm">No preview available</p>
            <p class="text-xs text-gray-400 mt-1">{{ getFileExtension(file.filename) }} file</p>
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { getFileType } from '../utils/filePreview';
import axios from 'axios';

const props = defineProps({
  files: Array
});

const emit = defineEmits(['deleted']);

const formatFileSize = (bytes) => {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(1)) + ' ' + sizes[i];
};

const getFileExtension = (filename) => {
  return filename.split('.').pop().toUpperCase();
};

const deleteFile = async (id) => {
  if (!confirm('Are you sure you want to delete this file?')) return;
  try {
    await axios.delete(`https://trasferserver.onrender.com/api/files/${id}`);
    emit('deleted');
  } catch (err) {
    alert('Failed to delete file. Please try again.');
  }
};
</script>

<style scoped>
/* Smooth transitions for hover effects */
a, button {
  transition: all 0.2s ease;
}

/* Better iframe styling */
iframe {
  border: none;
}
</style>