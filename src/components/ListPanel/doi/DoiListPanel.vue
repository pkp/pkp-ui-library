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

							<pkp-button
								:is-active="isSidebarVisible"
								@click="isSidebarVisible = !isSidebarVisible"
							>
								<icon icon="filter" :inline="true" />
								<!-- TODO: Localize text -->
								Filters
							</pkp-button>
						</template>
					</pkp-header>

					<div class="doiListPanel__options">
						<div v-if="isSelectable" class="listPanel__selectAllWrapper">
							<pkp-button @click="toggleSelectAll">
								<icon
									:icon="isSelectAllOn ? 'check-square-o' : 'square-o'"
									:inline="true"
								/>
								Select All
							</pkp-button>
							<div class="doiListPanel__options--button">
								<dropdown label="Actions">
									<ul>
										<li>
											<button class="pkpDropdown__action">
												Deposit selected
											</button>
										</li>
										<li>
											<button class="pkpDropdown__action">Button Two</button>
										</li>
									</ul>
								</dropdown>
							</div>
						</div>
						<span class="doiListPanel__options--button">
							<pkp-button @click="toggleExpandAll">
								<!-- TODO: Localize text -->
								{{ isExpandAllOn ? 'Collapse all' : 'Expand all' }}
							</pkp-button>
						</span>
					</div>
				</template>

				<template slot="sidebar">
					<pkp-header :isOneLine="false">
						<h3>
							<icon icon="filter" :inline="true" />
							Filters
						</h3>
					</pkp-header>
					<div>
						<!-- Publication Status -->
						<div class="listPanel__filterSet">
							<pkp-header>
								<h4>{{ filters.publicationStatus.heading }}</h4>
							</pkp-header>
							<component
								v-for="filter in filters.publicationStatus.filters"
								:key="filter.param + filter.value"
								:is="filter.filterType || 'pkp-filter'"
								v-bind="filter"
								:isFilterActive="isFilterActive(filter.param, filter.value)"
								@add-filter="addFilter"
								@remove-filter="removeFilter"
							/>
						</div>
						<!-- Crossref Status -->
						<div
							v-if="crossrefPluginEnabled && isSubmission"
							class="listPanel__filterSet"
						>
							<pkp-header>
								<h4>{{ filters.crossrefStatus.heading }}</h4>
							</pkp-header>
							<component
								v-for="filter in filters.crossrefStatus.filters"
								:key="filter.param + filter.value"
								:is="filter.filterType || 'pkp-filter'"
								v-bind="filter"
								:isFilterActive="isFilterActive(filter.param, filter.value)"
								@add-filter="addFilter"
								@remove-filter="removeFilter"
							/>
						</div>
						<div v-if="isSubmission" class="listPanel__filterSet">
							<!--							<pkp-header>-->
							<!--								<h4>Issues</h4>-->
							<!--							</pkp-header>-->
							<field-select
								v-bind="issueList"
								:allErrors="{}"
								@change="issueFilterChanged"
							/>
						</div>
					</div>
				</template>

				<template v-slot:item="{item}">
					<slot name="item" :item="item">
						<doi-list-item
							:key="item.id"
							:item="item"
							:apiUrl="apiUrl"
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
import PkpFilter from '@/components/Filter/Filter';
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
		issueList: {
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
			isSidebarVisible: false,
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
			// this.items = items;
			// this.itemsMax = itemsMax;
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
		// Filters

		/**
		 * Handle change to issue field select for filtering
		 *
		 * @param {String} name
		 * @param {String} prop
		 * @param {mixed} newValue
		 * @param {String} localeKey
		 */
		issueFilterChanged(name, prop, newValue, localeKey) {
			// Set value in issueList on DoiListPanel
			if (this.issueList.isMultilingual) {
				this.issueList.value[localeKey] = newValue;
			} else {
				this.issueList.value = newValue;
			}
			// Handle filtering
			if (newValue === '' || newValue === 0) {
				// Remove issue filter as 0 means no issue and '' is empty placeholder
				this.removeFilter(name, newValue);
			} else {
				// Add issue filter
				this.addFilter(name, newValue);
			}
		},
		/**
		 * Add a filter
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		addFilter(param, value) {
			let newFilters = {...this.activeFilters};
			if (
				['status'].includes(param) ||
				['issueIds'].includes(param) ||
				['isPublished'].includes(param)
			) {
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
			}
			if (Array.isArray(this.activeFilters[param])) {
				return this.activeFilters[param].includes(value);
			}
			return this.activeFilters[param] === value;
		},
		/**
		 * Remove a filter
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		removeFilter(param, value) {
			let newFilters = {...this.activeFilters};
			if (
				['status'].includes(param) ||
				['issueIds'].includes(param) ||
				['isPublished'].includes(param)
			) {
				// Handle "toggleable" filters
				delete newFilters[param];
			} else {
				// Handle multi-select filters
				newFilters[param] = newFilters[param].filter(v => v !== value);
			}
			this.activeFilters = newFilters;
		}
	},
	computed: {
		filters() {
			if (this.isSubmission) {
				return {
					publicationStatus: {
						heading: 'Publication Status',
						filters: [
							{
								// TODO: Should be pkp.const.STATUS_PUBLISHED, not working in OJS
								title: 'Published',
								param: 'status',
								value: '3'
							},
							{
								// TODO: Should also use pkp.const.STATUS_*
								title: 'Unpublished',
								param: 'status',
								value: '1,5'
							}
						]
					},
					crossrefStatus: {
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
				};
			} else {
				return {
					publicationStatus: {
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
				};
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
