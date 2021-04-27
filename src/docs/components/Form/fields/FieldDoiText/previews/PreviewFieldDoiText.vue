<template>
	<field-doi-text v-bind="field" @change="change" />
</template>

<script>
import FieldDoiText from '@/components/Form/fields/FieldDoiText';
import PreviewFieldBase from '../../FieldBase/previews/PreviewFieldBase.vue';
import fieldBase from '../../../helpers/field-base';
import field from '../../../helpers/field-text-given-name';

export default {
	extends: PreviewFieldBase,
	components: {
		FieldDoiText
	},
	data() {
		return {
			field: {
				...fieldBase,
				...field,
				isRequired: false
			}
		};
	}
};
</script>
