<template>
	<Combobox multiple nullable v-slot="{ open: isComboboxOpen }" v-model="selectedOptions">
		<Popover class="w-full">
			<template #target="{ open: openPopover }">
				<div class="w-full">
					<ComboboxButton
						class="flex w-full items-center justify-between rounded-md bg-gray-100"
						:class="{
							'rounded-b-none': isComboboxOpen,
							'p-1': selectedOptions.length > 0,
							'px-3 py-1.5': selectedOptions.length === 0,
						}"
						@click="openPopover()"
					>
						<span
							v-if="selectedOptions.length > 0"
							class="flex w-[calc(100%-1rem)] space-x-1.5 overflow-x-scroll p-0.5 text-gray-800 scrollbar-hide"
						>
							<span
								class="flex h-6 items-center rounded-md bg-white px-2 text-sm shadow"
								v-for="option in selectedOptions"
								:key="option.value || option"
							>
								<span class="text-ellipsis whitespace-nowrap">
									{{ option.label || option.value || option }}
								</span>
							</span>
						</span>

						<span class="text-base text-gray-500" v-else>
							{{ placeholder || '' }}
						</span>

						<FeatherIcon
							name="chevron-down"
							class="h-4 w-4 text-gray-500"
							aria-hidden="true"
						/>
					</ComboboxButton>
				</div>
			</template>
			<template #body="{ close: closePopover }">
				<div
					v-show="isComboboxOpen"
					class="rounded-md rounded-t-none bg-white px-1.5 pb-1.5 shadow-md"
				>
					<ComboboxOptions class="max-h-[20rem] overflow-y-auto scrollbar-hide" static>
						<div
							class="sticky top-0 mb-1.5 flex items-stretch space-x-1.5 bg-white pt-1.5"
						>
							<ComboboxInput
								class="form-input w-full placeholder-gray-500"
								ref="input"
								type="text"
								@change="
									(e) => {
										query = e.target.value
										emit('inputChange', e.target.value)
									}
								"
								:value="query"
								autocomplete="off"
								placeholder="Search..."
							/>
							<Button icon="x" @click="selectedOptions = []" />
						</div>
						<div
							v-if="filteredOptions.length === 0"
							class="flex h-8 w-full items-center rounded bg-gray-50 px-3 text-sm font-light"
						>
							No results found
						</div>
						<ComboboxOption
							v-for="(option, idx) in filteredOptions"
							:key="idx"
							:value="option"
							v-show="filteredOptions.length > 0"
							@click="
								() => {
									$refs.input.el.focus()
									// need to manually remove option from selectedOptions
									// because clicking on an option twice doesn't unselect it
									toggleSelection(option)
								}
							"
							v-slot="{ active }"
						>
							<div
								class="flex h-8 w-full cursor-pointer items-center justify-between rounded px-3 text-base"
								:class="{
									'bg-gray-100 text-gray-800': active,
									'bg-white': !active,
								}"
							>
								<div class="flex items-baseline space-x-2">
									<span>{{ option.label }}</span>
									<span
										v-if="option.description"
										class="text-sm font-light text-gray-500"
									>
										{{ option.description }}
									</span>
								</div>
								<FeatherIcon
									name="check"
									class="h-4 w-4"
									v-show="isSelected(option)"
								/>
							</div>
						</ComboboxOption>
						<div class="sticky bottom-0 flex justify-end space-x-2 bg-white pt-2">
							<Button appearance="secondary" @click.prevent.stop="selectOrClearAll()">
								{{ selectedOptions.length > 0 ? 'Clear' : 'Select All' }}
							</Button>
							<Button
								appearance="primary"
								@click="$emit('apply', selectedOptions) || closePopover()"
							>
								Apply
							</Button>
						</div>
					</ComboboxOptions>
				</div>
			</template>
		</Popover>
	</Combobox>
</template>

<script setup>
import {
	Combobox,
	ComboboxButton,
	ComboboxInput,
	ComboboxOption,
	ComboboxOptions,
} from '@headlessui/vue'
import { computed, inject, ref } from 'vue'

const emit = defineEmits(['inputChange', 'change', 'update:modelValue', 'apply'])
const props = defineProps({
	label: {
		type: String,
		default: '',
	},
	placeholder: {
		type: String,
		default: '',
	},
	value: {
		type: Array,
	},
	modelValue: {
		type: Array,
	},
	options: {
		type: Array,
		default: [],
	},
})

const query = ref('')

const options = computed(() => makeOptions(props.options.slice()))
function makeOptions(options) {
	// make sure options are objects with a label property and a value property with no duplicates

	if (!options || options.length === 0) {
		return []
	}

	if (options[0].hasOwnProperty('label') && options[0].hasOwnProperty('value')) {
		// pass through
	}

	if (typeof options[0] === 'string') {
		options = options.map((option) => {
			return {
				label: option,
				value: option,
			}
		})
	}

	options = options.filter((option, index, self) => {
		return option && option.value && self.findIndex((t) => t.value === option.value) === index
	})

	return options
}

const valueProp = props.modelValue ? 'modelValue' : 'value'
const selectedOptions = computed({
	get() {
		return makeOptions(props[valueProp]?.slice())
	},
	set(val) {
		emit('change', val)
		emit('update:modelValue', val)
	},
})

function isSelected(option) {
	return selectedOptions.value.findIndex((t) => t.value === option.value) > -1
}
function toggleSelection(option) {
	if (isSelected(option)) {
		selectedOptions.value = selectedOptions.value.filter((t) => t.value !== option.value)
	} else {
		selectedOptions.value = [...selectedOptions.value, option]
	}
}

const $utils = inject('$utils')
const filteredOptions = computed(() => {
	return !query.value
		? options.value
		: $utils.fuzzySearch(options.value, {
				term: query.value,
				keys: ['label', 'value'],
		  })
})

const selectOrClearAll = () => {
	selectedOptions.value = selectedOptions.value.length > 0 ? [] : filteredOptions.value
}
</script>
