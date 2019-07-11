<template>
  <ele-upload-video
    :fileSize="20"
    :responseFn="handleResponse"
    @error="handleUploadError"
    action="https://jsonplaceholder.typicode.com/posts/"
    name="video"
    style="margin: 50px"
    v-model="video"
  />
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      video: null
    }
  },
  methods: {
    handleUploadError (error) {
      this.$notify.error({
        title: 'Upload prompt',
        message: 'environment cannot upload files'
      })
      // eslint-disable-next-line no-console
      console.log('error', error)
    },
    handleResponse({ id }) {
      return 'https://png.pngtree.com/svg/20170213/hook_success_405368.png' + id
    }
  }
}
</script>

