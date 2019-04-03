<template>
  <div class="history">
    <div class="history__logs">
      <div v-if="!this.$store.state.history.commitInformation.isActive">
        <VueScrollbar class="history__logs__scrollbar">
          <div>
            <commitHistoryItem
              @click.native="gitShow(log.hash)"
              :data="log"
              v-for="log in repositoryLogs"
              :key="log.hash"
            />
          </div>
        </VueScrollbar>
      </div>
      <div v-else class="history__logs__detail">
        <div class="history__logs__detail__buttons">
          <div @click="toggleCommitDetail()" class="history__logs__detail__buttons__back">Back</div>
          <div
            @click="exportCommitDetail()"
            class="history__logs__detail__buttons__export d-flex ml-auto"
          >
            <fileIcon/>
          </div>
        </div>
        <commitInformation/>
      </div>
    </div>
    <div class="history__preview">
      <div v-if="this.$store.state.history.filePreview.isActive">
        <commitHistoryPreview/>
      </div>
      <div v-else>No content to show</div>
    </div>
  </div>
</template>

<script>
import git from "simple-git/promise"
import commitHistoryItem from "../../components/commit/commitHistoryItem"
import commitInformation from "../../components/commit/commitInformation"
import commitHistoryPreview from "../../components/commit/commitHistoryPreview"
import fileIcon from "../../components/icon/file"
import * as Sentry from "@sentry/electron"
import VueScrollbar from "vue2-scrollbar"

export default {
	name: "history",
	components: {
		commitHistoryItem,
		commitInformation,
		commitHistoryPreview,
		VueScrollbar,
		fileIcon
	},
	computed: {
		repositoryLogs() {
			return this.$store.getters["history/allLogs"]
		},
		currentRepository() {
			return this.$store.getters["workspace/currentRepository"]
		}
	},
	methods: {
		async getLogs() {
			const gitLog = await git(this.currentRepository.path).log()
			try {
				this.$store.dispatch("history/getRepositoryLogs", {
					logs: gitLog.all
				})
			} catch (error) {
				Sentry.captureException(error)
				let errorMessage = "Unable to fetch logs."
				console.log(errorMessage)
				Sentry.captureMessage(errorMessage, gitLog)
			}
		},
		toggleCommitDetail() {
			this.$store.commit("history/toggleCommitInformation")
			this.$store.commit({
				type: "history/toggleFilePreview",
				isActive: false
			})
		},
		gitShow(hash) {
			this.toggleCommitDetail()
			this.$store.dispatch({
				type: "history/updateCommitInformationMeta",
				commit_hash: hash
			})
		},
		exportCommitDetail() {
			this.$store.dispatch("model/showExportCommitData")
		}
	},
	mounted() {
		this.getLogs()
	}
}
</script>

<style lang='sass'>
	.history
		display: flex
		flex-direction: row

		&__logs
			border-right: 1px solid #DEE0E3
			width: 300px

			&__detail

				&__buttons
					display: flex
					padding: 10px
					flex-direction: row
					border-bottom: 1px solid #DEE0E3

					&__back
						font-size: 10px
						cursor: pointer
						color: #6C6F75
						padding: 2px 6px
						background-color: #DEE0E3
						border-radius: 10px

					&__export
						cursor: pointer

						svg
							stroke: #6C6F75
							width: 18px
							height: 18px

			&__scrollbar
				max-height: 90vh

		&__preview
			padding: 10px
</style>