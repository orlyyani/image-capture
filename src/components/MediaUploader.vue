<style scoped></style>

<template>
  <div class="media-uploader">
    <input
      v-show="false"
      :ref="'file-input'"
      type="file"
      :accept="`/${mediaType}*`"
      :multiple="!singleMedia"
      @change="initMediaToUpload"
    />
  </div>
</template>

<script>
export default {
  name: 'MediaUploader',
  props: {
    singleMedia: {
      type: Boolean,
      default: true,
    },
    mediaType: {
      type: String,
      default: 'image',
    },
  },
  data() {
    return {
      imageHolder: [],
    }
  },
  methods: {
    initMediaToUpload(e) {
      const files = e.target.files || e.dataTransfer.files
      const formData = new FormData()
      const reader = new FileReader()

      for (let i = 0; i < files.length; i++) {
        const file = files[i]

        formData.append('avatar', file)
        reader.readAsDataURL(file)
      }

      const on = this
      reader.onload = function (e) {
        on.mediaFile({ avatar: formData, image_preview: e.target.result })
      }
    },

    openFile() {
      this.$refs['file-input'].click()
    },

    mediaFile(data) {
      this.$emit('onMediaLoad', data)
    },
  },
}
</script>
