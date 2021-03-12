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

			<!-- Submission -->
			<div v-if="isSubmission" class="listPanel__itemIdentity">
				<div class="listPanel__itemTitle doiListItem__itemTitle">
					{{ item.id }} /
					<b>{{ currentPublication.authorsStringShort }}</b>
					/
					<a :href="this.currentPublication.urlPublished" target="_blank">
						{{ localize(currentPublication.fullTitle) }}
					</a>
				</div>
			</div>

			<!-- Issue -->
			<div v-else class="listPanel__itemIdentity">
				<div class="listPanel__itemTitle">
					{{ localize(item.title) }}
				</div>
				<div class="listPanel__itemSubtitle">
					<a :href="item.publishedUrl" target="_blank">
						{{ item.identification }}
					</a>
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
						v-if="isPublished && !isDeposited && crossrefPluginEnabled"
						class="doiListItem__itemMetadata--badge"
						:is-warnable="true"
					>
						<icon icon="exclamation" :inline="true" />
						__('plugins.importexport.crossref.status.notDeposited')
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
			<div class="doiListItem__doiSummary">
				<div class="doiListItem__doiDetail">
					<pkp-button v-if="crossrefPluginEnabled" :is-primary="true">
						Deposit DOI
					</pkp-button>
					<!-- TODO: Does not work with issues -->
					<pkp-button
						v-if="item['crossref::status'] === 'failed'"
						ref="depositFailureModalButton"
						@click="$modal.show('depositFailureMessage')"
					>
						<!-- TODO: Localized button name -->
						View error message
					</pkp-button>
					<modal
						v-bind="MODAL_PROPS"
						name="depositFailureMessage"
						@closed="setFocusToRef('depositFailureModalButton')"
					>
						<modal-content
							:closeLabel="__('common.close')"
							modalName="depositFailureMessage"
							title="Deposit Failure Message"
						>
							<!-- TODO: Add and localize explanatory message -->
							<p>
								---- Explanatory message about Crossref XML validation errors
								----
							</p>
							<div class="crossrefDepositError">
								<pre>{{ item['crossref::failedMsg'] }}</pre>
							</div>
						</modal-content>
					</modal>
				</div>
				<div class="doiListItem__doiActions">
					<badge v-if="crossrefPluginEnabled">
						{{ depositStatusString }}
					</badge>
				</div>
			</div>
			<list>
				<list-item v-for="item in doiList" :key="item.id">
					<template slot="value">{{ item.type }}</template>
					<div class="doiListItem__doiSummary">
						<div class="doiListItem__doiDetail">
							<field-doi-text
								v-bind="getDoiField(item.id)"
								:doiPrefix="doiPrefix"
								:opt-into-edit="true"
								:opt-into-edit-label="__('common.edit')"
								@change="changeDoiInput"
							/>
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
import modal from '@/mixins/modal';

export default {
	name: 'DoiListItem',
	components: {
		Expander,
		FieldDoiText,
		List,
		ListItem
	},
	mixins: [modal],
	props: {
		apiUrl: {
			type: String,
			required: true
		},
		doiPrefix: {
			type: String,
			default() {
				return '';
			}
		},
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

			return this.item.publications.find(
				publication => publication.id === this.item.currentPublicationId
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
						id: `article-${this.item.id}-${this.currentPublication.id}`,
						type: 'Article',
						identifier: this.currentPublication['pub-id::doi'],
						// TODO: Revisit removing deposit status. Only matters for submission as whole.
						depositStatus:
							this.item['crossref::status'] === null
								? 'notDeposited'
								: this.item['crossref::status'],
						apiPath: `${this.apiUrl}/${this.item.id}/publications/${this.currentPublication.id}/editDoi`
						// TODO: DOI is stored in publication but crossref deposit status is in submission only
					});
				}

				// Get galley DOIs
				for (let i = 0; i < this.currentPublication.galleys.length; i++) {
					let galley = this.currentPublication.galleys[i];
					if (galley.hasOwnProperty('pub-id::doi')) {
						dois.push({
							id: `galley-${this.item.id}-${this.currentPublication.id}-${galley.id}`,
							type: 'Galley',
							identifier: galley['pub-id::doi'],
							depositStatus:
								this.item['crossref::status'] === null
									? 'notDeposited'
									: this.item['crossref::status'],
							apiPath: `${this.apiUrl}/${this.item.id}/publications/${this.currentPublication.id}/galleys/${galley.id}/editDoi`
						});
					}
				}
			} else {
				// If not a submission, we have an issue
				if (this.item.hasOwnProperty('pub-id::doi')) {
					dois.push({
						id: `issue-${this.item.id}`,
						type: 'Issue',
						identifier: this.item['pub-id::doi'],
						depositStatus: 'Not deposited', // TODO: Needs to be addressed for issues, use `publicationWithCrossrefStatus`
						apiPath: `${this.apiUrl}/${this.item.id}/editDoi`
					});
				}
			}

			return dois;
		},
		/**
		 * Gets string for DOI deposit display
		 *
		 * @return {String}
		 */
		depositStatusString() {
			// TODO: Needs localization
			let item = this.isSubmission
				? this.item
				: this.publicationWithCrossrefStatus;

			return item['crossref::status'] === null
				? 'notDeposited'
				: item['crossref::status'];
		},
		/**
		 * Has the current item been deposited.
		 *
		 * @return {Boolean}
		 */
		isDeposited() {
			let item = this.isSubmission
				? this.item
				: this.publicationWithCrossrefStatus;

			return !(
				item['crossref::status'] === null ||
				item['crossref::status'] === 'failed'
			);
		},
		/**
		 * Has the current item been published.
		 *
		 * @return {Boolean}
		 */
		isPublished() {
			if (this.isSubmission) {
				// TODO: Should be pkp.const.STATUS_PUBLISHED, not included on this page
				return this.item.status === 3;
			} else {
				return this.item.isPublished;
			}
		},
		/**
		 * Display string for publication status
		 *
		 * @return {String}
		 */
		publicationStatusLabel() {
			if (this.isPublished) {
				return this.__('publication.status.published');
			} else {
				return this.__('publication.status.unpublished');
			}
		},
		/**
		 * Gets crossref status of first submission.
		 *
		 * For checking if issue has had DOI deposited along with a previous submission.
		 * Will check for registered/markRegistered, if failed will look for another registered submission.
		 *
		 * @returns {Object} Submission
		 */
		publicationWithCrossrefStatus() {
			if (this.isSubmission) return null;

			// Check if issue has at least one article associated with it
			if (this.item.articles && this.item.articles.length) {
				// We have at least one article
				for (const submission of this.item.articles) {
					if (
						submission['crossref::status'] === 'registered' ||
						submission['crossref::status'] === 'markedRegistered'
					) {
						return submission;
					}
				}

				return this.item.articles[0];
			} else {
				// Otherwise there are no articles associated with the issue
				// This means it is not deposited but also cannot be deposited so we return the standard 'not deposited' status
				return {'crossref::status': null};
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
				depositStatus: doiItem.depositStatus,
				apiPath: doiItem.apiPath
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

.crossrefDepositError {
	background: rgb(234, 237, 238);
}

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

	> * {
		white-space: nowrap;
	}

	// Space between each button or action
	> * + * {
		margin-left: 0.25rem;
	}
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
	margin-right: 0.25rem;

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
