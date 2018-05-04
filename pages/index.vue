<template>
  <section class="section">
    <div class="columns">
      <section class="column is-one-third has-text-centered">
        <b-field>
          <b-upload v-model="dropFiles"
                    multiple
                    drag-drop
                    @input="handleFileAdd">
            <section class="section">
              <div class="content has-text-centered">
                <p>
                  <b-icon
                    icon="upload"
                    size="is-large">
                  </b-icon>
                </p>
                <p>Drop your files here or click to upload</p>
              </div>
            </section>
          </b-upload>
        </b-field>

        <div class="tags">
            <span v-for="(file, index) in dropFiles"
                  :key="index"
                  class="tag is-primary">
                {{file.name}}
                <button class="delete is-small"
                        type="button"
                        @click="deleteDropFile(index)">
                </button>
            </span>
        </div>
      </section>
      <section class="column">
        <div class="columns is-multiline is-centered">
          <div class="column is-half" v-for="(url,index) in imageUrls" :key="index">
            <div class="card">
              <div class="card-image">
                <figure class="image">
                  <img :src="url" alt="" class="image">
                </figure>
              </div>
              <div class="card-content">
                <div class="content">
                  <pre v-highlightjs="jsonData[index]"><code
                    class="javascript"></code></pre>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </div>
    <b-loading :active="isLoading" :is-full-page="true"></b-loading>
  </section>
</template>

<script>
  export default {
    data() {
      return {
        dropFiles: [],
        jsonData: [],
        imageUrls: [],
        isLoading: false,
      }
    },
    methods: {
      deleteDropFile(index) {
        this.dropFiles.splice(index, 1)
        this.jsonData.splice(index, 1)
        this.imageUrls.splice(index, 1)
      },
      buildImageUrl(file) {
        return URL.createObjectURL(file)
      },
      fetchData(file) {
        this.$axios.setHeader('Content-Type', 'multipart/form-data')
        let bodyFormData = new FormData();
        bodyFormData.set('file', file);

        return this.$axios.post('/parse-text', bodyFormData)
          .then(response => {
            const data = JSON.stringify({ parsedText: response.data.response })
            this.jsonData.push(data)
          })
      },
      handleFileAdd(files) {
        const filesLength = this.dropFiles.length
        const currentFile = files[filesLength - 1]
        console.log(filesLength)
        if (this.jsonData.length !== filesLength) {
          this.isLoading = true
          this.fetchData(currentFile).then(() => {
            this.isLoading = false
            this.imageUrls.push(this.buildImageUrl(currentFile))
          })
        }


      }
    }
  }
</script>

<style>
  pre, code {
    white-space: normal;
  }
</style>
