<template>
	<doi-list-panel
		apiUrl="http://httpbin.org/get"
		id="previewDoiListPanel--articles"
		:items="previewItems"
		:itemsMax="previewItemsMax"
		title="Issue DOI Management"
		:isSelectable="true"
		:crossrefPluginEnabled="true"
		:isSubmission="false"
	/>
</template>
<script>
import DoiListPanel from '@/components/ListPanel/doi/DoiListPanel.vue';
// import submissions from '@/docs/data/submissions';
import issues from '@/docs/data/issues';

export default {
	components: {
		DoiListPanel
	},
	data() {
		// const submissionItems = [...submissions];
		const issueItems = [...issues];

		return {
			// previewItems: submissionItems,
			// previewItemsMax: submissionItems.length
			previewItems: issueItems,
			previewItemsMax: issueItems.length
		};
	}
};
</script>
