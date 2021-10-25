<template>
  <FileExplorerItem
    v-bind="$props"
    :opened="opened"
    :active="!empty"
    @open="openDirectory"
    @close="closeDirectory"
    @select="toggleDirectory"
    @focus="selectDirectory"
  >
    <template #arrow v-if="!empty">
      <IconArrow />
    </template>
    <template #icon>
      <IconDirectory />
    </template>
    <template #name>
      {{ name }}
    </template>
  </FileExplorerItem>
</template>

<script>
import FileExplorerItem from '@/components/FileExplorerItem.vue';
import IconArrow from '@/components/icons/IconArrow.vue';
import IconDirectory from '@/components/icons/IconDirectory.vue';

export default {
  name: 'Directory',
  components: {
    FileExplorerItem,
    IconArrow,
    IconDirectory
  },
  props: {
    name: String,
    level: Number,
    selected: Boolean,
    opened: Boolean,
    index: Number,
    empty: Boolean
  },
  methods: {
    openDirectory() {
      this.$emit('open');
    },
    closeDirectory() {
      this.$emit('close');
    },
    toggleDirectory() {
      this.$emit('toggle');
    },
    selectDirectory() {
      this.$emit('select');
    }
  }
};
</script>
