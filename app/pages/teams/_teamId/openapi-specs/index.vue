<template>
  <div>
    <h1 class="text-info my-3">{{ team.name }} Team - OpenAPI Specs</h1>
    <TeamTabs :team-id="team.id" />
    <OpenApiSpecsView :components="team.components" />
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { MetaInfo } from 'vue-meta'
import { Team } from '~/types/component-catalog-service'
import OpenApiSpecsView from '~/components/OpenApiSpecsView.vue'
import TeamTabs from '~/components/TeamTabs.vue'

export default Vue.extend({
  components: {
    OpenApiSpecsView,
    TeamTabs,
  },
  async asyncData({ $config, route }) {
    const team = await fetch(
      `${$config.serviceBaseUrl}/v1/teams/${route.params.teamId}?fields=team(id,name,components(id,name,typeId,tags,teams,platformId,openApiSpecs))`
    )
      .then((res) => res.json())
      .then((json) => json.team as Team)

    return {
      team,
    }
  },
  data() {
    return {
      team: {} as Team,
    }
  },
  head(): MetaInfo {
    return {
      title: `Component Catalog - ${this.team.name} Team - OpenAPI Specs`,
    }
  },
})
</script>
