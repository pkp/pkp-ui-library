<template>
	<doi-list-panel
		apiUrl="http://httpbin.org/get"
		id="previewDoiListPanel--articles"
		:items="previewItems"
		:itemsMax="previewItemsMax"
		title="Article DOIs"
		:crossrefPluginEnabled="true"
		:isSubmission="true"
		:issueFilter="issueFilter"
		hasDOIs="enabledPublishingObjects"
	/>
</template>
<script>
import DoiListPanel from '@/components/ListPanel/doi/DoiListPanel.vue';
import submissions from '@/docs/data/submissions';
// import issues from '@/docs/data/issues';
import fieldBase from '@/docs/components/Form/helpers/field-base';
import fieldBaseAutosuggest from '@/docs/components/Form/helpers/field-autosuggest';

export default {
	components: {
		DoiListPanel
	},
	data() {
		const submissionItems = [...submissions];
		// const issueItems = [...issues];

		return {
			previewItems: submissionItems,
			previewItemsMax: submissionItems.length,
			// previewItems: issueItems,
			// previewItemsMax: issueItems.length,
			filters: [
				// For submissions
				{
					filters: [
						{
							title: 'Issues',
							param: 'issueIds',
							value: [],
							component: 'field-select-issues',
							autosuggestProps: {
								...fieldBase,
								...fieldBaseAutosuggest,
								apiUrl: '/issues.json',
								name: 'issueIds',
								label: 'Issues',
								selectedLabel: 'Assigned'
							},
							filterType: 'pkp-filter-autosuggest'
						}
					]
				},
				// For submissions
				{
					heading: 'Publication Status',
					filters: [
						{
							title: 'Published',
							param: 'status',
							value: `${pkp.const.STATUS_PUBLISHED}`
						},
						{
							title: 'Unpublished',
							param: 'status',
							value: `${pkp.const.STATUS_QUEUED}, ${pkp.const.STATUS_SCHEDULED}` // '1,5'
						}
					]
				},
				{
					heading: 'CrossRef Deposit Status',
					filters: [
						{
							title: 'Not deposited',
							param: 'crossrefStatus',
							value: 'notDeposited'
						},
						{
							title: 'Active',
							param: 'crossrefStatus',
							value: 'registered'
						},
						{
							title: 'Failed',
							param: 'crossrefStatus',
							value: 'failed'
						},
						{
							title: 'Marked Active',
							param: 'crossrefStatus',
							value: 'markedRegistered'
						}
					]
				},
				// For issues
				{
					heading: 'Publication Status',
					filters: [
						{
							title: 'Published',
							param: 'isPublished',
							value: '1'
						},
						{
							title: 'Unpublished',
							param: 'isPublished',
							value: '0'
						}
					]
				}
			],
			enabledPublishingObjects: {
				issues: true,
				publications: true,
				representations: true
			}
		};
	},
	created() {
		/**
		 * Add required locale keys
		 */
		pkp.localeKeys['publication.status.unpublished'] = 'Unpublished';
	}
};
</script>
