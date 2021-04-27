<template>
	<div class="pkpFormField pkpFormField--text" :class="classes">
		<div class="pkpFormField__heading"></div>
		<div class="pkpFormField__control" :class="controlClasses">
			<div class="pkpFormField__control_top">
				<input
					class="pkpFormField__input pkpFormField--text__input"
					ref="input"
					v-model="currentValue"
					:type="inputType"
					:id="controlId"
					:name="localizedName"
					:aria-describedby="describedByIds"
					:aria-invalid="errors && errors.length"
					:disabled="isDisabled"
					:required="isRequired"
					:style="inputStyles"
				/>
				<span ref="buttons">
					<pkp-button
						v-if="optIntoEdit && isDisabled && !isDeposited"
						class="pkpFormField--text__optIntoEdit"
						@click="isDisabled = false"
					>
						{{ optIntoEditLabel }}
					</pkp-button>
					<pkp-button
						v-if="optIntoEdit && !isDisabled && !isDeposited"
						class="pkpFormField--text__optIntoEdit"
						@click="save"
					>
						{{ __('common.save') }}
					</pkp-button>
				</span>
				<span role="status" aria-live="polite" aria-atomic="true">
					<transition name="pkpFormPage__status">
						<span v-if="isSaving" class="pkpFormPage__status">
							<spinner />
							{{ __('common.saving') }}
						</span>
						<span v-else-if="hasRecentSave" class="pkpFormPage__status">
							<icon icon="check" :inline="true" />
							{{ __('form.saved') }}
						</span>
					</transition>
				</span>
			</div>
			<field-error
				v-if="errors && errors.length"
				:id="describedByErrorId"
				:messages="errors"
			/>
		</div>
	</div>
</template>

<script>
import FieldBase from './FieldBase.vue';

export default {
	name: 'FieldDoiText',
	extends: FieldBase,
	props: {
		apiPath: String,
		depositStatus: String,
		doiPrefix: String,
		inputType: String,
		optIntoEdit: Boolean,
		optIntoEditLabel: String,
		size: {
			default: 'normal',
			validator: function(value) {
				return ['small', 'normal', 'large'].indexOf(value) !== -1;
			}
		},
		prefix: String
	},
	data() {
		return {
			inputStyles: {},
			isDisabled: false,
			isSaving: false,
			hasRecentSave: false,
			// recentSaveInterval: null,
			lastSaveTimestamp: -1
		};
	},
	computed: {
		/**
		 * Add classes to wrapper element based on configuration
		 *
		 * @return {Array}
		 */
		classes() {
			return ['pkpFormField--size' + this.size];
		},

		/**
		 * Add classes to the input control
		 *
		 * @return {Array}
		 */
		controlClasses() {
			let classes = [];
			if (this.isMultilingual && this.locales.length > 1) {
				classes.push('pkpFormField__control--hasMultilingualIndicator');
			}
			if (this.prefix) {
				classes.push('pkpFormField__control--hasPrefix');
			}
			return classes;
		},
		/**
		 * Returns deposit status for editing control
		 *
		 * @return {Boolean}
		 */
		isDeposited() {
			return this.depositStatus === 'deposited';
		}
	},
	watch: {
		/**
		 * When saving, set the focus to the button wrapper element so it doesn't
		 * get dropped as the dom updates
		 */
		isSaving() {
			this.$refs.buttons.focus();
		},
		isDisabled(newValue, oldValue) {
			if (newValue === false && this.value === null) {
				this.currentValue = this.doiPrefix;
			}
		}
	},
	methods: {
		/**
		 * Submit the DOI edits
		 */
		save() {
			this.isSaving = true;

			// TODO: Error handling needed here

			$.ajax({
				url: this.apiPath,
				type: 'POST',
				headers: {
					'X-Csrf-Token': pkp.currentUser.csrfToken,
					'X-Http-Method-Override': 'PUT',
					contentType: 'application/x-www-form-urlencoded'
				},
				data: {'pub-id::doi': `${this.value}`},
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
			this.lastSaveTimestamp = Date.now();
			// TODO: Update value with response value
			// TODO: Emit set or saveDoi signal
			// this.$emit('saveDoi', this.name, this.value);
			this.isDisabled = true;
		},
		/**
		 * Callback to fire when the form submission's ajax request has been
		 * returned with errors
		 *
		 * @param {Object} r The response to the AJAX request
		 */
		error: function(r) {
			// TODO: Handle AJAX error
		},
		/**
		 * Callback to fire when the form's submission ajax request has been
		 * returned, and the success or error callbacks have already been fired.
		 *
		 * @param {Object} r The response to the AJAX request
		 */
		complete() {
			this.isSaving = false;
		}
	},
	mounted() {
		// Set the field to disabled if optIntoEdit is passed
		if (this.optIntoEdit) {
			this.isDisabled = true;
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

// From FormPage.vue
.pkpFormPage__status {
	display: inline-block;
	margin-right: 0.5rem;
	font-size: @font-tiny;
	transition: all 0.3s;
	text-align: right;

	.fa {
		color: @yes;
	}

	.pkpSpinner {
		margin-right: 0.25rem;
	}
}

.pkpFormPage__status-enter {
	transform: translateY(0.5rem);
	opacity: 0;
}

.pkpFormPage__status-leave-to {
	transform: translateY(-0.5rem);
	opacity: 0;
}
</style>
