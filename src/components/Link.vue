<template>
  <FileExplorerItem v-bind="$props" v-on="$listeners">
    <template #icon>
      <IconLink />
    </template>
    <template #name>
      {{ name }}
    </template>
  </FileExplorerItem>
</template>

<script>
import IconLink from '@/components/icons/IconLink.vue';
import FileExplorerItem from '@/components/FileExplorerItem.vue';

export default {
  name: 'Link',
  components: {
    FileExplorerItem,
    IconLink
  },
  props: {
    name: String,
    level: Number,
    index: Number,
    selected: Boolean
  }
};
</script>
