<!--
  - Copyright 2014-2018 the original author or authors.
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
  <section class="section">
    <div class="container">
      <div v-if="error" class="message is-warning">
        <div class="message-body">
          <strong>
            <font-awesome-icon class="has-text-warning" icon="exclamation-triangle" />
            Server connection failed.
          </strong>
          <p v-text="error.message" />
        </div>
      </div>
      <div class="level applications-stats">
        <div class="level-item has-text-centered">
          <div>
            <p class="heading">应用</p>
            <p class="title" v-text="applicationsCount">1</p>
          </div>
        </div>
        <div class="level-item has-text-centered">
          <div>
            <p class="heading">实例</p>
            <p class="title" v-text="instancesCount">1</p>
          </div>
        </div>
        <div class="level-item has-text-centered">
          <div v-if="downCount === 0">
            <p class="heading">状态</p>
            <p class="title has-text-success">all up</p>
          </div>
          <div v-else>
            <p class="heading">下线</p>
            <p class="title has-text-danger" v-text="downCount" />
          </div>
        </div>
      </div>
      <div class="application-group" v-for="group in statusGroups" :key="group.status">
        <p class="heading" v-text="group.status" />
        <applications-list :applications="group.applications" :selected="selected" />
      </div>
      <div v-if="statusGroups.length === 0">
        <p class="is-muted">No applications registered.</p>
      </div>
    </div>
  </section>
</template>

<script>
  import groupBy from 'lodash/groupBy';
  import sortBy from 'lodash/sortBy';
  import transform from 'lodash/transform';
  import applicationsList from './applications-list';
  import handle from './handle';

  export default {
    props: {
      applications: {
        type: Array,
        default: () => [],
      },
      error: {
        type: Error,
        default: null
      },
      selected: {
        type: String,
        default: null
      }
    },
    // eslint-disable-next-line vue/no-unused-components
    components: {applicationsList},
    computed: {
      statusGroups() {
        const byStatus = groupBy(this.applications, application => application.status);
        const list = transform(byStatus, (result, value, key) => {
          result.push({status: key, applications: sortBy(value, [application => application.name])})
        }, []);
        return sortBy(list, [item => item.status]);
      },
      applicationsCount() {
        return this.applications.length;
      },
      instancesCount() {
        return this.applications.reduce((current, next) => current + next.instances.length, 0);
      },
      downCount() {
        return this.applications.reduce((current, next) => {
          return current + (next.instances.filter(instance => instance.statusInfo.status !== 'UP').length);
        }, 0);
      }
    },
    install({viewRegistry}) {
      viewRegistry.addView({
        path: '/applications/:selected?',
        props: true,
        name: 'applications',
        label: '应用',
        handle,
        order: 0,
        component: this
      });
      viewRegistry.addRedirect('/', 'applications');
    }
  };
</script>


<style lang="scss">
  @import "~@/assets/css/utilities";

  .application-group {
    margin: $gap 0;
  }
</style>
