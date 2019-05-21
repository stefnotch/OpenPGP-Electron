<template>
  <q-page class="flex flex-center">
    <div>
      <div>
        <q-input v-model="password" filled :type="isPwd ? 'password' : 'text'" hint="Password">
          <template v-slot:append>
            <q-icon
              :name="isPwd ? 'visibility_off' : 'visibility'"
              class="cursor-pointer"
              @click="isPwd = !isPwd"
            />
          </template>
        </q-input>
      </div>
      <div v-if="initialized" class="flex row">
        <div>
          <h2>Encrypt -</h2>
          <file-picker type="file" v-on:file-changed="fileChangedEncrypt"></file-picker>
        </div>
        <div>
          <h2>-> Decrypt</h2>
          <file-picker type="file" v-on:file-changed="fileChangedDecrypt"></file-picker>
        </div>
      </div>
      <div v-if="!initialized">Initializing...</div>
    </div>
  </q-page>
</template>

<style>
</style>

<script>
import FilePicker from "./../components/FilePicker.vue";
//import openpgp from "openpgp";
import * as openpgp from "openpgp";
import fs from "fs";
import { QInput } from "quasar";

export default {
  name: "PageIndex",
  components: {
    FilePicker,
    QInput
  },
  data() {
    return {
      initialized: true,
      password: "",
      isPwd: true
    };
  },
  created() {
    // Disabled for now...
    /*openpgp
      .initWorker({ path: "statics/openpgp.worker.js" })
      .then(() => (this.initialized = true));*/
  },
  methods: {
    /** @param {File} file */
    fileChangedEncrypt(file) {
      let fileReader = new FileReader();

      fileReader.onloadend = evt => {
        if (evt.target.readyState == FileReader.DONE) {
          let fileForOpenpgpjs = evt.target.result;
          this.encryptFile(new Uint8Array(fileForOpenpgpjs), file.path);
        }
      };

      fileReader.readAsArrayBuffer(file);
    },
    /** @param {File} file */
    fileChangedDecrypt(file) {
      let fileReader = new FileReader();

      fileReader.onloadend = evt => {
        if (evt.target.readyState == FileReader.DONE) {
          let fileForOpenpgpjs = evt.target.result;
          this.decryptFile(fileForOpenpgpjs, file.path);
        }
      };

      fileReader.readAsArrayBuffer(file);
    },

    async encryptFile(fileBuffer, path) {
      let options = {
        message: await openpgp.message.fromBinary(fileBuffer), // parse encrypted bytes
        passwords: [this.password], // decrypt with password
        format: "binary" // output as Uint8Array
      };

      openpgp.encrypt(options).then(ciphertext => {
        fs.writeFileSync(path + ".gpg", ciphertext.data);
      });
    },
    async decryptFile(fileBuffer, path) {
      let options = {
        message: await openpgp.message.read(fileBuffer), // parse encrypted bytes
        passwords: [this.password], // decrypt with password
        format: "binary" // output as Uint8Array
      };

      openpgp.decrypt(options).then(plaintext => {
        console.log(plaintext);
        fs.writeFile(path.replace(/\.gpg$/), plaintext);
      });
    }
  }
};
</script>
