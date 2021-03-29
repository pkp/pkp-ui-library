<template>
	<doi-list-panel
		apiUrl="http://httpbin.org/get"
		id="previewDoiListPanel--articles"
		:items="previewItems"
		:itemsMax="previewItemsMax"
		title="Article DOIs"
		:isSelectable="true"
		:crossrefPluginEnabled="true"
		:isSubmission="true"
		:issueFilter="issueFilter"
		:enabledPublishingObjects="enabledPublishingObjects"
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
			issueFilter: {
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
