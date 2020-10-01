<template>
	<div class="listPanel__item--doi">
		<div class="listPanel__itemSummary">
			<!-- Item selector -->
			<label v-if="isSelectable" class="listPanel__selectWrapper">
				<div class="listPanel__selector">
					<input
						type="checkbox"
						name="submissions[]"
						:value="item.id"
						v-model="isSelected"
					/>
				</div>
			</label>

			<!-- Item overview -->
			<div v-if="isSubmission" class="listPanel__itemIdentity">
				<!-- Submission -->
				<div class="listPanel__itemTitle doiListItem__itemTitle">
					{{ item.object.id }} /
					<b>{{ currentPublication.authorsStringShort }}</b>
					/
					<a :href="this.currentPublication.urlPublished" target="_blank">
						{{ localize(currentPublication.fullTitle) }}
					</a>
				</div>
			</div>

			<div v-else class="listPanel__itemIdentity">
				<!-- Issue -->
				<div class="listPanel__itemTitle">
					{{ item.object.title }}
				</div>
				<div class="listPanel__itemSubtitle">
					{{ issueInfo }}
				</div>
			</div>

			<div class="listPanel__itemActions">
				<!-- DOI item metadata -->
				<div class="doiListItem__itemMetadata">
					<!--			TODO: Localize doi list metadata-->
					<badge v-if="!isPublished" class="doiListItem__itemMetadata--badge">
						{{ publicationStatusLabel }}
					</badge>
					<badge
						v-if="isPublished && !isEveryDoiDeposited"
						class="doiListItem__itemMetadata--badge"
						:is-warnable="true"
					>
						<icon icon="exclamation" :inline="true" />
						Needs depositing
					</badge>
				</div>

				<expander
					:isExpanded="isExpanded"
					:itemName="item.id.toString()"
					@toggle="isExpanded = !isExpanded"
				/>
			</div>
		</div>

		<!-- Expanded view/identifiers -->
		<div
			v-if="isExpanded"
			class="listPanel__itemExpanded listPanel__itemExpanded--doi"
		>
			<pkp-button :is-primary="true">Deposit DOI</pkp-button>
			<list>
				<list-item v-for="item in doiList" :key="item.id">
					<template slot="value">{{ item.type }}</template>
					<div class="doiListItem__doiSummary">
						<div class="doiListItem__doiDetail">
							<field-doi-text
								v-bind="getDoiField(item.id)"
								:opt-into-edit="true"
								:opt-into-edit-label="__('common.edit')"
								@change="changeDoiInput"
							/>
						</div>

						<div class="doiListItem__doiActions">
							<badge>Not deposited</badge>
							<!--							<pkp-button :isPrimary="true">-->
							<!--								Deposit DOI-->
							<!--							</pkp-button>-->
						</div>
					</div>
				</list-item>
			</list>
		</div>
	</div>
</template>

<script>
import Expander from '@/components/Expander/Expander.vue';
import FieldDoiText from '@/components/Form/fields/FieldDoiText';
import List from '@/components/List/List.vue';
import ListItem from '@/components/List/ListItem.vue';

export default {
	name: 'DoiListItem',
	components: {
		Expander,
		FieldDoiText,
		List,
		ListItem
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
		},
		isExpandAllOn: {
			type: Boolean,
			default() {
				return false;
			}
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
			doiFields: [],
			isExpanded: false,
			isSelected: false
		};
	},
	computed: {
		/**
		 * The current publication of the submission
		 *
		 * @return {Object}
		 */
		currentPublication() {
			if (this.isSubmission === false) return null;

			return this.item.object.publications.find(
				publication => publication.id === this.item.object.currentPublicationId
			);
		},
		/**
		 * Gets doi list for current publication and galleys
		 *
		 * @return {Array}
		 */
		doiList() {
			let dois = [];

			// DOI List can come from either submission or issue, check submission first
			if (this.isSubmission === true) {
				// Get publication (article) DOIs
				if (this.currentPublication.hasOwnProperty('pub-id::doi')) {
					dois.push({
						id: 'article-' + this.currentPublication.id,
						type: 'Article',
						identifier: this.currentPublication['pub-id::doi'],
						depositStatus: 'Not deposited'
					});
				}

				// Get galley DOIs
				for (let i = 0; i < this.currentPublication.galleys.length; i++) {
					let galley = this.currentPublication.galleys[i];
					if (galley.hasOwnProperty('pub-id::doi')) {
						dois.push({
							id: 'galley-' + galley.id,
							type: 'Galley',
							identifier: galley['pub-id::doi'],
							depositStatus: 'Not deposited'
						});
					}
				}
			} else {
				// If not a submission, we have an issue
				window.console.log(this.item.object);
				if (this.item.object.hasOwnProperty('pub-id::doi')) {
					dois.push({
						id: 'issue-' + this.item.object.issueId,
						type: 'Issue',
						identifier: this.item.object['pub-id::doi'],
						depositStatus: 'Not deposited'
					});
				}
			}

			return dois;
		},
		isEveryDoiDeposited() {
			for (const doi of this.doiList) {
				if (doi.depositStatus === 'Not deposited') {
					return false;
				}
			}
			return true;
		},
		isSubmission() {
			return this.item.objectType === 'submission';
		},
		issueInfo() {
			return (
				'Vol: ' +
				this.item.object.volume +
				', No: ' +
				this.item.object.number +
				', (' +
				this.item.object.year +
				')'
			);
		},
		/**
		 * Has the current publication been published.
		 *
		 * @return {Boolean}
		 */
		isPublished() {
			if (this.isSubmission) {
				return this.item.object.status === pkp.const.STATUS_PUBLISHED;
			} else {
				return this.item.object.published === 1; // 1/0 bool in `issues` table
			}
		},
		publicationStatusLabel() {
			// TODO: Needs localization
			if (this.isPublished) {
				return 'Published';
			} else {
				return 'Not published';
			}
		}
	},
	methods: {
		/**
		 * Update field-text field for doi input
		 *
		 * @param {String} name FieldText name (doi id)
		 * @param {String} prop FieldText field "value"
		 * @param {String} newValue
		 *
		 */
		changeDoiInput(name, prop, newValue) {
			this.getDoiField(name).value = newValue;
		},
		/**
		 * Builds DOI URLs
		 *
		 * @param {String} doi
		 *
		 * @return {String}
		 */
		doiURL(doi) {
			return 'https://doi.org/' + doi;
		},
		/**
		 * Gets field in doiFields array for fieldText input handling
		 *
		 * @param {String} id
		 *
		 * @returns {Object} doiField
		 */
		getDoiField(id) {
			return this.doiFields.find(fieldObj => fieldObj.name === id);
		},
		/**
		 * Toggles item as selected and notifies DoiListPanel
		 */
		toggleSelected() {
			// Notifies DoiListPanel of selection toggle
			this.$emit('toggleDoiSelected', this.item.id, this.isSelected);
		}
	},
	mounted: function() {
		// Gets doi objects from doiList on mount and adds them to doiFields
		// for further manipulation
		for (let doiItem of this.doiList) {
			this.doiFields.push({
				name: doiItem.id,
				value: doiItem.identifier,
				allErrors: {},
				isDisabled: true,
				depositStatus: doiItem.depositStatus
			});
		}
	},
	watch: {
		isExpandAllOn() {
			this.isExpanded = this.isExpandAllOn;
		},
		isSelected(newVal, oldVal) {
			this.toggleSelected();
		},
		selected(newVal, oldVal) {
			this.isSelected = this.selected.includes(this.item.id);
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

.doiListItem__doiSummary {
	position: relative;
	display: flex;
	align-items: center;
	width: 100%;
}

.doiListItem__doiDetail {
	display: flex;
	flex: 1;
	min-width: 0;
}

.doiListItem__doiDetail--editButton {
	margin-top: 0.25rem;
	margin-left: 0.25rem;
	margin-right: 0.25rem;
}

.doiListItem__doiActions {
	display: flex;
	align-items: center;
	margin-left: auto;
	padding-left: 0.5rem;

	> * {
		white-space: nowrap;
	}

	// Space between each button or action
	> * + * {
		margin-left: 0.25rem;
	}
}

.doiListItem__itemTitle {
	font-weight: 400;
}

.doiListItem__itemMetadata {
	margin-top: 0.5em;
	font-size: @font-tiny;
	line-height: 1.5em;
	color: @text;
}

.doiListItem__itemMetadata--badge {
	margin-right: 0.25rem;
}

.listPanel__itemExpanded--doi {
	margin-left: 2.25rem;
}
</style>
