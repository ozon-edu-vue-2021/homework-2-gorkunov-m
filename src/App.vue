<template>
  <div class="App">
    <FileExplorer v-if="directory" :directory="directory" :loading="loading" />
  </div>
</template>

<script>
import FileExplorer from '@/components/FileExplorer.vue';

export default {
  name: 'App',
  components: {
    FileExplorer
  },
  data: () => ({
    loading: false,
    directory: null
  }),
  async mounted() {
    this.fetchFiles();
  },
  methods: {
    async fetchFiles() {
      try {
        this.loading = true;

        const response = await fetch('/static/node_modules.json');
        const directory = await response.json();

        this.directory = directory;
        // this.directory = Object.freeze(directory);
      } catch (error) {
        console.error(error);
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>
<style lang="scss">
.App {
  padding: 10px;
}
</style>
