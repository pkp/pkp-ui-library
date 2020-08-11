<template>
	<div class="doiListPanel">
		<slot>
			<list-panel :items="items">
				<template slot="header">
					<pkp-header>
						<h2>{{ title }}</h2>
						<spinner v-if="isLoading" />
						<template slot="actions">
							<search
								:searchPhrase="searchPhrase"
								@search-phrase-changed="setSearchPhrase"
							/>
						</template>
					</pkp-header>

					<div class="doiListPanel__options">
						<div v-if="isSelectable" class="listPanel__selectAllWrapper">
							<input
								type="checkbox"
								:id="id + '-selectAll'"
								:checked="isSelectAllOn"
								@click="toggleSelectAll"
							/>
							<label class="listPanel__selectAllLabel" :for="id + '-selectAll'">
								Select All
							</label>
						</div>

						<span class="doiListPanel__options--expandAll">
							<pkp-button @click="toggleExpandAll">
								<!-- TODO: Localize text -->
								{{ isExpandAllOn ? 'Collapse all' : 'Expand all' }}
							</pkp-button>
						</span>
					</div>
				</template>

				<template v-slot:item="{item}">
					<slot name="item" :item="item">
						<doi-list-item
							:key="item.id"
							:item="item"
							@toggleDoiSelected="toggleDoiSelected"
							:selected="selected"
							:isSelectable="isSelectable"
							:isExpandAllOn="isExpandAllOn"
						/>
					</slot>
				</template>

				<pagination
					v-if="lastPage > 1"
					slot="footer"
					:currentPage="currentPage"
					:isLoading="isLoading"
					:lastPage="lastPage"
					@set-page="setPage"
				/>
			</list-panel>
		</slot>
	</div>
</template>

<script>
import ListPanel from '@/components/ListPanel/ListPanel.vue';
import Pagination from '@/components/Pagination/Pagination.vue';
import PkpHeader from '@/components/Header/Header.vue';
import Search from '@/components/Search/Search.vue';
import fetch from '@/mixins/fetch';
import DoiListItem from '@/components/ListPanel/doi/DoiListItem';

export default {
	components: {
		DoiListItem,
		ListPanel,
		Pagination,
		PkpHeader,
		Search
	},
	mixins: [fetch],
	props: {
		id: {
			type: String,
			required: true
		},
		items: {
			type: Array,
			default() {
				return [];
			}
		},
		itemsMax: {
			type: Number,
			default() {
				return 0;
			}
		},
		title: {
			type: String,
			required: true
		},
		isSelectable: {
			type: Boolean,
			default() {
				return true;
			}
		}
	},
	data() {
		return {
			isExpandAllOn: false,
			isSelectAllOn: false,
			selected: []
		};
	},
	methods: {
		/**
		 * Set the list of items
		 *
		 * @see @/mixins/fetch.js
		 * @param {Array} items
		 * @param {Number} itemsMax
		 */
		setItems(items, itemsMax) {
			this.$emit('set', this.id, {
				items,
				itemsMax
			});
		},
		/**
		 * Toggle DoiListItem status in DoiListPanel
		 *
		 * @param {Number} itemId
		 * @param {Boolean} isSelected Item selection status
		 */
		toggleDoiSelected(itemId, isSelected) {
			if (isSelected) {
				if (!this.selected.includes(itemId)) {
					// Add to selected
					this.selected.push(itemId);
				}
			} else {
				if (this.selected.includes(itemId)) {
					// Remove if exists
					this.selected = this.selected.filter(item => item !== itemId);
				}
			}
		},
		/**
		 * Toggles expand all for DOI tabs
		 */
		toggleExpandAll() {
			this.isExpandAllOn = !this.isExpandAllOn;
		},
		/**
		 * Toggle select all for Ids in selected
		 */
		toggleSelectAll() {
			if (this.isSelectAllOn) {
				this.selected = [];
				this.isSelectAllOn = false;
			} else {
				this.selected = this.items.map(i => i.id);
				this.isSelectAllOn = true;
			}
		}
	},
	watch: {
		/**
		 * Sets isSelectAllOn value based on items in `selected` array
		 *
		 * @param newVal
		 * @param oldVal
		 */
		selected(newVal, oldVal) {
			this.isSelectAllOn =
				this.selected.length && this.selected.length === this.items.length;
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

.doiListPanel {
	.doiListPanel__options {
		display: flex;
	}

	.doiListPanel__options--expandAll {
		margin-top: 0.5rem;
	}

	// From PreviewListPanelSelect.vue
	.listPanel__selectAllWrapper {
		display: flex;
		align-items: center;
		margin-top: 1rem;
		margin-left: -0.5rem;
		margin-right: auto;
		line-height: 1.5rem;

		> input {
			margin-left: 0.5rem;
		}
	}

	.listPanel__selectAllLabel {
		margin-left: 0.5rem;
	}

	.listPanel__selectWrapper {
		display: flex;
		align-items: center;
		margin-left: -1rem;
	}

	.listPanel__selector {
		width: 3rem;
		padding-left: 1rem;
	}
}
</style>
