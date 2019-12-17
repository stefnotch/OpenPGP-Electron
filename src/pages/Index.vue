<template>
  <q-page class="flex flex-center">
    <div>
      <div>
        <q-input
          v-model="password"
          filled
          :type="isPwd ? 'password' : 'text'"
          hint="Enter a password to contiinue"
          label="Password"
        >
          <template v-slot:append>
            <q-icon
              :name="isPwd ? 'visibility_off' : 'visibility'"
              class="cursor-pointer"
              @click="isPwd = !isPwd"
            />
          </template>
        </q-input>
      </div>
      <div
        v-if="initialized"
        :style="{ visibility: password ? 'visible' : 'hidden' }"
        class="flex row"
      >
        <div>
          <h2>Encrypt -</h2>
          <file-picker
            type="file"
            v-on:file-changed="fileChangedEncrypt"
          ></file-picker>
        </div>
        <div>
          <h2>-> Decrypt</h2>
          <file-picker
            type="file"
            v-on:file-changed="fileChangedDecrypt"
          ></file-picker>
        </div>
      </div>
      <div v-if="!initialized">Initializing...</div>
      <span style="width: 100%; text-align: center;">{{message}}</span>
    </div>
  </q-page>
</template>

<script>
import FilePicker from "./../components/FilePicker.vue";
import { QInput } from "quasar";
import * as fs from "fs";
import { shell } from "electron";
//import * as openpgp from "openpgp";
let openpgp = {};

function getOpenPgp() {
  let nodeRequire = window.require;
  let nodeExports = window.exports;
  let nodeModule = window.module;
  delete window.require;
  delete window.exports;
  delete window.module;

  openpgp = nodeRequire("openpgp");

  window.require = nodeRequire;
  window.exports = nodeExports;
  window.module = nodeModule;
}
getOpenPgp();

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
      isPwd: true,
      message: ""
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
      this.message = "Encrypting-->";
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
      this.message = "-->Decrypting";
      let fileReader = new FileReader();

      fileReader.onloadend = evt => {
        if (evt.target.readyState == FileReader.DONE) {
          let fileForOpenpgpjs = evt.target.result;
          this.decryptFile(new Uint8Array(fileForOpenpgpjs), file.path);
        }
      };

      fileReader.readAsArrayBuffer(file);
    },

    async encryptFile(fileBuffer, path) {
      let options = {
        message: await openpgp.message.fromBinary(fileBuffer), // parse encrypted bytes
        passwords: [this.password], // decrypt with password
        armor: false,
        format: "binary" // output as Uint8Array
      };

      openpgp.encrypt(options).then(ciphertext => {
        let newPath = path + ".gpg";
        fs.writeFileSync(path + ".gpg", ciphertext.message.packets.write());
        this.message = "Success " + newPath;
      });
    },
    async decryptFile(fileBuffer, path) {
      let options = {
        message: await openpgp.message.read(fileBuffer), // parse encrypted bytes
        passwords: [this.password], // decrypt with password
        format: "binary" // output as Uint8Array
      };

      openpgp.decrypt(options).then(
        plaintext => {
          let newPath = path.replace(/\.gpg$/, "");
          fs.writeFileSync(newPath, plaintext.data);

          this.message = "Success " + newPath;
        },
        error => {
          this.message = error + "";
        }
      );
    }
  }
};
</script>
