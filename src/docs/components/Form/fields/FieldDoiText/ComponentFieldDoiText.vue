<script>
import Component from '@/docs/Component.vue';
import PreviewFieldDoiText from './previews/PreviewFieldDoiText.vue';
import PreviewFieldDoiTextTemplate from '!raw-loader!./previews/PreviewFieldDoiText.vue';
import readme from '!raw-loader!./readme.md';

export default {
	extends: Component,
	data() {
		return {
			name: 'FieldDoiText',
			parentRoute: 'Form/fields',
			readme: readme,
			examples: [
				{
					component: PreviewFieldDoiText,
					name: 'Base',
					template: this.extractTemplate(PreviewFieldDoiTextTemplate)
				}
			]
		};
	}
};
</script>
