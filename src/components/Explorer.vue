<template>
  <div class="w-screen min-h-screen flex items-center justify-center bg-gray-100">
    <!-- Card container -->
    <div class="bg-white shadow-lg rounded-lg overflow-hidden w-3/4">
      <!-- Sidebar dan Konten Utama diatur dalam Grid -->
      <div class="grid grid-cols-4">
        <!-- Sidebar -->
        <aside class="bg-gray-50 p-4 border-r w-full">
          <div class="mb-4 flex justify-between">
            <button class="text-2xl font-bold py-1 px-4 rounded hover:bg-slate-200" @click="backButton">
              <!-- Ikon Panah Kiri -->
              <span>←</span>
            </button>
            <button class="text-2xl font-bold py-0.5 px-2 rounded-full bg-slate-200 hover:bg-[#e5e7eb]"
              @click="openModal">
              <!-- Ikon Panah Kiri -->
              <span>&#43;</span>
            </button>
          </div>
          <ul>
            <li class="flex items-center py-2 px-4 w-full rounded-lg hover:bg-gray-200 cursor-pointer"
              v-for="(item, index) in this.list_folder_master" :key="index" @click="viewFolderSubM(item.folder_m_id)">
              <span class="text-gray-600 mr-2">📂</span> {{ item.folder_name }}
            </li>
          </ul>
        </aside>

        <!-- Main content dengan tampilan table -->
        <main class="col-span-3 p-6">
          <table class="table-auto w-full text-left">
            <thead>
              <tr class="border-b">
                <th class="py-2 px-4 text-gray-700 font-semibold">Name</th>
                <th class="py-2 px-4 text-gray-700 font-semibold">Date Modified</th>
                <th class="py-2 px-4 text-gray-700 font-semibold">Type</th>
                <th class="py-2 px-4 text-gray-700 font-semibold">Size</th>
              </tr>
            </thead>
            <tbody>
              <tr class="hover:bg-gray-100 cursor-pointer" v-for="(item, index) in list_folder_subs" :key="index"
                @click="viewFolderSubM(undefined, item.id)">
                <td class="py-3 px-4 flex items-center">
                  <span class="text-yellow-500 mr-2" v-if="item.tipe == 'Folder'">📁</span>
                  <span class="text-yellow-500 mr-2" v-else>📄</span>
                  <span class="text-gray-800">{{ item.name }}</span>
                </td>
                <td class="py-3 px-4 text-gray-600">{{ formatDate(item.updatedAt) }}</td>
                <!-- <td class="py-3 px-4 text-gray-600">9/28/2024 10:21 AM</td> -->
                <td class="py-3 px-4 text-gray-600">{{ item.tipe }}</td>
                <td class="py-3 px-4 text-gray-600">-</td>
              </tr>
              <!-- Tambahkan baris tambahan di sini jika diperlukan -->
            </tbody>
          </table>
        </main>
      </div>
    </div>
    <!-- Modal -->
    <div v-if="isModalOpen" class="fixed z-10 inset-0 overflow-y-auto">
      <div class="flex items-center justify-center min-h-screen px-4 py-12">
        <!-- Background Overlay -->
        <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" aria-hidden="true"></div>

        <!-- Modal Content -->
        <div class="bg-white rounded-lg overflow-hidden shadow-xl transform transition-all sm:max-w-lg sm:w-full">
          <div class="px-4 py-5 sm:p-6">
            <h3 class="text-lg leading-6 font-medium text-gray-900">Pilih Opsi</h3>

            <!-- Input Select -->
            <div class="mt-4">
              <div>
                <label for="options" class="block text-sm font-medium text-gray-700">Pilih salah satu:</label>
                <select id="options" v-model="selectedOption"
                  class="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md">
                  <option disabled value="">-- Pilih Opsi --</option>
                  <option value="folder">Folder</option>
                  <option value="file">File</option>
                </select>
              </div>
              <div class="mt-2">
                <p class="text-sm text-gray-500">Kamu memilih: {{ selectedOption || 'Belum memilih' }}</p>
              </div>
              <div class="mt-2" v-if="selectedOption == 'folder'">
                <label for="folder-name" class="block text-sm font-medium text-gray-700">Nama Folder</label>
                <input type="text" id="folder-name" v-model="data_folder.folder_name"
                  class="mt-1 block w-full pl-3 pr-10 py-2 text-base border border-gray-500 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm rounded-md"
                  placeholder="Masukkan nama folder">
              </div>
              <div class="mt-2" v-if="selectedOption == 'file'">
                <label for="inputFile" class="block text-sm font-medium text-gray-700">Unggah File</label>
                <input type="file" id="inputFile" @change="handleFileUpload"
                  class="mt-1 block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100">
              </div>
            </div>
          </div>

          <!-- Tombol untuk menutup modal -->
          <div class="px-4 py-3 bg-gray-50 sm:px-6 sm:flex sm:flex-row-reverse">
            <button @click="createFolder"
              class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-green-500 text-base font-medium text-white hover:bg-green-700 sm:ml-3 sm:w-auto sm:text-sm">
              Submit
            </button>
            <button @click="closeModal"
              class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-blue-500 text-base font-medium text-white hover:bg-blue-700 sm:ml-3 sm:w-auto sm:text-sm">
              Tutup
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'Explorer',
  data() {
    return {
      list_folder_master: [],
      list_folder_subs: [],
      env: import.meta.env,
      folder_s_id: null,
      folder_m_id: null,
      isModalOpen: false,
      selectedOption: 'folder',
      data_folder: {},
      data_file: {
        selectedFile: null
      }
    }
  },
  mounted() {
    // console.log(process.env.VUE_API_URL);

    axios.get(`${this.env.VITE_API_URL}/api/folder-master`)
      .then(res => {
        this.list_folder_master = res.data.data;
      })
      .catch(err => {
        console.error(err);
      })
  },
  methods: {
    viewFolderSubM(folder_m_id, folder_s_id) {
      let url = `/api/folder-subs/show-master/${folder_m_id}`;
      this.folder_m_id = folder_m_id;
      this.folder_s_id = null
      if (folder_s_id) {
        this.folder_s_id = folder_s_id;
        url = `/api/folder-subs/show-parent/${folder_s_id}`;
      }
      axios.get(`${this.env.VITE_API_URL}${url}`)
        .then(res => {
          this.list_folder_subs = res.data.data;
        })
        .catch(err => {
          console.error(err);
        })
    },
    handleFileUpload(event) {
      this.selectedFile = event.target.files[0]; // Simpan file yang dipilih
      console.log("File yang dipilih:", this.selectedFile);
    },
    createFolder() {
      if (this.selectedOption == 'folder') {
        if (this.folder_s_id) {
          this.data_folder.parent_id = this.folder_s_id;
          delete this.data_folder.folder_m_id
        }
        if (this.folder_m_id) {
          this.data_folder.folder_m_id = this.folder_m_id;
          delete this.data_folder.parent_id
        }
        axios.post(`${this.env.VITE_API_URL}/api/folder-subs/`, this.data_folder)
          .then(res => {
            this.viewFolderSubM(this.folder_m_id, this.folder_s_id)
            this.closeModal()
          })
          .catch(err => {
            console.log(err);
          })
      } else {
        const formData = new FormData();
        if (this.folder_s_id) {
          formData.append('folder_s_id', this.folder_s_id);
        }
        if (this.folder_m_id) {
          formData.append('folder_m_id', this.folder_m_id);
        }
        formData.append('files', this.selectedFile)
        axios.post(`${this.env.VITE_API_URL}/api/files/upload`, formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
          },
        })
          .then(res => {
            this.viewFolderSubM(this.folder_m_id, this.folder_s_id)
            this.closeModal()
          })
          .catch(err => {
            console.log(err);
          })
      }
    },
    backButton() {
      console.log(this.folder_s_id);

      if (this.folder_s_id == null) {
        return
      }
      axios.get(`${this.env.VITE_API_URL}/api/folder-subs/show-back/${this.folder_s_id}`)
        .then(res => {
          this.list_folder_subs = res.data.data;
          this.folder_s_id = this.list_folder_subs[0].parent_id;
        })
        .catch(err => {
          console.error(err);
        })
    },
    openModal() {
      if (!this.folder_s_id && !this.folder_m_id) {
        return
      }
      this.isModalOpen = true; // Buka modal
    },
    closeModal() {
      this.isModalOpen = false; // Tutup modal
    },
    formatDate(dateString) {
      const options = {
        month: 'numeric',
        day: 'numeric',
        year: 'numeric',
        hour: 'numeric',
        minute: 'numeric',
        hour12: true
      };

      const date = new Date(dateString);
      return date.toLocaleString('en-US', options);
    }
  }
};
</script>

<style scoped>
/* Custom styles (if needed) */
.transition-opacity {
  transition: opacity 0.3s ease;
}
</style>
