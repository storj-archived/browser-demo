<template>
  <div id="app" v-on:click="handleOpenDropdownClicks">
    <router-view></router-view>
  </div>
</template>

<style scoped>
@import "../node_modules/browser/dist/browser.css";
</style>

<script>
export default {
	methods: {
		handleOpenDropdownClicks() {
			if (this.$store.state.files.openedDropdown) {
				this.$store.dispatch("files/openDropdown", null);
			}

			if (this.$store.state.files.selectedFile) {
				this.$store.dispatch("files/updateSelectedFile", null);
			}
		}
	}
};
</script>
