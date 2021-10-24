<template>
  <div
    class="FileExplorer"
    :style="`--indent: ${indent}`"
    @keydown.up.prevent="pressUpHandler"
    @keydown.down.prevent="pressDownHandler"
  >
    <div class="FileExplorer_path">{{ selected ? selected : '/' }}</div>
    <div class="FileExplorer_tree">
      <template v-if="loading">
        <div class="FileExplorer_text">
          Loading...
        </div>
      </template>
      <template v-else-if="directory.name && directory.contents">
        <Directory
          :name="directory.name"
          :contents="directory.contents"
          :path="`/${directory.name}`"
          :selected="selected"
        />
      </template>
      <template v-else>
        <div class="FileExplorer_text">
          Something went wrong...
        </div>
      </template>
    </div>
  </div>
</template>

<script>
import { eventBus } from '@/eventBus';
import Directory from '@/components/Directory.vue';

export default {
  name: 'FileExplorer',
  components: {
    Directory
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
      selected: ''
    };
  },
  methods: {
    selectHandler(path) {
      if (this.selected === path) {
        this.selected = '';
      } else {
        this.selected = path;
      }
    },
    pressUpHandler() {
      this.setNextFocus(-1);
    },
    pressDownHandler() {
      this.setNextFocus();
    },
    setNextFocus(direction = 1) {
      const focusable = document.querySelectorAll('[tabindex]');
      const activeIndex = [...focusable].findIndex(
        el => el === document.activeElement
      );
      const nextIndex = activeIndex + direction;
      const fixedIndex = Math.max(Math.min(nextIndex, focusable.length - 1), 0);
      const nextElement = focusable[fixedIndex];

      nextElement.focus();
    }
  },
  mounted() {
    eventBus.$on('fileexplorer-selected', this.selectHandler);
  },
  destroyed() {
    eventBus.$off('fileexplorer-selected', this.selectHandler);
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

  &_tree {
    max-height: 600px;
    margin-top: 15px;
    border: 1px solid #ccc;
    border-radius: 5px;
    box-shadow: 0px 3px 6px -6px #333;
    overflow: auto;
  }

  &_text {
    padding: 10px 16px;
  }
}
</style>
