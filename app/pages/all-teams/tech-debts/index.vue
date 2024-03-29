<template>
  <div>
    <AllTeamsTabs />
    <b-card-group v-for="(row, rowIndex) in rows" :key="rowIndex" deck>
      <b-card
        v-for="(item, itemIndex) in row"
        :key="itemIndex"
        bg-variant="dark"
        :header="item.team.name"
        text-variant="white"
        class="my-3"
      >
        <b-card
          v-for="itemPriority in item.priorities"
          :key="itemPriority.priority"
          no-body
          class="mt-1"
        >
          <b-card-header header-tag="header" class="p-1" role="tab">
            <b-button
              v-b-toggle="
                `accordion-${rowIndex}-${itemIndex}-${itemPriority.priority}`
              "
              block
              variant="info"
              >{{
                `${priorityName(
                  itemPriority.priority
                )} (${getItemPriorityTechDebtCount(itemPriority)})`
              }}</b-button
            >
          </b-card-header>
          <b-collapse
            :id="`accordion-${rowIndex}-${itemIndex}-${itemPriority.priority}`"
            visible
            :accordion="`accordion-${rowIndex}-${itemIndex}`"
            role="tabpanel"
          >
            <b-card-body>
              <ComponentTechDebtTable
                :components-and-tech-debts="itemPriority.componentsAndTechDebts"
              />
            </b-card-body>
          </b-collapse>
        </b-card>
      </b-card>
    </b-card-group>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import {
  BButton,
  BCard,
  BCardBody,
  BCardGroup,
  BCardHeader,
  BCollapse,
  VBToggle,
} from 'bootstrap-vue'
import { MetaInfo } from 'vue-meta'
import { Component, Team, Priority } from '~/types/component-catalog-service'
import AllTeamsTabs from '~/components/AllTeamsTabs.vue'
import ComponentTechDebtTable from '~/components/ComponentTechDebtTable.vue'
import { ComponentAndTechDebts } from '~/types/component-tech-debts'

interface Item {
  team: Team
  priorities: ItemPriority[]
}

interface ItemPriority {
  priority: Priority
  componentsAndTechDebts: ComponentAndTechDebts[]
}

export default Vue.extend({
  components: {
    AllTeamsTabs,
    'b-button': BButton,
    'b-card': BCard,
    'b-card-body': BCardBody,
    'b-card-group': BCardGroup,
    'b-card-header': BCardHeader,
    'b-collapse': BCollapse,
    ComponentTechDebtTable,
  },
  directives: {
    'b-toggle': VBToggle,
  },
  async asyncData({ $config }) {
    const teams = await fetch(
      `${$config.serviceBaseUrl}/v1/teams?fields=teams(id,name)`
    )
      .then((res) => res.json())
      .then((json) => json.teams as Team[])

    const components = await fetch(
      `${$config.serviceBaseUrl}/v1/components?fields=components(id,name,teams,techDebts)`
    )
      .then((res) => res.json())
      .then((json) => json.components as Component[])

    return {
      teams,
      components,
    }
  },
  data() {
    return {
      teams: [] as Team[],
      components: [] as Component[],
      priorities: [
        'very-high',
        'high',
        'medium',
        'low',
        undefined,
      ] as Priority[],
    }
  },
  computed: {
    rows() {
      const that = this
      const rows = [] as Array<Array<Item>>
      const teamCount = this.teams.length
      let startIndex = 0

      while (startIndex < teamCount) {
        const endIndex = Math.min(startIndex + 3, teamCount)
        const row = that.teams.slice(startIndex, endIndex).map(
          (team) =>
            ({
              team,
              priorities: getPriorities(team),
            } as Item)
        )
        rows.push(row)
        startIndex = endIndex
      }
      return rows

      function getPriorities(team: Team) {
        return that.priorities
          .map((priority) => {
            return {
              priority,
              componentsAndTechDebts: getComponentsAndTechDebts(team, priority),
            } as ItemPriority
          })
          .filter(
            (itemPriority) => itemPriority.componentsAndTechDebts.length > 0
          )
      }

      function getComponentsAndTechDebts(team: Team, priority: Priority) {
        return that.components
          .filter((component) => matchComponentTeam(component, team))
          .map((component) => {
            return {
              component,
              techDebts: getComponentTechDebts(component, priority),
            }
          })
          .filter(
            (componentAndTechDebts) =>
              componentAndTechDebts.techDebts.length > 0
          )
      }

      function matchComponentTeam(component: Component, team: Team) {
        return component.teams.some(
          (componentTeam) =>
            componentTeam.type !== 'previous' &&
            componentTeam.teamId === team.id
        )
      }

      function getComponentTechDebts(component: Component, priority: Priority) {
        if (!component.techDebts) {
          return []
        }

        return component.techDebts.filter(
          (techDebt) => techDebt.priority === priority
        )
      }
    },
  },
  methods: {
    priorityName(priority: Priority) {
      switch (priority) {
        case 'very-high':
          return 'Very High'
        case 'high':
          return 'High'
        case 'medium':
          return 'Medium'
        case 'low':
          return 'Low'
        case undefined:
          return 'Not Prioritized'
      }
    },
    getItemPriorityTechDebtCount(itemPriority: ItemPriority) {
      return itemPriority.componentsAndTechDebts
        .map((componentAndTechDebts) => componentAndTechDebts.techDebts.length)
        .reduce((previousValue, currentValue) => previousValue + currentValue)
    },
  },
  head(): MetaInfo {
    return {
      title: 'Component Catalog - All Teams - Tech Debts',
    }
  },
})
</script>
