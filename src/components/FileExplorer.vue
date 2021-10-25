<template>
  <div class="FileExplorer" :style="`--indent: ${indent}`">
    <div class="FileExplorer_path">
      /{{ selectedItem ? selectedItem.path : '' }}
    </div>
    <div class="FileExplorer_container">
      <div v-if="loading" class="FileExplorer_text">
        Loading...
      </div>
      <VirtualizedList
        ref="virtualList"
        :innerContainerClass="'FileExplorer_list'"
        :outerContainerClass="'FileExplorer_scroller'"
        :itemClass="'FileExplorer_item'"
        :items="visibleList"
        :item-height="36"
      >
        <template v-slot="{ item, index }">
          <Directory
            v-if="isDirectory(item)"
            :opened="isOpenedDirectory(item)"
            :empty="isEmptyDirectory(item)"
            :name="item.name"
            :level="item.level"
            :selected="isSelected(item)"
            :key="item.path"
            :index="index"
            @open="openDirectory(item)"
            @close="closeDirectory(item)"
            @toggle="toggleDirectory(item)"
            @select="selectItem(item, index)"
          />
          <component
            v-else
            :is="componentsMap[item.type]"
            :index="index"
            :name="item.name"
            :level="item.level"
            :selected="isSelected(item)"
            :key="item.path"
            @focus="selectItem(item, index)"
          ></component>
        </template>
      </VirtualizedList>
    </div>
  </div>
</template>

<script>
import { throttle } from '@/throttle';

import Directory from '@/components/Directory.vue';
import File from '@/components/File.vue';
import Link from '@/components/Link.vue';
import VirtualizedList from './VirtualizedListRender';

const sortContents = contents => {
  return contents.slice().sort((a, b) => {
    if (a.type === 'directory' && b.type === 'directory') {
      return a.name.localeCompare(b.name);
    }

    return a.type === 'directory' ? -1 : 1;
  });
};

const getDirectoryList = (directory, level = 0, parent = null) => {
  const path = `${parent ? `${parent.path}/` : ''}${directory.name}`;
  const contents = directory.contents?.length
    ? sortContents(directory.contents)
    : null;
  const meta = {
    contents: Boolean(contents),
    id: path,
    level,
    name: directory.name,
    parent,
    path,
    type: directory.type
  };

  const getContents = () => {
    return contents
      ? contents.reduce((dir, item) => {
          return [...dir, ...getDirectoryList(item, level + 1, meta)];
        }, [])
      : [];
  };

  return [meta, ...getContents()];
};

export default {
  name: 'FileExplorer',
  components: {
    Directory,
    VirtualizedList
  },
  props: {
    directory: {
      type: Object,
      required: true,
      validator: value => {
        return typeof value.name === 'string' && Array.isArray(value.contents);
      }
    },
    indent: {
      type: Number,
      default: 2
    },
    loading: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      selectedItem: null,
      focusedIndex: null,
      throttledSetFocus: null,
      openedDirectories: [],
      componentsMap: {
        file: File,
        link: Link
      }
    };
  },
  computed: {
    directoryList() {
      return Object.freeze(getDirectoryList(this.directory));
    },
    visibleList() {
      return this.directoryList.filter(item => this.isVisible(item));
    }
  },
  methods: {
    isVisible(item) {
      return item.parent
        ? this.isOpenedDirectory(item.parent) && this.isVisible(item.parent)
        : true;
    },
    isSelected(item) {
      return this.selectedItem && item.path === this.selectedItem.path;
    },
    isDirectory(item) {
      return item.type === 'directory';
    },
    isEmptyDirectory(directory) {
      return !directory.contents;
    },
    isOpenedDirectory(directory) {
      return (
        this.isDirectory(directory) &&
        this.openedDirectories.includes(directory.path)
      );
    },
    openDirectory(directory) {
      this.openedDirectories.push(directory.path);
      this.$refs.virtualList.update();
    },
    closeDirectory(directory) {
      this.openedDirectories = this.openedDirectories.filter(
        path => !path.startsWith(directory.path)
      );
      this.$refs.virtualList.update();
    },
    toggleDirectory(directory) {
      this.isOpenedDirectory(directory)
        ? this.closeDirectory(directory)
        : this.openDirectory(directory);
    },
    selectItem(item, index) {
      if (item) {
        this.selectedItem = { ...item, index };
        this.focusedIndex = index;
      } else {
        this.selectedItem = null;
        this.focusedIndex = null;
      }
    },
    keypressHandler(e) {
      if (['ArrowDown', 'ArrowUp', 'Tab'].includes(e.key)) {
        e.preventDefault();
        e.stopPropagation();
      }

      if (e.key === 'ArrowUp' || (e.key === 'Tab' && e.shiftKey)) {
        this.throttledSetFocus(-1);
        return;
      }

      if (e.key === 'ArrowDown' || e.key === 'Tab') {
        this.throttledSetFocus();
      }
    },
    setNextFocus(direction = 1) {
      const virtualList = this.$refs.virtualList;
      const nextIndex = this.focusedIndex + direction;
      const fixedIndex = Math.max(
        Math.min(nextIndex, this.visibleList.length - 1),
        0
      );
      const nextElement = virtualList.$el.querySelector(
        `[data-index="${fixedIndex}"]`
      );

      nextElement?.focus();
    }
  },
  mounted() {
    this.throttledSetFocus = throttle(dir => this.setNextFocus(dir), 100);
    document.addEventListener('keydown', this.keypressHandler);
  },
  destroyed() {
    document.removeEventListener('keydown', this.keypressHandler);
  }
};
</script>

<style lang="scss">
.FileExplorer {
  max-width: 1020px;
  margin: 15px auto;
  font-family: sans-serif;
  font-size: 14px;
  font-weight: 400;
  line-height: 1.21;

  &_path {
    color: #333;
  }

  &_container {
    margin-top: 15px;
    border: 1px solid #ccc;
    border-radius: 5px;
    box-shadow: 0px 3px 6px -6px #333;
    overflow: hidden;
  }

  &_scroller {
    max-height: 600px;
    overflow: auto;
  }

  &_list {
  }

  &_item {
    & + & {
      border-top: 1px solid #ccc;
    }
  }

  &_text {
    padding: 10px 16px;
  }
}
</style>
