<template>
  <div class="ele-upload-video">
    <!-- Upload component -->
    <el-upload
      :accept="accept"
      :action="action"
      :before-upload="handleBeforeUploadVideo"
      :data="data"
      :disabled="videoUploadPercent > 0 && videoUploadPercent < 100"
      :headers="headers"
      :httpRequest="httpRequest"
      :name="name"
      :on-error="handleUploadError"
      :on-progress="handleUploadProcess"
      :on-success="handleUploadSuccess"
      :show-file-list="false"
      :withCredentials="withCredentials"
      drag
      v-if="!value"
    >
      <!-- Upload progress -->
      <el-progress
        :percentage="videoUploadPercent"
        style="margin-top: 20px;"
        type="circle"
        v-if="videoUploadPercent > 0"
      />

      <!-- Upload prompt -->
      <template v-else>
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">
          Drag the video here, or
          <em>Click to upload</em>
        </div>
        <div
          class="el-upload__tip"
          slot="tip"
          v-if="isShowTip"
        >
          Please upload

          <span
            style="color: #F56C6C"
          >&nbsp;{{this.fileType ? this.fileType.join('/') : 'video'}}&nbsp;</span>
Format file
          <template v-if="fileSize">
            
And the file size does not exceed
            <span style="color: #F56C6C">{{fileSize}}</span>&nbsp;MB
          </template>
        </div>
      </template>
    </el-upload>

    <!-- Video thumbnail -->
    <vue-hover-mask v-if="value">
      <video
        :autoplay="false"
        :src="value"
        :style="{width: width + 'px', height: height ? height + 'px' : 'auto'}"
      >
Your browser does not support video playback</video>
      <template v-slot:action>
        <span
          @click="handlePlayerVideo"
          class="ele-upload-video_mask__item"
        >
          <i class="el-icon-zoom-in"></i>
        </span>
        <span
          @click="handleDeleteVideo"
          class="ele-upload-video_mask__item"
        >
          <i class="el-icon-delete"></i>
        </span>
      </template>
    </vue-hover-mask>

    <!-- Pop-up window play -->
    <el-dialog
      :visible.sync="isShowVideo"
      append-to-body
    >
      <video
        :autoplay="true"
        :src="value"
        controls="controls"
        style="width: 100%"
        v-if="isShowVideo"
      >Your browser does not support video playback
</video>
    </el-dialog>
  </div>
</template>

<script>
import VueHoverMask from 'vue-hover-mask/src/index'

export default {
  name: 'EleUploadVideo',
  components: {
    VueHoverMask
  },
  props: {
    // value
    value: {
      type: String
    },
    // Upload address
    action: {
      type: String,
      required: true
    },
    // Response handler
    responseFn: Function,
    // File size limit(Mb)
    fileSize: {
      type: Number
    },
    // Display width(px)
    width: {
      type: Number,
      default: 360
    },
    // Display height(auto)
    height: {
      type: Number
    },
    // Whether to display prompts
    isShowTip: {
      type: Boolean,
      default: true
    },
    // file type
    fileType: {
      type: Array
    },
     // Set the request header for uploading (same official website)
    headers: Object,
    // Support sending cookie Voucher information
    withCredentials: {
      type: Boolean,
      default: false
    },
    // Additional parameters attached to the upload (same official website)
    data: {
      type: Object
    },
    // Uploaded file field name (Same official website)
    name: {
      type: String,
      default: 'file'
    },
      // Override the default upload behavior, you can customize the implementation of the upload (same official website)
    httpRequest: Function,
    // Accept the uploaded file type (this parameter is invalid in thumbnail-mode mode) (same official website)
    accept: String
  },
  data () {
    return {
      isShowVideo: false,
      videoUploadPercent: 0
    }
  },
  methods: {
    // Upload size and format detection
    handleBeforeUploadVideo (file) {
      // Check format
      let isVideo = false
      if (Array.isArray(this.fileType)) {
        const type = file.type.split('/')
        isVideo = type[0] === 'video' && this.fileType.includes(type[1])
      } else {
        isVideo = file.type.includes('video')
      }

      if (!isVideo) {
        this.$message.error(`${file.name}Incorrect format, Please upload a properly formatted video`)
        return false
      }

      // Check file size
      if (this.fileSize) {
        const isLt = file.size / 1024 / 1024 < this.fileSize
        if (!isLt) {
          this.$message.error(`Upload video size cannot exceed${this.fileSize}MB!`)
          return false
        }
      }
      return true
    },

    // Upload progress
    handleUploadProcess (event, file) {
      this.videoUploadPercent = Number(file.percentage.toFixed(0))
    },

    // Upload success
    handleUploadSuccess (response, file) {
      this.videoUploadPercent = 0
      this.$message.success('success')
      if (this.responseFn) {
        this.$emit('input', this.responseFn(response, file))
      } else {
        this.$emit('input', response)
      }
    },

    // upload failed
    handleUploadError (err, file, fileList) {
      this.$message.error('upload failed')
      this.videoUploadPercent = 0
      this.$emit('error', err, file, fileList)
    },

    // Delete video
    handleDeleteVideo () {
      this.$emit('delete')
      this.$emit('input', null)
    },

    // Play video
    handlePlayerVideo () {
      this.isShowVideo = true
    }
  }
}
</script>

<style>
.ele-upload-video_mask__item {
  padding: 0 10px;
}
</style>
