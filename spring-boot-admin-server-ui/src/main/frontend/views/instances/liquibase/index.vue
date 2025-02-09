<!--
  - Copyright 2014-2019 the original author or authors.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <sba-instance-section :loading="!hasLoaded" :error="error">
    <template v-for="(context, ctxName) in contexts" :key="ctxName">
      <template
        v-for="(report, name) in context.liquibaseBeans"
        :key="`${ctxName}-${name}`"
      >
        <sba-panel
          :title="`${ctxName}: ${name}`"
          class="change-set"
          :header-sticks-below="'#subnavigation'"
        >
          <table class="table is-hoverable w-full">
            <thead>
              <tr>
                <th v-text="$t('instances.liquibase.id')" />
                <th v-text="$t('instances.liquibase.execution')" />
                <th v-text="$t('instances.liquibase.description')" />
                <th v-text="$t('instances.liquibase.tag')" />
                <th v-text="$t('instances.liquibase.contexts')" />
                <th v-text="$t('instances.liquibase.labels')" />
              </tr>
            </thead>
            <tbody>
              <template
                v-for="changeSet in report.changeSets"
                :key="`${ctxName}-${name}-${changeSet.id}`"
              >
                <tr
                  class="is-selectable"
                  @click="
                    showDetails[changeSet.checksum]
                      ? delete showDetails[changeSet.checksum]
                      : (showDetails[changeSet.checksum] = true)
                  "
                >
                  <td v-text="changeSet.id" />
                  <td>
                    <span
                      class="tag"
                      :class="execClass(execType)"
                      v-text="changeSet.execType"
                    />
                  </td>
                  <td v-text="changeSet.description" />
                  <td v-text="changeSet.tag" />
                  <td v-text="changeSet.contexts.join(', ')" />
                  <td>
                    <span
                      v-for="label in changeSet.labels"
                      :key="`${ctxName}-${name}-${changeSet.id}-${label}`"
                      class="tag is-info"
                      v-text="label"
                    />
                  </td>
                </tr>
                <tr
                  v-if="showDetails[changeSet.checksum]"
                  :key="`${ctxName}-${name}-${changeSet.id}-details`"
                >
                  <td colspan="6">
                    <table class="table is-fullwidth">
                      <tr>
                        <th v-text="$t('instances.liquibase.changelog')" />
                        <td colspan="3" v-text="changeSet.changeLog" />
                        <th v-text="$t('instances.liquibase.author')" />
                        <td v-text="changeSet.author" />
                      </tr>
                      <tr>
                        <th v-text="$t('instances.liquibase.checksum')" />
                        <td v-text="changeSet.checksum" />
                        <th v-text="$t('instances.liquibase.comments')" />
                        <td colspan="3" v-text="changeSet.comments" />
                      </tr>
                      <tr>
                        <th
                          v-text="$t('instances.liquibase.execution_order')"
                        />
                        <td v-text="changeSet.orderExecuted" />
                        <th v-text="$t('instances.liquibase.execution_date')" />
                        <td v-text="changeSet.dateExecuted" />
                        <th v-text="$t('instances.liquibase.deployment_id')" />
                        <td v-text="changeSet.deploymentId" />
                      </tr>
                    </table>
                  </td>
                </tr>
              </template>
            </tbody>
          </table>
        </sba-panel>
      </template>
    </template>
  </sba-instance-section>
</template>

<script>
import Instance from '@/services/instance';
import { VIEW_GROUP } from '@/views/ViewGroup';
import SbaInstanceSection from '@/views/instances/shell/sba-instance-section';

export default {
  components: { SbaInstanceSection },
  props: {
    instance: {
      type: Instance,
      required: true,
    },
  },
  data: () => ({
    hasLoaded: false,
    error: null,
    contexts: null,
    showDetails: {},
  }),
  computed: {},
  created() {
    this.fetchLiquibase();
  },
  methods: {
    async fetchLiquibase() {
      this.error = null;
      try {
        const res = await this.instance.fetchLiquibase();
        this.contexts = res.data.contexts;
      } catch (error) {
        console.warn('Fetching Liquibase changeSets failed:', error);
        this.error = error;
      }
      this.hasLoaded = true;
    },
    execClass(execType) {
      switch (execType) {
        case 'EXECUTED':
          return 'is-success';
        case 'FAILED':
          return 'is-danger';
        case 'SKIPPED':
          return 'is-light';
        case 'RERAN':
        case 'MARK_RAN':
          return 'is-warning';
        default:
          return 'is-info';
      }
    },
  },
  install({ viewRegistry }) {
    viewRegistry.addView({
      name: 'instances/liquibase',
      parent: 'instances',
      path: 'liquibase',
      component: this,
      label: 'instances.liquibase.label',
      group: VIEW_GROUP.DATA,
      order: 900,
      isEnabled: ({ instance }) => instance.hasEndpoint('liquibase'),
    });
  },
};
</script>
å
