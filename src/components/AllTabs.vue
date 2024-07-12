<template>
  <div>
    <nav class="bg-white px-8 pt-2 shadow-md">
      <div class="-mb-px flex">
        <a v-for="tab in tabsInternal"
           class="no-underline border-b-2 border-transparent uppercase tracking-wide font-bold text-xs py-3 mr-8 cursor-pointer "
           :class="{ 'text-teal-950': tab.isActive, 'text-gray-400': !tab.isActive }"

           :href="tab.href"
           @click="selectTab(tab)"
           :key="tab.name"
        >
          {{ tab.name }}
        </a>
      </div>
    </nav>

    <div class="tabs-details">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    tabs: Array,
  },
  data() {
    return {
      tabsInternal: this.tabs,
    }
  },

  methods: {
    selectTab(selectedTab) {
      console.log('selecting tab', selectedTab)
      this.tabsInternal.forEach(tab => {
        tab.isActive = (tab.name === selectedTab.name);
      });
    }
  }
}
</script>