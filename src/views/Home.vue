<template>
  <div class="home">
    <div class="file-browser">
      <file-browser></file-browser>
    </div>
  </div>
</template>

<script>
import { FileBrowser } from "browser";

export default {
  name: 'Home',
  components: {
    FileBrowser
  }
}
</script>
