<template functional>
  <div
    :class="[
      'FileExplorerItem',
      {
        '-opened': props.opened,
        '-selected': props.selected
      }
    ]"
    :tabindex="props.focusable ? 0 : null"
    @keypress.space.enter.prevent="listeners['select'] && listeners['select']()"
    @keydown.right="listeners['open'] && listeners['open']()"
    @keydown.left="listeners['close'] && listeners['close']()"
    @click="listeners['select'] && listeners['select']()"
    @focus="listeners['focus'] && listeners['focus']()"
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
  props: {
    opened: {
      type: Boolean,
      default: false
    },
    selected: {
      type: Boolean,
      default: false
    },
    focused: {
      type: Boolean,
      default: false
    },
    focusable: {
      type: Boolean,
      default: true
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
