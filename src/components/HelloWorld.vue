<template>
  <div>
    <a
      href="https://u8486.imgbb.com/"
      target="_blank"
      class="my-5 underline text-blue-500"
      >Lihat Semua Postingan Disini</a
    >
    <div
      class="border-dashed border-2 border-gray-300 p-4 text-center cursor-pointer w-[80%] m-auto"
      @dragover.prevent
      @drop="handleDrop"
      @click="openFileInput"
    >
      <p class="text-gray-700 text-lg mb-2">Drag & Drop or Click to Add Image</p>
      <input type="file" @change="handleFileUpload" class="hidden" ref="fileInput" />
      <div v-if="selectedFile" class="mt-4">
        <img
          :src="previewImageUrl"
          alt="Preview Image"
          class="max-w-full m-auto w-[400px]"
        />
      </div>
    </div>

    <button
      class="mt-4 bg-blue-500 text-white px-4 py-2 rounded flex items-center mx-auto justify-center"
      @click="uploadImage"
      :disabled="isUploading"
    >
      <p v-if="isUploading" class="animate-spin mr-2">&#9696;</p>
      Upload Image
    </button>

    <div class="my-8">
      <p v-if="uploadStatus === 'success'" class="text-green-700">
        {{ messages["success"] }}
        <a :href="uploadedImageUrl" target="_blank" class="underline">Lihat Di ImgBB</a>
      </p>
      <p
        v-else-if="
          uploadStatus === 'error' ||
          uploadStatus === 'invalidFormat' ||
          uploadStatus === 'fileTooLarge'
        "
        class="text-red-700"
      >
        {{ messages[uploadStatus] }}
      </p>
    </div>

    <h2 class="mt-10">Gambar Yang Di Upload</h2>

    <div class="grid gap-2 mx-auto w-[80%]">
      <div v-for="(url, index) in uploadedImageUrls" :key="index" class="mx-auto">
        <a :href="url" target="_blank">
          <img
            :src="url"
            alt="Uploaded Image"
            style="margin: 10px"
            class="w-[300px] h-[300px] object-cover"
          />
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      selectedFile: null,
      apiKey: process.env.VUE_APP_API_KEY,
      uploadStatus: null,
      uploadedImageUrl: "",
      isUploading: false,
      uploadedImageUrls: [],
      messages: {
        success: "Gambar berhasil diupload. ",
        invalidFormat: "Hanya boleh file gambar (jpg, jpeg, bmp, png, gif)",
        fileTooLarge: "File gambar terlalu besar. Max ukuran file 2MB.",
        error: "Upload error",
      },
    };
  },
  created() {
    this.loadUploadedImagesFromLocalStorage();
  },
  methods: {
    handleDrop(event) {
      event.preventDefault();
      this.selectedFile = event.dataTransfer.files[0];
    },
    openFileInput() {
      this.$refs.fileInput.click();
    },
    handleFileUpload(event) {
      this.selectedFile = event.target.files[0];
    },
    uploadImage() {
      if (!this.selectedFile) return;

      if (!this.isValidImageFormat(this.selectedFile)) {
        this.uploadStatus = "invalidFormat";
        return;
      }

      if (!this.isValidImageSize(this.selectedFile)) {
        this.uploadStatus = "fileTooLarge";
        return;
      }

      this.isUploading = true;

      const formData = new FormData();
      formData.append("key", this.apiKey);
      formData.append("image", this.selectedFile);

      axios
        .post("https://api.imgbb.com/1/upload", formData)
        .then((response) => {
          this.uploadedImageUrl = response.data.data.url;
          this.uploadStatus = "success";
          this.saveImageUrlToLocalStorage(response.data.data.url);
        })
        .catch(() => {
          this.uploadStatus = "error";
        })
        .finally(() => {
          this.isUploading = false;
        });
    },

    isValidImageFormat(file) {
      const allowedFormats = ["image/jpeg", "image/png", "image/bmp", "image/gif"];
      return allowedFormats.includes(file.type);
    },
    isValidImageSize(file) {
      return file.size <= 2 * 1024 * 1024;
    },
    saveImageUrlToLocalStorage(url) {
      const existingUrls = JSON.parse(localStorage.getItem("uploadedImageUrls")) || [];
      existingUrls.push(url);
      localStorage.setItem("uploadedImageUrls", JSON.stringify(existingUrls));
      this.loadUploadedImagesFromLocalStorage();
    },
    loadUploadedImagesFromLocalStorage() {
      this.uploadedImageUrls =
        JSON.parse(localStorage.getItem("uploadedImageUrls")) || [];
    },
  },
  computed: {
    previewImageUrl() {
      return this.selectedFile ? URL.createObjectURL(this.selectedFile) : null;
    },
  },
};
</script>

<style>
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
}
</style>
