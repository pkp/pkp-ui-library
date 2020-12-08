<template>
	<div class="pkpFormField pkpFormField--text" :class="classes">
		<div class="pkpFormField__heading">
			<form-field-label
				:controlId="controlId"
				:label="label"
				:localeLabel="localeLabel"
				:isRequired="isRequired"
				:requiredLabel="__('common.required')"
				:multilingualLabel="multilingualLabel"
			/>
			<tooltip
				v-if="isPrimaryLocale && tooltip"
				aria-hidden="true"
				:tooltip="tooltip"
				label=""
			/>
			<span
				v-if="isPrimaryLocale && tooltip"
				class="-screenReader"
				:id="describedByTooltipId"
				v-html="tooltip"
			/>
			<help-button
				v-if="isPrimaryLocale && helpTopic"
				:id="describedByHelpId"
				:topic="helpTopic"
				:section="helpSection"
				:label="__('help.help')"
			/>
		</div>
		<div
			v-if="isPrimaryLocale && description"
			class="pkpFormField__description"
			v-html="description"
			:id="describedByDescriptionId"
		/>
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
				<span
					v-if="prefix"
					class="pkpFormField__inputPrefix"
					v-html="prefix"
					ref="prefix"
					:style="prefixStyles"
					@click="setFocus"
				/>
				<multilingual-progress
					v-if="isMultilingual && locales.length > 1"
					:id="multilingualProgressId"
					:count="multilingualFieldsCompleted"
					:total="locales.length"
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
						@click="triggerDoiSave"
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
			prefixStyles: {},
			isSaving: false,
			hasRecentSave: false,
			recentSaveInterval: null,
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
				window.console.log('settingValue', this.value);
			}
		}
	},
	methods: {
		/**
		 * Set focus to the control input
		 */
		setFocus() {
			this.$refs.input.focus();
		},
		/**
		 * Submit the DOI edits
		 */
		triggerDoiSave() {
			this.isSaving = true;

			// TODO: Error handling here

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
			window.console.log('error', r);
		},
		/**
		 * Callback to fire when the form's submission ajax request has been
		 * returned, and the success or error callbacks have already been fired.
		 *
		 * @param {Object} r The response to the AJAX request
		 */
		complete() {
			this.isDisabled = true;
			this.isSaving = false;
		}
	},
	mounted() {
		/**
		 * Increase input padding to account for a prefix if one exists and truncate
		 * prefix if it takes up the whole input area.
		 *
		 * In at least one case, the prefix clientWidth changes shortly after mount.
		 * Unable to track down the source of this changing clientWidth, we instead
		 * wait a bit before calculating the prefix padding. The delay is
		 * generous to account for the possibility that the clientWidth update will
		 * take longer on slower machines.
		 */
		this.$nextTick(() => {
			setTimeout(() => {
				if (this.prefix) {
					this.inputStyles = {
						'padding-left':
							this.$refs.prefix.clientWidth +
							this.$refs.prefix.offsetLeft +
							'px'
					};
					this.$nextTick(() => {
						const prefixLength =
							this.$refs.prefix.clientWidth + this.$refs.prefix.offsetLeft;
						if (prefixLength > this.$refs.input.clientWidth - 20) {
							this.prefixStyles = {
								width:
									this.$refs.input.clientWidth -
									this.$refs.prefix.offsetLeft -
									80 +
									'px',
								display: 'block',
								'white-space': 'nowrap',
								'overflow-x': 'hidden',
								'text-overflow': 'ellipsis'
							};
							this.$nextTick(() => {
								this.inputStyles = {
									'padding-left':
										this.$refs.prefix.clientWidth +
										this.$refs.prefix.offsetLeft +
										'px'
								};
							});
						}
					});
				}
			}, 700);
		});

		// Set the field to disabled if optIntoEdit is passed
		if (this.optIntoEdit) {
			this.isDisabled = true;
		}

		/**
		 * Set an interval timer to check if there is a recent save
		 *
		 */
		this.recentSaveInterval = setInterval(() => {
			const expire = this.lastSaveTimestamp + 5000;
			if (this.hasRecentSave && expire < Date.now()) {
				this.hasRecentSave = false;
			} else if (!this.hasRecentSave && expire > Date.now()) {
				this.hasRecentSave = true;
			}
		}, 250);
	},
	destroyed() {
		clearInterval(this.recentSaveInterval);
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

.pkpFormField--text__input {
	width: 20em;
	display: inline-block;
}

.pkpFormField__control {
	position: relative;
}

.pkpFormField__inputPrefix {
	position: absolute;
	top: 0;
	left: 1rem;
	height: 2.5rem;
	line-height: 2.5rem;
	font-size: 0.9em;
	color: @text-light;
}

.pkpFormField__control--hasMultilingualIndicator {
	.pkpFormField--text__input {
		padding-left: 3rem;
	}

	.pkpFormField__inputPrefix {
		left: 3rem;
	}
}

.pkpFormField--text .multilingualProgress {
	position: absolute;
	top: 0;
	left: 0;

	button {
		position: absolute;
		top: 0;
		bottom: 0;
		left: 0;
		width: 2.5rem;
		height: 2.5rem;
		border: 1px solid transparent;
		border-right: @bg-border;

		&:focus {
			outline: 0;
			border-color: @primary;
			box-shadow: inset 0 -3px 0 @primary;

			.fa {
				color: @primary;
			}
		}
	}

	.fa {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
	}
}

.pkpFormField--text__input:hover + .multilingualProgress button {
	border-color: @shade;
}

.pkpFormField--text__input:focus + .multilingualProgress button {
	border-color: @primary;
}

.pkpFormField--text__optIntoEdit {
	margin-left: 0.25rem;
	margin-right: 0.25rem;
	height: 2.5rem; // Match input height
}

.pkpFormField--sizesmall {
	.pkpFormField--text__input {
		width: 10em;
	}
}

.pkpFormField--sizelarge {
	.pkpFormField--text__input {
		width: 100%;
	}

	.pkpFormField--text__optIntoEdit {
		margin-left: 0;
		margin-top: 0.25rem;
		height: inherit;
	}
}

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
