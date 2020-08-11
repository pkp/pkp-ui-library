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
				<div class="listPanel__itemTitle">
					{{ currentPublication.authorsStringShort }}
				</div>
				<div class="listPanel__itemSubtitle">
					<a :href="this.currentPublication.urlPublished" target="_blank">
						{{ localize(currentPublication.fullTitle) }}
					</a>
				</div>
				<!-- DOI item metadata -->
				<div class="doiListItem__itemMetadata">
					<!--			TODO: Localize doi list metadata-->
					ID: {{ item.object.id }}
					<badge
						class="doiListItem__publicationBadge"
						:is-success="isPublished"
					>
						{{ publicationStatusLabel }}
					</badge>
					<!-- TODO: Implement properly -->
					<badge>
						Not deposited
					</badge>
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
				<!-- DOI item metadata -->
				<div class="doiListItem__itemMetadata">
					<!--			TODO: Localize doi list metadata-->
					ID: {{ item.object.issueId }}
					<badge
						class="doiListItem__publicationBadge"
						:is-success="isPublished"
					>
						{{ publicationStatusLabel }}
					</badge>
					<!-- TODO: Implement properly -->
					<badge>
						Not deposited
					</badge>
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

		<!-- Expanded view/identifiers -->
		<div
			v-if="isExpanded"
			class="listPanel__itemExpanded listPanel__itemExpanded--doi"
		>
			<list>
				<list-item v-for="item in doiList" :key="item.id">
					<template slot="value">{{ item.type }}</template>

					<div class="doiListItem__doiSummary">
						<div class="doiListItem__doiDetail">
							<a :href="doiURL(item.identifier)" target="_blank">
								{{ item.identifier }}
							</a>

							<!-- TODO: Add localization text -->
							<pkp-button>
								Edit
							</pkp-button>
						</div>

						<div class="doiListItem__doiActions">
							<badge>Not deposited</badge>
							<pkp-button :isPrimary="true">
								Deposit DOI
							</pkp-button>
						</div>
					</div>
				</list-item>
			</list>
		</div>
	</div>
</template>

<script>
import Expander from '@/components/Expander/Expander.vue';
import List from '@/components/List/List.vue';
import ListItem from '@/components/List/ListItem.vue';
export default {
	name: 'DoiListItem',
	components: {
		Expander,
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
						id: this.currentPublication.id,
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
							id: galley.id,
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
						id: this.item.object.issueId,
						type: 'Issue',
						identifier: this.item.object['pub-id::doi'],
						depositStatus: 'Not deposited'
					});
				}
			}

			return dois;
		},
		isSubmission() {
			if (this.item.objectType === 'submission') {
				return true;
			} else {
				return false;
			}
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
		 * Toggles item as selected and notifies DoiListPanel
		 */
		toggleSelected() {
			// Notifies DoiListPanel of selection toggle
			this.$emit('toggleDoiSelected', this.item.id, this.isSelected);
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
	flex: 1;
	min-width: 0;
}

.doiListItem__doiDetail .pkpButton {
	margin-left: 1rem;
	margin-right: 1rem;
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

.doiListItem__itemMetadata {
	margin-top: 0.5em;
	font-size: @font-tiny;
	line-height: 1.5em;
	color: @text;
}

.listPanel__itemExpanded--doi {
	margin-left: 2.25rem;
}

.doiListItem__publicationBadge {
	margin-left: 0.5rem;
	margin-right: 0.25rem;
}
</style>
