<template>
  <all-tabs :tabs="tabs">
    <a-tab
        v-for="tab in tabs"
        :key="tab.id"
        :name="tab.name"
        :selected="tab.isActive"
    >
    </a-tab>
  </all-tabs>
  <ByBrowser v-show="tabs[0].isActive"/>
  <ByCountry v-show="tabs[1].isActive"/>
</template>
<script>

import AllTabs from "@/components/AllTabs.vue";
import ATab from "@/components/Tab.vue";
import ByBrowser from "@/components/ByBrowser.vue";
import ByCountry from "@/components/ByCountry.vue";

export default {
  name: 'App',
  components: {
    ByCountry,
    ByBrowser,
    AllTabs,
    ATab,
  },

  data() {
    return {
      tabs: [
        {id: 1, name: "By Browser", isActive: true},
        {id: 2, name: "By Country", isActive: false},
      ],
    }
  },
}
</script>