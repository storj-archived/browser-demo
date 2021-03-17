<style>
  .home {
    font-family: sans-serif;
  }
</style>

<template>
  <div class="home">
    <div class="file-browser">
      <div v-if="displayCredentials">
        <div class="container col-6 mt-5">
          <div class="mb-5 text-center">Please enter the following credentials to use the file browser</div>

          <div class="mb-3">
            <label for="access-grant" class="form-label">Access Grant</label>
            <input v-model="accessGrant" type="text" class="form-control" id="access-grant">

            <button v-on:click="generateAccess" v-bind:disabled="!generateAccessEnabled" type="submit" class="btn btn-primary mt-3">Generate access key and create bucket if it doesn't exist</button>
          </div>

          <div class="my-5 text-center">Or</div>

          <div class="mb-3">
            <label for="access-key" class="form-label">Access Key</label>
            <input v-bind:disabled="generated" v-model="accessKey" type="text" class="form-control" id="access-key">
          </div>

          <div class="mb-3">
            <label for="secret-key" class="form-label">Secret Key</label>
            <input v-bind:disabled="generated" v-model="secretKey" type="text" class="form-control" id="secret-key">
          </div>

          <div class="mb-3">
            <label for="access-key" class="form-label">Bucket</label>
            <input v-bind:disabled="generated" v-model="bucket" type="text" class="form-control" id="bucket">
          </div>

          <button v-on:click="initializeFiles" v-bind:disabled="!submitEnabled" type="submit" class="btn btn-primary mb-3">Submit</button>
        </div>
      </div>
      <div v-else>
        <file-browser></file-browser>
      </div>
    </div>
  </div>
</template>

<script>
import { FileBrowser } from "browser";
import S3 from "aws-sdk/clients/s3";

export default {
  name: 'Home',
  data: () => ({
    displayCredentials: true,
    accessGrant: "",
    endpoint: "gateway.tardigradeshare.io",
    accessKey: "",
    secretKey: "",
    bucket: "",
    browserRoot: "/",
    generated: false
  }),
  components: {
    FileBrowser
  },
  computed: {
    submitEnabled() {
      return this.accessKey && this.secretKey && this.bucket;
    },
    generateAccessEnabled() {
      return this.accessGrant;
    }
  },
  methods: {
    async generateAccess() {
      this.generated = true;

      const response = await fetch("https://auth.tardigradeshare.io/v1/access", {
        method: "POST",
        body: JSON.stringify({
          access_grant: this.accessGrant,
          public: false
        })
      });

      const {
        access_key_id,
        secret_key,
        endpoint
      } = await response.json();

      this.accessKey = access_key_id;
      this.secretKey = secret_key;
      this.endpoint = endpoint;

      const s3 = new S3({
        accessKeyId: this.accessKey,
        secretAccessKey: this.secretKey,
        endpoint: this.endpoint,
        s3ForcePathStyle: true,
        signatureVersion: "v4"
      });

      try {
        await s3.createBucket({ Bucket: "filebrowser" }).promise();
      } catch (e) {
        if (e.message !== "The requested bucket name is not available. The bucket namespace is shared by all users of the system. Please select a different name and try again.") {
          throw e;
        }
      }

      this.bucket = "filebrowser";
    },
    initializeFiles() {
      this.$store.commit('files/init', {
        endpoint: this.endpoint,
        accessKey: this.accessKey,
        secretKey: this.secretKey,
        bucket: this.bucket,
        browserRoot: this.browserRoot
      });

      this.displayCredentials = false;
    },
  },
}
</script>
