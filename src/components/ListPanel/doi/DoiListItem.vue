<template>
	<div class="listPanel__item--doi">
		<div class="listPanel__itemSummary">
			<label class="listPanel__selectWrapper">
				<div class="listPanel__selector">
					<input
						type="checkbox"
						name="submissions[]"
						:value="item.id"
						v-model="isSelected"
					/>
				</div>
			</label>
			<div class="listPanel__itemIdentity">
				<div class="listPanel__itemTitle">
					{{ localize(item.author) }}
				</div>
				<div class="listPanel__itemSubtitle">
					{{ localize(item.title) }}
				</div>
			</div>

			<div class="listPanel__itemActions">
				<expander
					:isExpanded="isExpanded"
					:itemName="item.id.toString()"
					@toggle="isExpanded = !isExpanded"
				/>
			</div>
		</div>

		<div
			v-if="isExpanded"
			class="listPanel__itemExpanded listPannel__itemExpanded--doi"
		>
			Many more details here.
		</div>
	</div>
</template>

<script>
import Expander from '@/components/Expander/Expander.vue';
export default {
	name: 'DoiListItem',
	components: {
		Expander
	},
	props: {
		item: {
			type: Object,
			required: true
		},
		selected: {
			type: Array,
			default() {
				return [];
			}
		}
	},
	data() {
		return {
			isExpanded: false,
			isSelected: false
		};
	},
	methods: {
		/**
		 * Toggles item as selected and notifies DoiListPanel
		 */
		toggleSelected() {
			// if (isSelected != null) {
			// 	this.isSelected = isSelected;
			// }
			// Notifies DoiListPanel of selection toggle
			this.$emit('toggleDoiSelected', this.item.id, this.isSelected);
		}
	},
	watch: {
		isSelected(newVal, oldVal) {
			this.toggleSelected();
		},
		selected(newVal, oldVal) {
			if (this.selected.includes(this.item.id)) {
				this.isSelected = true;
			} else {
				this.isSelected = false;
			}
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';
</style>
