<template>
  <div
    :class="[
      'FileExplorerItem',
      {
        '-opened': opened,
        '-selected': selected,
        '-disabled': !active
      }
    ]"
    :tabindex="0"
    :style="`--level: ${level}`"
    :data-index="index"
    @keypress.space.enter.prevent="$emit('select')"
    @keydown.right="$emit('open')"
    @keydown.left="$emit('close')"
    @click="$emit('select')"
    @focus="$emit('focus')"
  >
    <div class="FileExplorerItem_arrow">
      <slot name="arrow"></slot>
    </div>
    <div class="FileExplorerItem_icon">
      <slot name="icon"></slot>
    </div>
    <div class="FileExplorerItem_name">
      <slot name="name"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FileExplorerItem',
  inheritAttrs: false,
  props: {
    opened: {
      type: Boolean,
      default: false
    },
    active: {
      type: Boolean,
      default: true
    },
    selected: {
      type: Boolean,
      default: false
    },
    level: {
      type: Number,
      default: 0
    },
    index: {
      type: Number,
      default: 0
    }
  }
};
</script>

<style lang="scss">
.FileExplorerItem {
  $this: &;

  display: flex;
  align-items: center;
  padding: 8px 0;
  padding-left: calc(var(--indent) * var(--level, 0) * 1ch + 30px);
  cursor: pointer;
  outline: none;
  transition: ease color 200ms, ease background-color 200ms;

  &:hover {
    background-color: #eee;
  }

  &.-disabled {
    color: #999;
  }

  &.-selected {
    background-color: #ccc;
  }

  &.-opened {
    #{$this}_arrow {
      transform: rotate(0) translateY(-3px);
    }
  }

  &_arrow {
    flex-shrink: 0;
    width: 12px;
    height: 12px;
    margin-left: -17px;
    transform: translateX(-3px) rotate(-90deg);
    transition: ease-in-out 100ms transform;
  }

  &_arrow + &_icon {
    margin-left: 5px;
  }

  &_icon {
    width: 20px;
    height: 20px;
  }

  &_name {
    margin-left: 3px;
  }
}
</style>
