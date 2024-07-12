<template>
  <div v-if="isActive">
    <slot></slot>
  </div>
</template>

<script>
import {ref} from "vue";

export default {
  name: 'a-tab',
  setup(props) {
    const isActive = ref(props.selected);
    const name = ref(props.name);

    return {
      isActive,
      name
    }
  },
  computed: {
    href() {
      return '#' + this.name.toLowerCase().replace(/ /g, '-');
    }
  },
  mounted() {
    this.isActive = this.selected;
  },
}
</script>