<template>
  <v-list-tile @click="triggerUpload">
    <v-list-tile-action>
      <v-icon>cloud_upload</v-icon>
    </v-list-tile-action>
    <v-list-tile-title v-translate>Upload</v-list-tile-title>
    <input id="fileUploadInput" type="file" name="file" @change="onChangeInputFile" multiple ref="input" />
  </v-list-tile>
</template>

<script>
import FileUpload from '../FileUpload.js'
import { mapActions, mapGetters } from 'vuex'

export default {
  props: {
    url: { type: String, required: true },
    headers: {
      type: Object,
      default: () => {
        return {}
      }
    },
    additionalData: {
      type: Object,
      default: () => {
        return {}
      }
    },
    requestType: { type: String, default: 'PUT' }
  },
  data () {
    return {
    }
  },
  computed: {
    ...mapGetters('Files', ['inProgress'])
  },
  methods: {
    ...mapActions('Files', ['addFileToProgress']),
    triggerUpload () {
      this.$refs.input.click()
    },
    onChangeInputFile (e) {
      let files = e.target.files || e.dataTransfer.files
      if (!files.length) return
      for (let file of files) {
        this.upload(file)
      }
    },

    upload (file) {
      this.addFileToProgress(file)
      let fileUpload = new FileUpload(file, this.url, this.headers, this.onProgress, this.requestType)
      fileUpload
        .upload(this.additionalData)
        .then(e => {
          this.$emit('success', e, file)
          this.cleanInput()
        })
        .catch(e => {
          this.$emit('error', e)
          this.cleanInput()
        })
    },

    cleanInput () {
      let input = this.$refs.input
      if (input) {
        input.value = ''
      }
    },

    onProgress (e, file) {
      let progress = parseInt(e.loaded * 100 / e.total)
      this.$emit('progress', {
        fileName: file.name,
        progress
      })
    }
  }
}
</script>

<style scoped="true">
  #fileUploadInput {
    position: absolute;
    left: -99999px;
  }
</style>
