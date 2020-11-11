<template>
	<doi-list-panel
		apiUrl="http://httpbin.org/get"
		id="previewDoiListPanel--articles"
		:items="articleItems"
		:itemsMax="articleItemMax"
		title="Article DOI Management"
		:isSelectable="true"
		:crossrefPluginEnabled="true"
	/>
</template>
<script>
import DoiListPanel from '@/components/ListPanel/doi/DoiListPanel.vue';
import submissions from '@/docs/data/submissions';

export default {
	components: {
		DoiListPanel
	},
	data() {
		const submissionItems = [...submissions];
		// const issueItems = [
		// 	{
		// 		id: 1,
		// 		objectType: 'issue',
		// 		object: {
		// 			issueId: 1,
		// 			volume: 1,
		// 			number: 2,
		// 			year: 2020,
		// 			title: 'Test Issue',
		// 			'pub-id::doi': '10.9876/123abc',
		// 			published: 1
		// 		}
		// 	}
		// ];

		return {
			articleItems: submissionItems,
			articleItemMax: submissionItems.length
			// issueItems: issueItems,
			// issueItemsMax: issueItems.length
		};
	}
};
</script>
