<template>
  <div>
    <h1 class="text-info my-3">{{ area.name }} Area - Tech Debts</h1>
    <AreaTabs :area-id="area.id" />
    <TechDebtsView :components="area.components" />
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { MetaInfo } from 'vue-meta'
import { Area } from '~/types/component-catalog-service'
import AreaTabs from '~/components/AreaTabs.vue'
import TechDebtsView from '~/components/TechDebtsView.vue'

export default Vue.extend({
  components: {
    AreaTabs,
    TechDebtsView,
  },
  async asyncData({ $config, route, store }) {
    const area = await fetch(
      `${$config.serviceBaseUrl}/v1/areas/${route.params.areaId}?fields=area(id,name,components(id,name,typeId,tags,teams,platformId,techDebts))`
    )
      .then((res) => res.json())
      .then((json) => json.area as Area)

    store.commit('componentFilters/initialize', {
      components: area.components,
      route,
    })

    return {
      area,
    }
  },
  data() {
    return {
      area: {} as Area,
    }
  },
  head(): MetaInfo {
    return {
      title: `Component Catalog - ${this.area.name} Area - Tech Debts`,
    }
  },
})
</script>
