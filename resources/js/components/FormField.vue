<template>
    <DefaultField :field="currentField" :errors="errors" :full-width-content="true">
        <template #field>
            <div :class="{'flex flex-wrap' : !field.stack, 'border-danger': hasError}">
                <div v-for="(option, val) in field.options" :class="{'mb-2' : field.stack || field.addPadding}"  class="mlbz-radio-container">
                    <label :for="`${field.attribute}_${val}`">
                        <input v-model="value" :value="val" :id="`${field.attribute}_${val}`" :name="`${formUniqueId}_${field.attribute}`" type="radio" :disabled="field.disabled" @input="handleChange">
                        <span class="mlbz-radio-label">{{ getOptionLabel(option) }}</span>
                        <span v-if="field.stack && hasOptionHint(option)" class="mlbz-radio-hint mt-1 block text-sm text-80 leading-normal">{{ getOptionHint(option) }}</span>
                    </label>
                </div>
            </div>
        </template>
    </DefaultField>
</template>

<script>
    import HasOptions from '../mixins/HasOptions';
    import CanToggle from '../mixins/CanToggle';
    import {DependentFormField, HandlesValidationErrors} from 'laravel-nova';
    import find from "lodash/find";
    import isNil from "lodash/isNil";
    import first from "lodash/first";

    export default {
        mixins: [DependentFormField, HandlesValidationErrors, HasOptions, CanToggle],

        props: ['resourceName', 'resourceId', 'field','formUniqueId'],

        computed: {
            rawValue() {
                return this.value;
            }
        },
        methods: {
            /*
             * Set the initial, internal value for the field.
             */
            setInitialValue() {
                if (this.value) {
                    return;
                }

                if (this.field.value !== null) {
                    return this.value = this.field.value;
                }

				if (this.field.hasOwnProperty('default')) {
					return this.value = this.field.default;
				}

				return this.value = ''
            },

            /**
             * Fill the given FormData object with the field's internal value.
             */
            fill(formData) {
                const value = this.value !== null ? this.value : this.currentField.default;
                formData.append(this.currentField.attribute, value);
            },

            /**
             * Handle on synced field.
             */
            onSyncedField() {
                this.field.options = this.currentField.options;
                this.selectOption(this.currentField.default)
            },

            clearSelection() {
                this.value = null

                if (this.field) {
                    this.emitFieldValueChange(this.field.attribute, this.value)
                }
            },

            /**
             * Select the given option.
             */
            selectOption(option) {
                if (isNil(option)) {
                    this.clearSelection()
                    return
                }

                let selectedOption = find(
                    this.field.options,
                    (l, v) => v == option
                )

                if (selectedOption) {
                    this.value = option
                } else {
                    this.value = null
                }

                if (this.field) {
                    this.emitFieldValueChange(this.field.attribute, this.value)
                }
            },
        }
    }
</script>

<style>
    .mlbz-radio-container:not(:last-child) {
        margin-right: 20px;
    }

    .mlbz-radio-label {
        padding-left: 5px;
    }

    .mlbz-radio-hint {
        padding-left: 24px;
    }

    .mlbz-hidden {
        display: none !important;
    }
</style>
