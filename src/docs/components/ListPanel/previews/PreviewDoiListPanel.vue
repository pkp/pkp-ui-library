<template>
	<doi-list-panel
		apiUrl=""
		id="previewDoiListPanel"
		:items="items"
		:itemsMax="itemsMax"
		title="DOI Management"
	/>
</template>
<script>
import DoiListPanel from '@/components/ListPanel/doi/DoiListPanel.vue';

export default {
	components: {
		DoiListPanel
	},
	data() {
		const items = [
			{
				dateExpire: '2022-02-03',
				datePosted: '2020-01-29',
				id: 1,
				author: {
					en_US: 'A. Author et al.',
					fr_CA: 'A. Author et al.',
					ar_AR: 'A. Author et al.'
				},
				title: {
					en_US: 'Title number one',
					fr_CA: 'Title number one',
					ar_AR: 'Title number one'
				}
			},
			{
				dateExpire: '2021-12-08',
				datePosted: '2019-12-12',
				id: 2,
				author: {
					en_US: 'B. Author et al.',
					fr_CA: 'B. Author et al.',
					ar_AR: 'B. Author et al.'
				},
				title: {
					en_US: 'Title number two',
					fr_CA: 'Title number two',
					ar_AR: 'Title number two'
				}
			},
			{
				dateExpire: '2021-12-08',
				datePosted: '2019-12-12',
				id: 3,
				author: {
					en_US: 'C. Author et al.',
					fr_CA: 'C. Author et al.',
					ar_AR: 'C. Author et al.'
				},
				title: {
					en_US: 'Title number three',
					fr_CA: 'Title number three',
					ar_AR: 'Title number three'
				}
			}
		];

		return {
			items: [...items],
			itemsMax: items.length
		};
	}
};
</script>
