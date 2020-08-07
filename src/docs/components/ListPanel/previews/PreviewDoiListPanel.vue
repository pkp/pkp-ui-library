<template>
	<doi-list-panel
		apiUrl=""
		id="previewDoiListPanel"
		:items="items"
		:itemsMax="itemsMax"
		title="DOI Management"
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
		// const items = [
		// {
		// 	dateExpire: '2022-02-03',
		// 	datePosted: '2020-01-29',
		// 	id: 1,
		// 	author: {
		// 		en_US: 'A. Author et al.',
		// 		fr_CA: 'A. Author et al.',
		// 		ar_AR: 'A. Author et al.'
		// 	},
		// 	title: {
		// 		en_US: 'Title number one',
		// 		fr_CA: 'Title number one',
		// 		ar_AR: 'Title number one'
		// 	},
		// 	dois: [
		// 		{
		// 			id: 1,
		// 			type: 'Article',
		// 			doi: '10.2345/abc123'
		// 		},
		// 		{
		// 			id: 2,
		// 			type: 'Galley',
		// 			doi: '10.2345/def456'
		// 		}
		// 	]
		// },
		// 	{
		// 	}
		// ];

		return {
			items: [...submissions],
			itemsMax: submissions.length
		};
	}
};
</script>
