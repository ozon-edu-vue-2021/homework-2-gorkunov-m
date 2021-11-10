<template>
  <FileExplorerItem v-bind="$props" v-on="$listeners">
    <template #icon>
      <IconFile />
    </template>
    <template #name>
      {{ name }}
    </template>
  </FileExplorerItem>
</template>

<script>
import IconFile from '@/components/icons/IconFile.vue';
import FileExplorerItem from '@/components/FileExplorerItem.vue';

export default {
  name: 'File',
  components: {
    FileExplorerItem,
    IconFile
  },
  props: {
    name: String,
    level: Number,
    index: Number,
    selected: Boolean
  }
};
</script>
