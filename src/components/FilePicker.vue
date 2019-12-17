<template>
  <div
    v-bind:class="{ 'drag-over': isDragOver }"
    class="file-upload"
    @click="selectFile"
    @drag="stopEvent"
    @dragstart="stopEvent"
    @dragover="onDragover"
    @dragenter="onDragover"
    @dragleave="onDragout"
    @dragend="onDragout"
    @drop="onDrop"
  >
    <span>
      <span class="clickable blue--text">Select File</span>
      <span class="grey--text">&nbsp;or drag it here</span>
    </span>
    <input
      ref="fileInput"
      type="file"
      name="Datei auswählen"
      :multiple="false"
      @change="fileSelected"
    />
    <v-progress-linear :indeterminate="true" v-if="isUploading"></v-progress-linear>
  </div>
</template>
<style scoped>
.file-upload {
  margin: 12px 24px;
  padding: 24px;
  outline: 2px dashed #bdbdbd; /*grey lighten-1*/
  text-align: center;
  transition: outline-offset 0.15s ease-in-out, background-color 0.15s linear;
}
.clickable {
  text-decoration: underline;
}
.clickable:hover {
  cursor: pointer;
}
/*Hide file input hack*/
input[type="file"] {
  position: absolute;
  left: -99999px;
}
.drag-over {
  background-color: rgba(158, 158, 158, 0.3); /*grey + transparency*/
  outline-offset: 6px;
}
</style>
<script>
export default {
  name: "file-picker",
  data() {
    return {
      isDragOver: false,
      isUploading: false
    };
  },
  methods: {
    selectFile() {
      // @ts-ignore
      this.$refs.fileInput.click();
    },
    /** @param {any} event */
    fileSelected(event) {
      if (!event || !event.target.files) return;

      let file = event.target.files[0];
      this.fileChange(file);
    },
    /** @param {Event} event */
    stopEvent(event) {
      event.preventDefault();
      event.stopPropagation();
    },
    /** @param {Event} event */
    onDragover(event) {
      this.isDragOver = true;
      this.stopEvent(event);
    },
    /** @param {Event} event */
    onDragout(event) {
      this.isDragOver = false;
      this.stopEvent(event);
    },
    /** @param {Event} event */
    onDrop(event) {
      this.isDragOver = false;
      let dataTransfer = event["dataTransfer"];
      if (dataTransfer && dataTransfer.files[0]) {
        this.fileChange(dataTransfer.files[0]);
      }
      this.stopEvent(event);
    },
    /** @param {File} file */
    fileChange(file) {
      if (!file) return;

      this.$emit("file-changed", file);
    }
  }
};
</script>