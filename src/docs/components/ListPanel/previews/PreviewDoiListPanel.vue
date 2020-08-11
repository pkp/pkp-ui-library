<template>
	<doi-list-panel
		apiUrl=""
		id="previewDoiListPanel--articles"
		:items="articleItems"
		:itemsMax="articleItemMax"
		title="Article DOI Management"
		:isSelectable="true"
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
		const submissionItems = [
			{
				id: 1,
				objectType: 'submission',
				object: submissions[0]
			},
			{
				id: 2,
				objectType: 'submission',
				object: submissions[1]
			},
			{
				id: 3,
				objectType: 'submission',
				object: submissions[2]
			},
			{
				id: 4,
				objectType: 'submission',
				object: submissions[3]
			},
			{
				id: 5,
				objectType: 'submission',
				object: submissions[4]
			},
			{
				id: 6,
				objectType: 'submission',
				object: submissions[5]
			},
			{
				id: 7,
				objectType: 'submission',
				object: submissions[6]
			},
			{
				id: 8,
				objectType: 'submission',
				object: submissions[7]
			},
			{
				id: 9,
				objectType: 'submission',
				object: submissions[8]
			},
			{
				id: 10,
				objectType: 'submission',
				object: submissions[9]
			}
		];
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
