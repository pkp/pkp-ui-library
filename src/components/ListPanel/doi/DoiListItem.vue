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
			<div class="listPanel__itemIdentity">
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
					ID: {{ item.id }}
					<badge
						class="doiListItem__publicationBadge"
						:is-success="isPublished"
					>
						{{ publicationStatusLabel }}
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
						</div>
						<div class="doiListItem__doiActions">
							<pkp-button>Edit</pkp-button>
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

			// Get publication (article) DOIs
			if (this.currentPublication.hasOwnProperty('pub-id::doi')) {
				dois.push({
					id: this.currentPublication.id,
					type: 'Article',
					identifier: this.currentPublication['pub-id::doi']
				});
			}

			// Get galley DOIs
			for (let i = 0; i < this.currentPublication.galleys.length; i++) {
				let galley = this.currentPublication.galleys[i];
				if (galley.hasOwnProperty('pub-id::doi')) {
					dois.push({
						id: galley.id,
						type: 'Galley',
						identifier: galley['pub-id::doi']
					});
				}
			}

			return dois;
		},
		/**
		 * Has the current publication been published.
		 *
		 * @return {Boolean}
		 */
		isPublished() {
			return this.item.status === pkp.const.STATUS_PUBLISHED;
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
