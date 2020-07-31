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
					<div v-if="canSelectAll" class="listPanel__selectAllWrapper">
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
				</template>

				<template v-slot:itemTitle="{item}">
					<!--Temp selector placement-->
					<label class="listPanel__selectWrapper">
						<div class="listPanel__selector">
							<input
								type="checkbox"
								name="submissions[]"
								:value="item.id"
								v-model="selected"
							/>
						</div>
					</label>
					<div>{{ localize(item.author) }}</div>
				</template>

				<template v-slot:itemSubtitle="{item}">
					{{ localize(item.title) }}
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

export default {
	components: {
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
		canSelectAll: {
			type: Boolean,
			default() {
				return true;
			}
		},
		isSelectAllOn: {
			type: Boolean,
			default() {
				return false;
			}
		},
		selected: {
			type: Array,
			default() {
				return [];
			}
		},
		title: {
			type: String,
			required: true
		},
		urlBase: {
			type: String,
			required: true
		}
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
		 * Toggle select all for list of items
		 */
		toggleSelectAll() {
			if (this.isSelectAllOn) {
				this.selected = [];
			} else {
				this.selected = this.items.map(i => i.id);
			}
		}
	},
	watch: {
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
	// From PreviewListPanelSelect.vue
	.listPanel__selectAllWrapper {
		display: flex;
		align-items: center;
		margin-top: 1rem;
		margin-left: -0.5rem;
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
