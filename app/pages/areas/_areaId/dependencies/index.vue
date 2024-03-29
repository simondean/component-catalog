<template>
  <div>
    <h1 class="text-info my-3">{{ area.name }} Area - Dependencies</h1>
    <AreaTabs :area-id="area.id" />
    <ComponentDependenciesView
      :all-components="allComponents"
      :components="area.components"
      :component-dependencies="summary.componentDependencies"
      :sub-component-dependencies="summary.subComponentDependencies"
    />
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { MetaInfo } from 'vue-meta'
import AreaTabs from '~/components/AreaTabs.vue'
import ComponentDependenciesView from '~/components/ComponentDependenciesView.vue'
import { Area, Component, Summary } from '~/types/component-catalog-service'

export default Vue.extend({
  components: {
    ComponentDependenciesView,
    AreaTabs,
  },
  async asyncData({ $config, route, store }) {
    const area = await fetch(
      `${$config.serviceBaseUrl}/v1/areas/${route.params.areaId}?fields=area(id,name,components(id,name,typeId,tags,description,notes,responsibilities,teams,platformId))`
    )
      .then((res) => res.json())
      .then((json) => json.area as Area)

    store.commit('componentFilters/initialize', {
      components: area.components,
      route,
    })

    const allComponents = await fetch(
      `${$config.serviceBaseUrl}/v1/components?fields=components(id,name,typeId,tags,description,notes,responsibilities,teams,platformId)`
    )
      .then((res) => res.json())
      .then((json) => json.components as Component[])

    const summary = await fetch(
      `${$config.serviceBaseUrl}/v1/summary?fields=summary(componentDependencies,subComponentDependencies)`
    )
      .then((res) => res.json())
      .then((json) => json.summary as Summary)

    return {
      area,
      allComponents,
      summary,
    }
  },
  data() {
    return {
      area: {} as Area,
      allComponents: [] as Component[],
      summary: {} as Summary,
    }
  },
  head(): MetaInfo {
    return {
      title: `Component Catalog - ${this.area.name} Area - Dependencies`,
    }
  },
})
</script>
