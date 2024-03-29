<template>
  <div>
    <b-alert show="30" dismissible variant="info" class="my-3">
      The dependencies on this page come from a combination of Zipkin traces and
      dependencies manually specified in the catalog's metadata YAML files. The
      visualisation is not out-of-the-box Zipkin functionality and is bespoke to
      the Component Catalog. The other data on the page like descriptions,
      teams, tags etc. come from the Component Catalog and not from Zipkin.
    </b-alert>
    <div class="graph">
      <ComponentDependencyGraph
        id="component-dependency-graph"
        :dependencies="dependencies"
        dependency-type="scope-related"
        :zoom="zoom"
        :selected-component-id="selectedComponentId"
        :scoped-component-ids="filteredComponentIds"
        :fixed-scope="true"
        :scope-related-radius="parseInt(selectedScopeRelatedRadius, 10)"
        @networkChange="networkChange"
        @selectedNodeChange="selectedNodeChange"
      />
    </div>
    <div class="panel">
      <b-alert show="60" dismissible variant="info">
        Hover over or click a component's dot in the dependencies diagram to see
        more information about it
      </b-alert>

      <ComponentPanel v-if="selectedComponent" :component="selectedComponent" />

      <div v-else class="my-3">
        <ComponentFilters :components="components" />

        <b-form-group
          label-cols="6"
          label-size="sm"
          label="Radius:"
          label-for="graph-scope-related-radius"
          class="my-3 mr-2"
        >
          <b-form-select
            id="graph-scope-related-radius"
            v-model="selectedScopeRelatedRadius"
            :options="scopeRelatedRadiusOptions"
            size="sm"
          />
        </b-form-group>

        <b-form-group
          label-cols="6"
          label-size="sm"
          label="Zoom:"
          label-for="graph-zoom"
          class="my-3 mr-2"
        >
          <b-form-select
            id="graph-zoom"
            v-model="zoom"
            :options="zoomOptions"
            size="sm"
          />
        </b-form-group>

        <b-form-checkbox
          id="detailed-dependencies"
          v-model="detailed"
          :value="true"
          :unchecked-value="false"
        >
          Detailed dependencies
        </b-form-checkbox>
      </div>
    </div>

    <div
      v-if="excludedComponents && excludedComponents.length > 0"
      class="excluded-components my-3"
    >
      <h2>Excluded Components</h2>
      <ComponentTable :components="excludedComponents" />
    </div>
  </div>
</template>

<style scoped>
.graph {
  float: left;
  width: calc(100% - 275px);
  overflow-x: scroll;
  scrollbar-color: #444 #111;
}

.panel {
  float: right;
  width: 250px;
}

.excluded-components {
  float: left;
  width: calc(100% - 275px);
}
</style>

<script lang="ts">
import Vue, { PropType } from 'vue'
import { BAlert, BFormCheckbox, BFormGroup, BFormSelect } from 'bootstrap-vue'
import {
  Component,
  SummaryComponentDependencies,
  SummaryComponentDependencyNode,
  SummarySubComponentDependencies,
  SummarySubComponentDependencyNode,
} from '~/types/component-catalog-service'
import { Network } from '~/types/component-dependency-graph'
import { intRange } from '~/src/arrayUtils'
import ComponentDependencyGraph from '~/components/ComponentDependencyGraph.vue'
import ComponentPanel from '~/components/ComponentPanel.vue'
import ComponentTable from '~/components/ComponentTable.vue'
import ComponentFilters from '~/components/ComponentFilters.vue'

interface Option {
  value: string | undefined
  text: string
}

export default Vue.extend({
  components: {
    'b-alert': BAlert,
    'b-form-checkbox': BFormCheckbox,
    'b-form-group': BFormGroup,
    'b-form-select': BFormSelect,
    ComponentDependencyGraph,
    ComponentFilters,
    ComponentPanel,
    ComponentTable,
  },
  props: {
    components: {
      type: Array as PropType<Component[]>,
      required: true,
    },
    componentDependencies: {
      type: Object as PropType<SummaryComponentDependencies>,
      required: true,
    },
    subComponentDependencies: {
      type: Object as PropType<SummarySubComponentDependencies>,
      required: true,
    },
    allComponents: {
      type: Array as PropType<Component[]>,
      required: true,
    },
    selectedComponentId: {
      type: String,
      default: undefined,
    },
    scopeRelatedRadius: {
      type: Number,
      default: 1,
    },
  },
  data() {
    return {
      selectedNode: undefined as
        | SummaryComponentDependencyNode
        | SummarySubComponentDependencyNode
        | undefined,
      selectedComponent: undefined as Component | undefined,
      selectedScopeRelatedRadius: this.scopeRelatedRadius,
      zoom: 100,
      detailed: false,
      network: undefined as Network | undefined,
    }
  },
  computed: {
    filteredComponentIds(): Component[] {
      return this.$store.state.componentFilters.filteredComponentIds
    },
    dependencies():
      | SummaryComponentDependencies
      | SummarySubComponentDependencies {
      return this.detailed
        ? this.subComponentDependencies
        : this.componentDependencies
    },
    scopeRelatedRadiusOptions(): Option[] {
      return intRange(0, 11).map((value) => {
        return {
          value: value.toString(),
          text: value.toString(),
        }
      })
    },
    zoomOptions(): Option[] {
      const zoomOptions = [25, 50, 75, 100, 125, 150, 200, 400]
      return zoomOptions.map((zoomOption) => {
        return {
          value: zoomOption.toString(),
          text: `${zoomOption}%`,
        }
      })
    },
    excludedComponents(): Component[] | undefined {
      if (!this.network) {
        return undefined
      }

      const componentIds = this.network.nodes.map((node) => {
        return node.node.componentId
      })

      return this.components.filter((component) => {
        return !componentIds.includes(component.id)
      })
    },
  },
  methods: {
    findComponent(id: string): Component | undefined {
      return this.allComponents.find((component) => component.id === id)
    },
    networkChange(network: Network): void {
      this.network = network
    },
    selectedNodeChange(
      _: Event,
      node:
        | SummaryComponentDependencyNode
        | SummarySubComponentDependencyNode
        | undefined
    ): void {
      if (node) {
        this.selectedNode = node
        this.selectedComponent = this.findComponent(node.componentId)
      } else {
        this.selectedNode = undefined
        this.selectedComponent = undefined
      }
    },
  },
})
</script>
