<template>
	<div class="doiListPanel">
		<slot>
			<list-panel :items="items" :isSidebarVisible="isSidebarVisible">
				<template slot="header">
					<pkp-header>
						<h2>{{ title }}</h2>
						<spinner v-if="isLoading" />
						<template slot="actions">
							<search
								:searchPhrase="searchPhrase"
								@search-phrase-changed="setSearchPhrase"
							/>

							<!-- TODO: Localize dropdown -->
							<dropdown label="Bulk Actions">
								<div class="pkpDropdown__section">
									<div class="app__userNav__loggedInAs">
										Take action on {{ selected.length }} selected item(s)
									</div>
									<button class="-linkButton" @click="toggleSelectAll">
										{{ isSelectAllOn ? 'Deselect all' : 'Select all' }}
									</button>
								</div>

								<div class="pkpDropdown__section">
									<ul>
										<li>
											<button class="pkpDropdown__action">
												Deposit
											</button>
										</li>

										<li>
											<button class="pkpDropdown__action">
												Assign DOIs
											</button>
										</li>

										<li>
											<button
												class="pkpDropdown__action"
												@click="toggleExpandAll"
											>
												<!-- TODO: Localize text -->
												{{ isExpandAllOn ? 'Collapse' : 'Expand' }}
											</button>
										</li>
									</ul>
								</div>
							</dropdown>

							<!-- TODO: Localize -->
							<pkp-button
								ref="modalDepositButton"
								:isPrimary="true"
								@click="openDepositDialog"
							>
								Deposit
							</pkp-button>
						</template>
					</pkp-header>
				</template>

				<template slot="sidebar">
					<pkp-header :isOneLine="false">
						<h3>
							<icon icon="filter" :inline="true" />
							{{ __('common.filter') }}
						</h3>
					</pkp-header>
					<div
						v-for="(filterSet, index) in filters"
						:key="index"
						class="listPanel__block"
					>
						<pkp-header v-if="filterSet.heading">
							<h4>{{ filterSet.heading }}</h4>
						</pkp-header>
						<component
							v-for="filter in filterSet.filters"
							:key="filter.param + filter.value"
							:is="filter.filterType || 'pkp-filter'"
							v-bind="filter"
							:isFilterActive="isFilterActive(filter.param, filter.value)"
							@add-filter="addFilter"
							@remove-filter="removeFilter"
							@update-filter="addFilter"
						/>
					</div>
				</template>

				<template v-slot:item="{item}">
					<slot name="item" :item="item">
						<doi-list-item
							:key="item.id"
							:item="item"
							:apiUrl="apiUrl"
							:doiPrefix="doiPrefix"
							@toggleDoiSelected="toggleDoiSelected"
							:selected="selected"
							:isSelectable="isSelectable"
							:isExpandAllOn="isExpandAllOn"
							:crossrefPluginEnabled="crossrefPluginEnabled"
							:isSubmission="isSubmission"
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
import PkpFilter from '@/components/Filter/Filter.vue';
import PkpFilterAutosuggest from '@/components/Filter/FilterAutosuggest.vue';
import Search from '@/components/Search/Search.vue';
import fetch from '@/mixins/fetch';
import DoiListItem from '@/components/ListPanel/doi/DoiListItem';
import Dropdown from '@/components/Dropdown/Dropdown';
import FieldSelect from '@/components/Form/fields/FieldSelect';

export default {
	components: {
		Dropdown,
		DoiListItem,
		FieldSelect,
		ListPanel,
		Pagination,
		PkpFilter,
		PkpFilterAutosuggest,
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
		issueFilter: {
			type: Object,
			default() {
				return {};
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
		},
		doiPrefix: {
			type: String,
			default() {
				return '';
			}
		},
		crossrefPluginEnabled: {
			type: Boolean,
			default() {
				return false;
			}
		},
		isSubmission: {
			type: Boolean,
			default() {
				return true;
			}
		}
	},
	data() {
		return {
			activeFilters: {},
			isExpandAllOn: false,
			isSelectAllOn: false,
			isSidebarVisible: true,
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
		// Toggles
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
		},
		/**
		 * Add an active filter
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		addFilter(param, value) {
			let newFilters = {...this.activeFilters};
			if (['status', 'isPublished'].includes(param)) {
				// Handle "toggleable" or single select filters
				newFilters[param] = value;
			} else {
				// Handle multi-select filters
				if (!newFilters[param]) {
					newFilters[param] = [];
				}
				newFilters[param].push(value);
			}
			this.activeFilters = newFilters;
		},
		/**
		 * Is a filter currently active?
		 *
		 * @param {string} param The filter param
		 * @param {mixed} value The filter value
		 * @return {Boolean}
		 */
		isFilterActive: function(param, value) {
			if (!Object.keys(this.activeFilters).includes(param)) {
				return false;
			} else if (Array.isArray(this.activeFilters[param])) {
				return this.activeFilters[param].includes(value);
			} else {
				return this.activeFilters[param] === value;
			}
		},
		/**
		 * Remove an active filter
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		removeFilter(param, value) {
			let newFilters = {...this.activeFilters};
			if (['status', 'isPublished'].includes(param)) {
				// Handle "toggleable" filters
				delete newFilters[param];
			} else {
				// Handle multi-select filters
				newFilters[param] = newFilters[param].filter(v => v !== value);
			}
			this.activeFilters = newFilters;
		},
		openDepositDialog() {
			this.openDialog({
				cancelLabel: 'Cancel',
				confirmLabel: 'Deposit DOIs',
				message: `You are about to send DOI metadata records for 23 submission(s) to CrossRef. Are you sure you want to deposit these records?`,
				modalName: 'deposit',
				title: 'Deposit DOIs',
				callback: () => {
					// Make ajax Request
					// This code just simulates a server request
					setTimeout(() => {
						this.$modal.hide('deposit');
					}, 2000);
				}
			});
		}
	},
	computed: {
		computedFilters() {
			if (this.isSubmission) {
				return [
					{
						heading: 'Publication Status',
						filters: [
							{
								title: 'Published',
								param: 'status',
								value: `${pkp.const.STATUS_PUBLISHED}`
							},
							{
								title: 'Unpublished',
								param: 'status',
								value: `${pkp.const.STATUS_QUEUED}, ${pkp.const.STATUS_SCHEDULED}` // '1,5'
							}
						]
					},
					{
						heading: 'CrossRef Deposit Status',
						filters: [
							{
								title: 'Not deposited',
								param: 'crossrefStatus',
								value: 'notDeposited'
							},
							{
								title: 'Active',
								param: 'crossrefStatus',
								value: 'registered'
							},
							{
								title: 'Failed',
								param: 'crossrefStatus',
								value: 'failed'
							},
							{
								title: 'Marked Active',
								param: 'crossrefStatus',
								value: 'markedRegistered'
							}
						]
					}
				];
			} else {
				return [
					{
						heading: 'Publication Status',
						filters: [
							{
								title: 'Published',
								param: 'isPublished',
								value: '1'
							},
							{
								title: 'Unpublished',
								param: 'isPublished',
								value: '0'
							}
						]
					}
				];
			}
		},
		filters() {
			return [...this.computedFilters, this.issueFilter];
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
		margin-top: 0.5rem;
	}

	.doiListPanel__options--button {
		margin-left: 0.25rem;
	}

	// From PreviewListPanelSelect.vue
	.listPanel__selectAllWrapper {
		display: flex;
		align-items: center;
		//margin-top: 1rem;
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
