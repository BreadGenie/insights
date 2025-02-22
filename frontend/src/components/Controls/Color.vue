<template>
	<div>
		<div class="mb-1 text-sm text-gray-500" v-if="label">
			{{ label }}
		</div>
		<Popover placement="bottom-start" class="w-full">
			<template #target="{ togglePopover }">
				<div
					tabindex="0"
					:class="inputClass"
					class="flex h-8 w-full items-center rounded-md bg-gray-100 px-3 placeholder-gray-500 focus:outline-none"
					@click="togglePopover()"
					readonly
				>
					<div class="flex items-center">
						<div
							class="flex items-center space-x-2"
							v-if="(!multiple && value) || (multiple && value.length > 0)"
						>
							<div class="flex -space-x-2 overflow-hidden">
								<div
									v-for="(color, idx) in multiple && value.length > 0
										? value
										: [value]"
									:key="idx"
									class="h-4 w-4 rounded-full"
									:style="{ backgroundColor: color }"
								></div>
							</div>
							<span>
								{{ selectedColorLabel }}
							</span>
						</div>
						<span class="text-gray-400" v-else>
							{{ placeholder || label }}
						</span>
					</div>
				</div>
			</template>
			<template #body>
				<div class="mt-1 w-fit rounded-md border bg-white p-3 pt-2 text-center shadow-md">
					<div>
						<div class="inline-grid grid-cols-5 gap-3 border-b border-none">
							<div
								class="col-span-5 text-left text-sm uppercase tracking-wide text-gray-500"
							>
								Select Color
							</div>
							<div
								v-for="color in colors"
								:key="color"
								class="flex h-5 w-5 cursor-pointer items-center justify-center rounded bg-opacity-50"
								:class="[
									isSelected(color) ? 'bg-opacity-50 shadow' : 'bg-opacity-100',
								]"
								:style="{
									backgroundColor: color,
								}"
								@click="toggleColor(color)"
							>
								<FeatherIcon
									v-if="isSelected(color)"
									class="h-4 w-4 text-white"
									name="check"
									@click.prevent.stop="toggleColor(color)"
								></FeatherIcon>
							</div>
							<div
								class="col-span-5 text-left text-sm uppercase tracking-wide text-gray-500"
							>
								Hex
							</div>
							<input
								ref="input"
								type="text"
								placeholder="Custom Hex"
								:class="inputClass"
								class="col-span-5 -mt-1 flex h-8 items-center rounded-md border-0 bg-gray-100 px-3 text-base placeholder-gray-500 focus:outline-none"
								@change="(e) => setColorValue(e.target.value)"
							/>
						</div>
					</div>
				</div>
			</template>
		</Popover>
	</div>
</template>

<script>
import { COLOR_MAP } from '@/utils/colors'

export default {
	name: 'Color',
	props: {
		label: String,
		modelValue: [String, Array],
		options: Array,
		inputClass: String,
		placeholder: String,
		autofocus: Boolean,
		multiple: Boolean,
		max: Number,
	},
	emits: ['focus', 'update:modelValue'],
	mounted() {
		if (this.autofocus) {
			this.focus()
		}
	},
	methods: {
		focus() {
			if (this.$refs.input && this.$refs.input.focus) {
				this.$refs.input.focus()
			}
		},
		setColorValue(value) {
			if (this.multiple && this.max) {
				if (this.value.length >= this.max) {
					return
				}
			}
			if (this.isValidColor(value)) {
				this.triggerChange(value)
			}
		},
		isValidColor(value) {
			if (!value.startsWith('#')) {
				value = '#' + value
			}
			if (/^#[0-9A-F]{6}$/i.test(value)) {
				return true
			}
			return false
		},
		triggerChange(value) {
			if (value === '') {
				value = null
			}
			if (this.multiple) {
				this.$emit('update:modelValue', [...this.modelValue, value])
			} else {
				this.$emit('update:modelValue', value)
			}
		},
		isSelected(color) {
			if (this.multiple) {
				return this.value.includes(color)
			}
			return this.value === color
		},
		toggleColor(color) {
			if (this.isSelected(color)) {
				this.removeColor(color)
			} else {
				this.setColorValue(color)
			}
		},
		removeColor(color) {
			if (this.multiple) {
				const value = this.modelValue.filter((c) => c !== color)
				this.$emit('update:modelValue', value)
			}
		},
	},
	computed: {
		value: {
			get() {
				if (this.multiple) {
					!this.modelValue && this.$emit('update:modelValue', [])
					return this.modelValue || []
				}
				return Array.isArray(this.modelValue) ? this.modelValue[0] : this.modelValue
			},
			set(value) {
				this.setColorValue(value)
			},
		},
		colors() {
			return this.options || Object.values(COLOR_MAP)
		},
		selectedColorLabel() {
			return this.multiple
				? this.modelValue.length > 1
					? `${this.modelValue.length} colors`
					: this.value[0]
				: this.value
		},
	},
}
</script>
