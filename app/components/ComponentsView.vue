<template>
  <div>
    <div class="main">
      <b-card
        :title="`${componentCount} Component${componentCount === 1 ? '' : 's'}`"
      >
        <b-card-text>
          <ComponentTable :components="filteredComponents" />
        </b-card-text>
      </b-card>
    </div>

    <div class="panel">
      <ComponentFilters :components="components" />
    </div>
  </div>
</template>

<style scoped>
.main {
  float: left;
  width: calc(100% - 275px);
}

.panel {
  float: right;
  width: 250px;
}
</style>

<script lang="ts">
import Vue, { PropType } from 'vue'
import { BCard, BCardText } from 'bootstrap-vue'
import { Component } from '~/types/component-catalog-service'
import ComponentFilters from '~/components/ComponentFilters.vue'
import ComponentTable from '~/components/ComponentTable.vue'

export default Vue.extend({
  components: {
    'b-card': BCard,
    'b-card-text': BCardText,
    ComponentFilters,
    ComponentTable,
  },
  props: {
    components: {
      type: Array as PropType<Component[]>,
      default: undefined,
    },
  },
  computed: {
    filteredComponents(): Component[] {
      return this.$store.state.componentFilters.filteredComponents
    },
    componentCount(): number {
      return this.filteredComponents.length
    },
  },
})
</script>
