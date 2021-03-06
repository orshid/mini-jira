<template>
  <v-row v-if="activeIssue">
    <v-col sm="12">
      <div class="d-flex">
        <v-btn class="mr-6" small fab color="success" @click="redirectToEdit">
          <v-icon>
            edit
          </v-icon>
        </v-btn>
        <autocomplete :list="issuesList" :label="'Link issue'" class="mb-6 mw-25" @onChange="linkIssues"></autocomplete>
      </div>
      <v-card min-height="200">
        <v-card-title>
          <h2>{{ activeIssue.id }} - {{ activeIssue.summary }}</h2>
        </v-card-title>
        <v-card-text>
          <p>
            {{ activeIssue.description }}
          </p>
        </v-card-text>
        <hr />
        <v-card-title>
          <h3>
            Related issues:
          </h3>
        </v-card-title>
        <v-card-text @click="refreshList">
          <v-list>
            <issues-list
              v-for="item of relatedIssues"
              :id="item.id"
              :key="item.id"
              :title="item.summary"
              :link="`/issues/${item.id}`"
              :status="item.status"
              :project-id="item.projectId"
              no-icon
              :unlink="true"
            />
          </v-list>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'
import { namespace } from 'vuex-class'
import IssuesList from '@/components/issues-list.vue'
import Autocomplete from '@/components/autocomplete.vue'
import { Issue } from '@/types/issues/types'
const issues = namespace('issues')

@Component({
  components: {
    Autocomplete,
    IssuesList,
  },
})
export default class SingleIssue extends Vue {
  @issues.Action('getSingleIssue') getSingleIssue: (issueId: string) => void
  @issues.Action('getRelatedIssues') getRelatedIssues: ({
    issueId,
    projectId,
  }: {
    issueId: string
    projectId: string
  }) => void
  @issues.Action('addRelation') addRelation: ({
    issueId,
    projectId,
    relatedIssueId,
  }: {
    issueId: string
    projectId: string
    relatedIssueId: string
  }) => void
  @issues.Action('toggleLoading') toggleLoading: void
  @issues.Getter('activeIssue') activeIssue: Issue
  @issues.Getter('relatedIssues') relatedIssues: Issue[]
  @issues.Getter('issuesList') issuesList: Issue[]

  mounted() {
    this.refreshList()
  }

  refreshList() {
    this.getSingleIssue(this.$route.params.issueId)
    this.getRelatedIssues({
      issueId: this.$route.params.issueId,
      projectId: this.activeIssue.projectId,
    })
  }

  linkIssues(data: Issue) {
    this.addRelation({
      issueId: this.$route.params.issueId,
      projectId: this.activeIssue.projectId,
      relatedIssueId: data.id,
    })
  }

  redirectToEdit() {
    this.$router.push(`/issues/${this.activeIssue.projectId}/${this.activeIssue.id}/edit`)
  }
}
</script>
