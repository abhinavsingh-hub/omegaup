<template>
  <div>
    <div v-if="hasTeamsGroups" class="mb-3 text-right">
      <a class="btn btn-primary mx-1" href="/teamsgroup/new/">
        {{ T.teamsGroupsCreateNew }}
      </a>
    </div>
    <div class="card">
      <div class="card-header mb-3">
        <div class="mb-2 text-right">
          <label>
            <input type="checkbox" v-model="showArchived" />
            Show archived
          </label>
        </div>
        <h3 class="card-title">{{ T.omegaupTitleTeamsGroups }}</h3>
      </div>

      <table v-if="hasTeamsGroups" class="table" data-table-teams-groups>
        <thead>
          <tr>
            <th>{{ T.teamsGroupTeamsGroupName }}</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="teamsGroup in visibleTeamsGroups"
            :key="`${teamsGroup.type}_${teamsGroup.alias}`"
          >
            <td>
              <strong>
                <a :href="teamsGroupUrl(teamsGroup)">
                  {{ teamsGroup.name }}
                </a>
              </strong>
            </td>
            <td>
              <a :href="teamsGroupEditUrl(teamsGroup)" :title="T.wordsEdit">
                <font-awesome-icon :icon="['fas', 'edit']" />
              </a>

              <button
                class="btn btn-link text-danger p-0 ml-2"
                @click="archiveGroup(teamsGroup)"
                title="Archive"
              >
                <font-awesome-icon :icon="['fas', 'archive']" />
              </button>
            </td>
          </tr>
        </tbody>
      </table>

      <div v-else class="text-center py-5">
        <font-awesome-icon
          :icon="['fas', 'users']"
          size="3x"
          class="mb-3 text-muted"
        />

        <h4 class="mb-2">
          {{ T.teamsGroupEmptyTitle }}
        </h4>

        <p class="text-muted mb-4">
          {{ T.teamsGroupEmptyDescription }}
        </p>

        <a class="btn btn-primary btn-lg" href="/teamsgroup/new/">
          {{ T.createTeamsGroup }}
        </a>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';
import { types } from '../../api_types';
import T from '../../lang';
import { library } from '@fortawesome/fontawesome-svg-core';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faEdit, faArchive } from '@fortawesome/free-solid-svg-icons';
library.add(faEdit, faArchive);
@Component({
  components: {
    FontAwesomeIcon,
  },
})
export default class TeamsGroupList extends Vue {
  @Prop() teamsGroups!: types.TeamsGroup[];
  T = T;
  teamsGroupUrl(teamsGroup: types.TeamsGroup): string {
    return `/teamsgroup/${teamsGroup.alias}/edit/#teams`;
  }
  teamsGroupEditUrl(teamsGroup: types.TeamsGroup): string {
    return `/teamsgroup/${teamsGroup.alias}/edit/#edit`;
  }
  get hasTeamsGroups(): boolean {
    return this.visibleTeamsGroups && this.visibleTeamsGroups.length > 0;
  }
  showArchived: boolean = false;
  get visibleTeamsGroups(): types.TeamsGroup[] {
    if (this.showArchived) return this.teamsGroups;
    return this.teamsGroups.filter(g => !g.is_archived);
  }
  async archiveGroup(teamsGroup: types.TeamsGroup) {
  if (!confirm('Are you sure you want to archive this team group?')) return;

  try {
    await this.$api.TeamsGroup.update({
      team_group_alias: teamsGroup.alias,
      is_archived: true,
    });

    teamsGroup.is_archived = true;
  } catch (e) {
    console.error(e);
  }
}
}
</script>
