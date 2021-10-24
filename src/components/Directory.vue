<template>
  <div :class="['Directory', { '-empty': isEmpty }]">
    <div class="Directory_row">
      <FileExplorerItem
        :opened="opened"
        :selected="isSelectedDirectory()"
        :focusable="!isEmpty"
        @open="openDirectory"
        @close="closeDirectory"
        @select="toggleDirectory"
        @focus="selectDirectory"
      >
        <template #arrow v-if="!isEmpty">
          <IconArrow />
        </template>
        <template #icon>
          <IconDirectory />
        </template>
        <template #name>
          {{ name }}
        </template>
      </FileExplorerItem>
    </div>
    <transition name="fade">
      <ul class="Directory_list" :style="`--level: ${level}`" v-if="opened">
        <li
          class="Directory_item"
          v-for="item in sortedContents"
          :key="`${item.type}-${item.name}`"
        >
          <template v-if="item.type === 'directory'">
            <Directory
              :name="item.name"
              :contents="item.contents"
              :selected="selected"
              :level="level + 1"
              :path="`${path}/${item.name}`"
            ></Directory>
          </template>
          <template v-if="componentsMap[item.type]">
            <component
              :is="componentsMap[item.type]"
              :name="item.name"
              :selected="isSelectedFile(item.name)"
              @focus="selectFile(item.name)"
            ></component>
          </template>
        </li>
      </ul>
    </transition>
  </div>
</template>

<script>
import { eventBus } from '@/eventBus';

import FileExplorerItem from '@/components/FileExplorerItem.vue';
import File from '@/components/File.vue';
import IconArrow from '@/components/icons/IconArrow.vue';
import IconDirectory from '@/components/icons/IconDirectory.vue';
import Link from '@/components/Link.vue';

const Directory = {
  name: 'Directory',
  components: {
    Directory,
    File,
    FileExplorerItem,
    IconArrow,
    IconDirectory,
    Link
  },
  props: {
    name: {
      type: String,
      required: true
    },
    path: {
      type: String,
      required: true
    },
    contents: {
      type: Array,
      default: () => []
    },
    selected: {
      type: String,
      default: ''
    },
    level: {
      type: Number,
      default: 1
    }
  },
  data: () => ({
    componentsMap: {
      file: File,
      link: Link
    },
    opened: false
  }),
  computed: {
    isEmpty() {
      return !this.contents.length;
    },
    sortedContents() {
      return this.contents.slice().sort((a, b) => {
        if (this.isDirectory(a.type) && this.isDirectory(b.type)) {
          return a.name.localeCompare(b.name);
        }

        return this.isDirectory(a.type) ? -1 : 1;
      });
    }
  },
  methods: {
    isDirectory(type) {
      return type === 'directory';
    },
    openDirectory() {
      this.opened = true;
    },
    closeDirectory() {
      this.opened = false;
    },
    toggleDirectory() {
      this.opened = !this.opened;
    },
    getFilePath(name) {
      return `${this.path}/${name}`;
    },
    isSelectedFile(name) {
      return this.selected === this.getFilePath(name);
    },
    isSelectedDirectory() {
      return this.selected === this.path;
    },
    selectDirectory() {
      eventBus.$emit('fileexplorer-selected', this.path);
    },
    selectFile(name) {
      eventBus.$emit('fileexplorer-selected', this.getFilePath(name));
    }
  }
};

export default Directory;
</script>

<style lang="scss">
.Directory {
  opacity: var(--opacity, 1);
  transition: ease 200ms opacity;

  &.-empty {
    opacity: 0.5;
    pointer-events: none;
  }

  &_row {
  }

  &_list {
    background-color: #fff;
  }

  &_item {
    background-color: #fff;
    transition: ease background-color 200ms;
    border-top: 1px solid #ccc;
  }
}
</style>
