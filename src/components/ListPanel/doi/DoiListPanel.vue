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
									<br />
									<button class="-linkButton" @click="toggleExpandAll">
										<!-- TODO: Localize text -->
										{{ isAllExpanded ? 'Collapse all' : 'Expand all' }}
									</button>
								</div>

								<div class="pkpDropdown__section">
									<ul>
										<li>
											<button
												class="pkpDropdown__action"
												@click="openDepositDialog(selected, 'export')"
											>
												Export
											</button>
										</li>

										<li>
											<button
												class="pkpDropdown__action"
												@click="openDepositDialog(selected, 'markRegistered')"
											>
												Mark registered
											</button>
										</li>

										<li>
											<button
												class="pkpDropdown__action"
												@click="openDepositDialog(selected, 'deposit')"
											>
												Deposit
											</button>
										</li>

										<!--										<li>-->
										<!--											<button class="pkpDropdown__action">-->
										<!--												Assign DOIs-->
										<!--											</button>-->
										<!--										</li>-->
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

				<!-- TODO: Temporary solution to no items found text appearing on load -->
				<template slot="itemsEmpty">
					<template v-if="isLoading">
						<spinner />
						{{ __('common.loading') }}
					</template>
					<template v-else>
						{{ __('common.noItemsFound') }}
					</template>
				</template>

				<template v-slot:item="{item}">
					<slot name="item" :item="item">
						<doi-list-item
							:key="item.id"
							:item="item"
							:apiUrl="apiUrl"
							:doiPrefix="doiPrefix"
							:selected="selected"
							:isAllExpanded="isAllExpanded"
							:crossrefPluginEnabled="crossrefPluginEnabled"
							:isSubmission="isSubmission"
							:hasDOIs="hasDOIs"
							@toggleDoiSelected="toggleDoiSelected"
							@deposit-triggered="openDepositDialog"
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
		},
		hasDOIs: {
			type: Array,
			default() {
				return [];
			}
		}
	},
	data() {
		return {
			activeFilters: {},
			isAllExpanded: false,
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
					this.selected.push(itemId);
				}
			} else {
				if (this.selected.includes(itemId)) {
					this.selected = this.selected.filter(item => item !== itemId);
				}
			}
		},
		/**
		 * Toggles expand all for DOI tabs
		 */
		toggleExpandAll() {
			this.isAllExpanded = !this.isAllExpanded;
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
			if (['status', 'isPublished', 'issueIds'].includes(param)) {
				// Handle "toggleable" filters
				delete newFilters[param];
			} else {
				// Handle multi-select filters
				newFilters[param] = newFilters[param].filter(v => v !== value);
			}
			this.activeFilters = newFilters;
		},
		openDepositDialog(itemIds = [], action = 'deposit') {
			const items = itemIds.length > 0 ? itemIds : this.selected;

			let actionMessage = '';
			let actionLabel = '';
			switch (action) {
				case 'deposit':
					actionLabel = 'Deposit DOIs';
					actionMessage = `You are about to send DOI metadata records for ${items.length} submission(s) to CrossRef. Are you sure you want to deposit these records?`;
					break;
				case 'markRegistered':
					actionLabel = 'Mark DOIs registered';
					actionMessage = `You are about to mark DOI metadata records for ${items.length} submission(s) as registered. Are you sure you want to mark these records as registered?`;
					break;
				case 'export':
					actionLabel = 'Export DOIs';
					actionMessage = `You are about to export DOI metadata records for ${items.length} submission(s) for CrossRef. Are you sure you want to export these records?`;
					break;
			}
			this.openDialog({
				cancelLabel: 'Cancel',
				confirmLabel: actionLabel,
				message: actionMessage,
				modalName: 'deposit',
				title: actionLabel,
				callback: () => {
					// Make ajax Request
					// This code just simulates a server request
					// 	setTimeout(() => {
					// 		this.$modal.hide('deposit');
					// 	}, 2000);
					this.executeExportAction(items, action);
				}
			});
		},
		executeExportAction(itemIds, action) {
			const exportUrl = `${this.apiUrl}/crossref?${action}=${action}&submissionIds=${itemIds}`;
			$.ajax({
				url: exportUrl,
				type: 'POST',
				headers: {
					'X-Csrf-Token': pkp.currentUser.csrfToken
					// 'X-Http-Method-Override': 'PUT',
					// contentType: 'application/x-www-form-urlencoded'
				},
				// data: {},
				success: this.success,
				error: this.error,
				complete: this.complete
			});
		},
		/**
		 * Callback to fire when the form submission's ajax request has been
		 * returned successfully
		 *
		 * @param {Object} r The response to the AJAX request
		 */
		success: function(r) {
			window.console.log('[Success]', r);
			this.get();
		},
		/**
		 * Callback to fire when the form submission's ajax request has been
		 * returned with errors
		 *
		 * @param {Object} r The response to the AJAX request
		 */
		error: function(r) {
			window.console.log('[Error]', r);
		},
		/**
		 * Callback to fire when the form's submission ajax request has been
		 * returned, and the success or error callbacks have already been fired.
		 *
		 * @param {Object} r The response to the AJAX request
		 */
		complete: function(r) {
			window.console.log('[Complete]', r);
			this.$modal.hide('deposit');
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
	},
	mounted() {
		this.$on('deposit-triggered', (id, action) => {
			this.openDepositDialog([id], action);
		});
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
		line-height: 100%;
		width: 3rem;
		padding-left: 1rem;
	}
}
</style>
