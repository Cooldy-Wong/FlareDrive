<template>
  <div class="main" @dragenter.prevent @dragover.prevent @drop.prevent="onDrop">
    <progress
      v-if="uploadProgress !== null"
      :value="uploadProgress"
      max="100"
    ></progress>
    <UploadPopup
      v-model="showUploadPopup"
      @upload="onUploadClicked"
      @createFolder="createFolder"
    ></UploadPopup>
    <button class="upload-button circle" @click="showUploadPopup = true">
      <img
        style="filter: invert(100%)"
        src="https://cdnjs.cloudflare.com/ajax/libs/material-design-icons/4.0.0/png/file/upload_file/materialicons/36dp/2x/baseline_upload_file_black_36dp.png"
        alt="Upload"
        width="36"
        height="36"
        @contextmenu.prevent
      />
    </button>
    <div class="app-bar">
      <input type="search" v-model="search" aria-label="Search" />
      <div class="menu-button">
        <button class="circle" @click="showMenu = true">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 448 512"
            width="24"
            height="24"
            title="Menu"
            style="display: block; margin: 4px"
          >
            <!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. -->
            <path
              d="M120 256c0 30.9-25.1 56-56 56s-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56zm160 0c0 30.9-25.1 56-56 56s-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56zm104 56c-30.9 0-56-25.1-56-56s25.1-56 56-56s56 25.1 56 56s-25.1 56-56 56z"
            />
          </svg>
        </button>
        <Menu
          v-model="showMenu"
          :items="[{ text: '名稱A-Z' }, { text: '大小↑' } ,{ text: '大小↓' }, { text: '貼上' }]"
          @click="onMenuClick"
        />
      </div>
    </div>
    <ul class="file-list">
      <li v-if="cwd !== ''">
        <div
          tabindex="0"
          class="file-item"
          @click="cwd = cwd.replace(/[^\/]+\/$/, '')"
          @contextmenu.prevent
        >
          <div class="file-icon">
            <img
              src="https://cdnjs.cloudflare.com/ajax/libs/material-design-icons/4.0.0/png/file/folder/materialicons/36dp/2x/baseline_folder_black_36dp.png"
              width="36"
              height="36"
              alt="Folder"
            />
          </div>
          <span class="file-name">..</span>
        </div>
      </li>
      <li v-for="folder in filteredFolders" :key="folder">
        <div
          tabindex="0"
          class="file-item"
          @click="cwd = folder"
          @contextmenu.prevent="
            showContextMenu = true;
            focusedItem = folder;
          "
        >
          <div class="file-icon">
            <img
              src="https://cdnjs.cloudflare.com/ajax/libs/material-design-icons/4.0.0/png/file/folder/materialicons/36dp/2x/baseline_folder_black_36dp.png"
              width="36"
              height="36"
              alt="Folder"
            />
          </div>
          <span
            class="file-name"
            v-text="folder.match(/.*?([^/]*)\/?$/)[1]"
          ></span>
          <div style="margin-right: 10px;margin-left: auto;"
            @click.stop="
              showContextMenu = true;
              focusedItem = folder;
            "
            >
              <svg viewBox="0 0 24 24" style="height: 30px; width: 30px;"><path fill="currentColor" d="M10.5,12A1.5,1.5 0 0,1 12,10.5A1.5,1.5 0 0,1 13.5,12A1.5,1.5 0 0,1 12,13.5A1.5,1.5 0 0,1 10.5,12M10.5,16.5A1.5,1.5 0 0,1 12,15A1.5,1.5 0 0,1 13.5,16.5A1.5,1.5 0 0,1 12,18A1.5,1.5 0 0,1 10.5,16.5M10.5,7.5A1.5,1.5 0 0,1 12,6A1.5,1.5 0 0,1 13.5,7.5A1.5,1.5 0 0,1 12,9A1.5,1.5 0 0,1 10.5,7.5M12,2A10,10 0 0,1 22,12A10,10 0 0,1 12,22A10,10 0 0,1 2,12A10,10 0 0,1 12,2M12,4A8,8 0 0,0 4,12A8,8 0 0,0 12,20A8,8 0 0,0 20,12A8,8 0 0,0 12,4Z"></path></svg>
          </div>
        </div>
      </li>
      <li v-for="file in filteredFiles" :key="file.key">
        <div
          @click="preview(`/raw/${file.key}`)"
          @contextmenu.prevent="
            showContextMenu = true;
            focusedItem = file;
          "
        >
          <div class="file-item">
            <MimeIcon
              :content-type="file.httpMetadata.contentType"
              :thumbnail="
                file.customMetadata.thumbnail
                  ? `/raw/_$flaredrive$/thumbnails/${file.customMetadata.thumbnail}.png`
                  : null
              "
            />
            <div>
              <div class="file-name" v-text="file.key.split('/').pop()"></div>
              <div class="file-attr">
                <span v-text="new Date(file.uploaded).toLocaleString()"></span>
                <span v-text="formatSize(file.size)"></span>
              </div>
            </div>
            <div style="margin-right: 10px;margin-left: auto;"
            @click.stop="
              showContextMenu = true;
              focusedItem = file;
            "
            >
              <svg viewBox="0 0 24 24" style="height: 30px; width: 30px;"><path fill="currentColor" d="M10.5,12A1.5,1.5 0 0,1 12,10.5A1.5,1.5 0 0,1 13.5,12A1.5,1.5 0 0,1 12,13.5A1.5,1.5 0 0,1 10.5,12M10.5,16.5A1.5,1.5 0 0,1 12,15A1.5,1.5 0 0,1 13.5,16.5A1.5,1.5 0 0,1 12,18A1.5,1.5 0 0,1 10.5,16.5M10.5,7.5A1.5,1.5 0 0,1 12,6A1.5,1.5 0 0,1 13.5,7.5A1.5,1.5 0 0,1 12,9A1.5,1.5 0 0,1 10.5,7.5M12,2A10,10 0 0,1 22,12A10,10 0 0,1 12,22A10,10 0 0,1 2,12A10,10 0 0,1 12,2M12,4A8,8 0 0,0 4,12A8,8 0 0,0 12,20A8,8 0 0,0 20,12A8,8 0 0,0 12,4Z"></path></svg>
            </div>
          </div>
        </div>
      </li>
    </ul>
    <div v-if="loading" style="margin-top: 12px; text-align: center">
      <span>載入中...</span>
    </div>
    <div
      v-else-if="!filteredFiles.length && !filteredFolders.length"
      style="margin-top: 12px; text-align: center"
    >
      <span>沒有檔案</span>
    </div>
    <Dialog v-model="showContextMenu">
      <div
        v-text="focusedItem.key || focusedItem"
        class="contextmenu-filename"
        @click.stop.prevent
      ></div>
      <ul v-if="typeof focusedItem === 'string'" class="contextmenu-list">
        <!-- 【新增】1. 在這裡新增下載資料夾按鈕 -->
        <li>
          <button @click="downloadFolder">
            <span>下載資料夾</span>
          </button>
        </li>
        <li>
          <button @click="copyLink(`/?p=${encodeURIComponent(focusedItem)}`)">
            <span>複製鏈接</span>
          </button>
        </li>
        <li>
          <button @click="moveFile(focusedItem + '_$folder$')">
            <span>移動</span>
          </button>
        </li>
        <li>
          <button
            style="color: red"
            @click="removeFile(focusedItem + '_$folder$')"
          >
            <span>刪除</span>
          </button>
        </li>
      </ul>
      <ul v-else class="contextmenu-list">
        <li>
          <button @click="renameFile(focusedItem.key)">
            <span>重新命名</span>
          </button>
        </li>
        <li>
          <a :href="`/raw/${focusedItem.key}`" target="_blank" download>
            <span>下載</span>
          </a>
        </li>
        <li>
          <button @click="clipboard = focusedItem.key">
            <span>複製</span>
          </button>
        </li>
        <li>
          <button @click="moveFile(focusedItem.key)">
            <span>移動</span>
          </button>
        </li>
        <li>
          <button @click="copyLink(`/raw/${focusedItem.key}`)">
            <span>複製鏈接</span>
          </button>
        </li>
        <li>
          <button style="color: red" @click="removeFile(focusedItem.key)">
            <span>刪除</span>
          </button>
        </li>
      </ul>
    </Dialog>
  </div>
</template>

<script>
import {
  generateThumbnail,
  blobDigest,
  multipartUpload,
  SIZE_LIMIT,
} from "/assets/main.mjs";
import Dialog from "./Dialog.vue";
import Menu from "./Menu.vue";
import MimeIcon from "./MimeIcon.vue";
import UploadPopup from "./UploadPopup.vue";

export default {
  data: () => ({
    cwd: new URL(window.location).searchParams.get("p") || "",
    files: [],
    folders: [],
    clipboard: null,
    focusedItem: null,
    loading: false,
    order: null,
    search: "",
    showContextMenu: false,
    showMenu: false,
    showUploadPopup: false,
    uploadProgress: null,
    uploadQueue: [],
  }),

  computed: {
    filteredFiles() {
      let files = this.files;
      if (this.search) {
        files = files.filter((file) =>
          file.key.split("/").pop().includes(this.search)
        );
      }
      return files;
    },

    filteredFolders() {
      let folders = this.folders;
      if (this.search) {
        folders = folders.filter((folder) => folder.includes(this.search));
      }
      return folders;
    },
  },

  methods: {
    copyLink(link) {
      const url = new URL(link, window.location.origin);
      navigator.clipboard.writeText(url.toString());
    },

    async copyPaste(source, target) {
      const uploadUrl = `/api/write/items/${target}`;
      await axios.put(uploadUrl, "", {
        headers: { "x-amz-copy-source": encodeURIComponent(source) },
      });
    },

    async createFolder() {
      try {
        const folderName = window.prompt("請輸入資料夾名稱");
        if (!folderName) return;
        this.showUploadPopup = false;
        const uploadUrl = `/api/write/items/${this.cwd}${folderName}/_$folder$`;
        await axios.put(uploadUrl, "");
        this.fetchFiles();
      } catch (error) {
        fetch("/api/write/")
          .then((value) => {
            if (value.redirected) window.location.href = value.url;
          })
          .catch(() => {});
        console.log(`Create folder failed`);
      }
    },

    // 【新增】2. 在這裡新增下載資料夾功能的函式
    downloadFolder() {
      if (typeof this.focusedItem === 'string') {
        const folderPath = this.focusedItem;
        window.open(`/api/download/folder/${folderPath}`);
      }
      this.showContextMenu = false;
    },

    fetchFiles() {
      this.files = [];
      this.folders = [];
      this.loading = true;
      fetch(`/api/children/${this.cwd}`)
        .then((res) => res.json())
        .then((files) => {
          this.files = files.value;
          if (this.order) {
            this.files.sort((a, b) => {
              if (this.order === "size") {
                return b.size - a.size;
              }
            });
          }
          this.folders = files.folders;
          this.loading = false;
        });
    },

    formatSize(size) {
      const units = ["B", "KB", "MB", "GB", "TB"];
      let i = 0;
      while (size >= 1024) {
        size /= 1024;
        i++;
      }
      return `${size.toFixed(1)} ${units[i]}`;
    },

    onDrop(ev) {
      let files;
      if (ev.dataTransfer.items) {
        files = [...ev.dataTransfer.items]
          .filter((item) => item.kind === "file")
          .map((item) => item.getAsFile());
      } else files = ev.dataTransfer.files;
      this.uploadFiles(files);
    },

    onMenuClick(text) {
      switch (text) {
        case "名稱A-Z":
          this.order = null;
          break;
        case "大小↑":
          this.order = "大小↑";
          break;
        case "大小↓":
          this.order = "大小↓";
          break;
        case "貼上":
          return this.pasteFile();
      }
      this.files.sort((a, b) => {
        if (this.order === "大小↑") {
          return a.size - b.size;
        } else if (this.order === "大小↓") {
          return b.size - a.size;
        } else {
          return a.key.localeCompare(b.key);
        }
      });
    },

    onUploadClicked(fileElement) {
      if (!fileElement.value) return;
      this.uploadFiles(fileElement.files);
      this.showUploadPopup = false;
      fileElement.value = null;
    },

    preview(filePath){
      window.open(filePath);
    },

    async pasteFile() {
      if (!this.clipboard) return;
      let newName = window.prompt("Rename to:");
      if (newName === null) return;
      if (newName === "") newName = this.clipboard.split("/").pop();
      await this.copyPaste(this.clipboard, `${this.cwd}${newName}`);
      this.fetchFiles();
    },

    async processUploadQueue() {
      if (!this.uploadQueue.length) {
        this.fetchFiles();
        this.uploadProgress = null;
        return;
      }

      /** @type File **/
      const { basedir, file } = this.uploadQueue.pop(0);
      let thumbnailDigest = null;

      if (file.type.startsWith("image/") || file.type === "video/mp4") {
        try {
          const thumbnailBlob = await generateThumbnail(file);
          const digestHex = await blobDigest(thumbnailBlob);

          const thumbnailUploadUrl = `/api/write/items/_$flaredrive$/thumbnails/${digestHex}.png`;
          try {
            await axios.put(thumbnailUploadUrl, thumbnailBlob);
            thumbnailDigest = digestHex;
          } catch (error) {
            fetch("/api/write/")
              .then((value) => {
                if (value.redirected) window.location.href = value.url;
              })
              .catch(() => {});
            console.log(`Upload ${digestHex}.png failed`);
          }
        } catch (error) {
          console.log(`Generate thumbnail failed`);
        }
      }

      try {
        const uploadUrl = `/api/write/items/${basedir}${file.name}`;
        const headers = {};
        const onUploadProgress = (progressEvent) => {
          var percentCompleted =
            (progressEvent.loaded * 100) / progressEvent.total;
          this.uploadProgress = percentCompleted;
        };
        if (thumbnailDigest) headers["fd-thumbnail"] = thumbnailDigest;
        if (file.size >= SIZE_LIMIT) {
          await multipartUpload(`${basedir}${file.name}`, file, {
            headers,
            onUploadProgress,
          });
        } else {
          await axios.put(uploadUrl, file, { headers, onUploadProgress });
        }
      } catch (error) {
        fetch("/api/write/")
          .then((value) => {
            if (value.redirected) window.location.href = value.url;
          })
          .catch(() => {});
        console.log(`Upload ${file.name} failed`, error);
      }
      setTimeout(this.processUploadQueue);
    },

    async removeFile(key) {
      if (!window.confirm(`確定要刪除 ${key} 嗎？`)) return;
      await axios.delete(`/api/write/items/${key}`);
      this.fetchFiles();
    },

    async renameFile(key) {
      const newName = window.prompt("重新命名為:");
      if (!newName) return;
      await this.copyPaste(key, `${this.cwd}${newName}`);
      await axios.delete(`/api/write/items/${key}`);
      this.fetchFiles();
    },

    async moveFile(key) {
      // 獲取目前的目錄結構
      const currentPath = this.cwd; // 目前所在目錄
      const allFolders = [...this.folders]; // 所有可用目錄
      
      // 如果不在根目錄，新增返回上級目錄選項
      if (currentPath !== '') {
        const parentPath = currentPath.replace(/[^\/]+\/$/, '');
        if (!allFolders.includes(parentPath) && parentPath !== '') {
          allFolders.unshift(parentPath);
        }
      }
      
      // 新增根目錄選項
      if (!allFolders.includes('')) {
        allFolders.unshift('');
      }
      
      // 構建選擇列表
      const folderOptions = allFolders.map(folder => {
        const displayName = folder === '' ? '根目錄' : 
                          folder === currentPath ? '目前目錄' :
                          folder.replace(/.*\/(?!$)|\//g, '') + '/';
        return {
          display: displayName,
          value: folder
        };
      });
      
      // 建立選擇提示
      const options = folderOptions.map((opt, index) => 
        `${index + 1}. ${opt.display}`
      ).join('\n');
      
      const promptText = `請選擇目標目錄(輸入數字):\n${options}\n`;
      const selection = window.prompt(promptText);
      
      if (!selection) return;
      
      const selectedIndex = parseInt(selection) - 1;
      if (isNaN(selectedIndex) || selectedIndex < 0 || selectedIndex >= folderOptions.length) {
        alert('無效的選擇');
        return;
      }
      
      const targetPath = folderOptions[selectedIndex].value;
      
      // 獲取檔名
      const fileName = key.split('/').pop();
      // 如果是資料夾,需要移除_$folder$後綴
      const finalFileName = fileName.endsWith('_$folder$') ? fileName.slice(0, -9) : fileName;
      
      // 修復：正確處理目標路徑，避免雙斜槓
      const normalizedPath = targetPath === '' ? '' : (targetPath.endsWith('/') ? targetPath : targetPath + '/');
      
      try {
        // 如果是目錄（以_$folder$結尾），則需要移動整個目錄內容
        if (key.endsWith('_$folder$')) {
          // 獲取源目錄的基礎路徑（移除_$folder$後綴）
          const sourceBasePath = key.slice(0, -9);
          // 獲取目標目錄的基礎路徑，修復根目錄的情況
          const targetBasePath = normalizedPath + finalFileName + '/';
          
          // 遞迴獲取所有子檔案和子目錄
          const allItems = await this.getAllItems(sourceBasePath);
          
          // 顯示進度提示
          const totalItems = allItems.length;
          let processedItems = 0;
          
          // 移動所有專案
          for (const item of allItems) {
            const relativePath = item.key.substring(sourceBasePath.length);
            const newPath = targetBasePath + relativePath;
            
            try {
              // 複製到新位置
              await this.copyPaste(item.key, newPath);
              // 刪除原位置
              await axios.delete(`/api/write/items/${item.key}`);
              
              // 更新進度
              processedItems++;
              this.uploadProgress = (processedItems / totalItems) * 100;
            } catch (error) {
              console.error(`移動 ${item.key} 失敗:`, error);
            }
          }
          
          // 移動目錄標記
          const targetFolderPath = targetBasePath.slice(0, -1) + '_$folder$';
          await this.copyPaste(key, targetFolderPath);
          await axios.delete(`/api/write/items/${key}`);
          
          // 清除進度
          this.uploadProgress = null;
        } else {
          // 單檔案移動邏輯，修復根目錄的情況
          const targetFilePath = normalizedPath + finalFileName;
          await this.copyPaste(key, targetFilePath);
          await axios.delete(`/api/write/items/${key}`);
        }
        
        // 重新整理檔案列表
        this.fetchFiles();
      } catch (error) {
        console.error('移動失敗:', error);
        alert('移動失敗,請檢查目標路徑是否正確');
      }
    },

    // 新增：遞迴獲取目錄下所有檔案和子目錄
    async getAllItems(prefix) {
      const items = [];
      let marker = null;
      
      do {
        const url = new URL(`/api/children/${prefix}`, window.location.origin);
        if (marker) {
          url.searchParams.set('marker', marker);
        }
        
        const response = await fetch(url);
        const data = await response.json();
        
        // 新增檔案
        items.push(...data.value);
        
        // 處理子目錄
        for (const folder of data.folders) {
          // 新增目錄標記
          items.push({
            key: folder + '_$folder$',
            size: 0,
            uploaded: new Date().toISOString(),
          });
          
          // 遞迴獲取子目錄內容
          const subItems = await this.getAllItems(folder);
          items.push(...subItems);
        }
        
        marker = data.marker;
      } while (marker);
      
      return items;
    },

    uploadFiles(files) {
      if (this.cwd && !this.cwd.endsWith("/")) this.cwd += "/";

      const uploadTasks = Array.from(files).map((file) => ({
        basedir: this.cwd,
        file,
      }));
      this.uploadQueue.push(...uploadTasks);
      setTimeout(() => this.processUploadQueue());
    },
  },

  watch: {
    cwd: {
      handler() {
        this.fetchFiles();
        const url = new URL(window.location);
        if ((url.searchParams.get("p") || "") !== this.cwd) {
          this.cwd
            ? url.searchParams.set("p", this.cwd)
            : url.searchParams.delete("p");
          window.history.pushState(null, "", url.toString());
        }
        document.title = `${
          this.cwd.replace(/.*\/(?!$)|\//g, "") || "/"
        } - 檔案庫`;
      },
      immediate: true,
    },
  },

  created() {
    window.addEventListener("popstate", (ev) => {
      const searchParams = new URL(window.location).searchParams;
      if (searchParams.get("p") !== this.cwd)
        this.cwd = searchParams.get("p") || "";
    });
  },

  components: {
    Dialog,
    Menu,
    MimeIcon,
    UploadPopup,
  },
};
</script>

<style>
.main {
  height: 100%;
}

.app-bar {
  position: sticky;
  top: 0;
  padding: 8px;
  background-color: white;
  display: flex;
}

.menu-button {
  display: flex;
  position: relative;
  margin-left: 4px;
}

.menu-button > button {
  transition: background-color 0.2s ease;
}

.menu-button > button:hover {
  background-color: whitesmoke;
}

.menu {
  position: absolute;
  top: 100%;
  right: 0;
}
</style>
