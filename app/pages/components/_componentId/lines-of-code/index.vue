<template>
  <div>
    <h1 class="text-info my-3">{{ component.name }} - Lines of Code</h1>
    <ComponentTabs :component-id="component.id" />

    <div class="row">
      <div class="col-sm-6">
        <b-card title="Total Lines of Code">
          <b-list-group>
            <b-list-group-item :variant="linesOfCodeCountVariant">
              <span class="display-1">
                <FormattedNumber :value="linesOfCodeCount" />
              </span>
              lines of code{{ linesOfCodeCount === 1 ? '' : 's' }}
            </b-list-group-item>
          </b-list-group>
        </b-card>
      </div>
      <div class="col-sm-6">
        <b-card title="Lines of Code by File Extension">
          <b-list-group>
            <b-list-group-item
              v-for="fileExtensionCount in fileExtensionCounts"
              :key="fileExtensionCount.fileExtension"
              class="d-flex justify-content-between align-items-center"
            >
              {{ fileExtensionCount.fileExtension }}
              <b-badge variant="primary" pill>
                <FormattedNumber :value="fileExtensionCount.count" />
              </b-badge>
            </b-list-group-item>
          </b-list-group>
        </b-card>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { MetaInfo } from 'vue-meta'
import { BBadge, BCard, BListGroup, BListGroupItem } from 'bootstrap-vue'
import {
  Component,
  FileExtensionCount,
} from '~/types/component-catalog-service'
import ComponentTabs from '~/components/ComponentTabs.vue'
import FormattedNumber from '~/components/FormattedNumber.vue'

export default Vue.extend({
  components: {
    'b-badge': BBadge,
    'b-card': BCard,
    'b-list-group': BListGroup,
    'b-list-group-item': BListGroupItem,
    ComponentTabs,
    FormattedNumber,
  },
  async asyncData({ $config, route }) {
    const component = await fetch(
      `${$config.serviceBaseUrl}/v1/components/${route.params.componentId}?fields=component(id,name,linesOfCode)`
    )
      .then((res) => res.json())
      .then((json) => json.component as Component)

    return {
      component,
    }
  },
  data() {
    return {
      component: {} as Component,
    }
  },
  computed: {
    linesOfCodeCount(): number {
      return this.component.linesOfCode?.count ?? 0
    },
    linesOfCodeCountVariant(): string {
      return this.linesOfCodeCount > 100000 ? 'danger' : 'success'
    },
    fileExtensionCounts(): FileExtensionCount[] {
      if (!this.component.linesOfCode) {
        return []
      }

      return this.component.linesOfCode.fileExtensionCounts.map(
        ({ fileExtension, count }) => {
          return {
            fileExtension: fileExtension || 'No file extension',
            count,
          } as FileExtensionCount
        }
      )
    },
  },
  head(): MetaInfo {
    return {
      title: `Component Catalog - ${this.component.name} - Lines of Code`,
    }
  },
})
</script>
