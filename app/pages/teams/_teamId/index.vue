<template>
  <div>
    <h1 class="text-info my-3">{{ team.name }} Team</h1>
    <TeamTabs :team-id="team.id" />

    <b-card v-if="team.areaId" title="Area" class="my-3">
      <AreaName :area="{ id: team.areaId }" />
    </b-card>

    <b-card
      v-if="team.links && team.links.length > 0"
      title="Links"
      class="my-3"
    >
      <Links :links="team.links" />
    </b-card>

    <b-card v-if="team.description" title="Description" class="my-3">
      <Markdown :markdown="team.description" />
    </b-card>

    <b-card v-if="team.emailAddress" title="Email Address" class="my-3">
      <EmailAddress :email-address="team.emailAddress" />
    </b-card>

    <b-card v-if="team.notes" title="Notes" class="my-3">
      <Markdown :markdown="team.notes" :toc="true" />
    </b-card>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { MetaInfo } from 'vue-meta'
import { BCard } from 'bootstrap-vue'
import { Team } from '~/types/component-catalog-service'
import AreaName from '~/components/AreaName.vue'
import EmailAddress from '~/components/EmailAddress.vue'
import Links from '~/components/Links.vue'
import Markdown from '~/components/Markdown.vue'
import TeamTabs from '~/components/TeamTabs.vue'

export default Vue.extend({
  components: {
    AreaName,
    'b-card': BCard,
    EmailAddress,
    Links,
    Markdown,
    TeamTabs,
  },
  async asyncData({ $config, route }) {
    const team = await fetch(
      `${$config.serviceBaseUrl}/v1/teams/${route.params.teamId}?fields=team(id,areaId,name,emailAddress,description,notes,links)`
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
      title: `Component Catalog - ${this.team.name} Team`,
    }
  },
})
</script>
