<template>
	<div class="w-fit select-none space-y-3 bg-white p-1 text-base">
		<div class="flex items-center text-gray-700">
			<div
				class="flex h-6 w-6 cursor-pointer items-center justify-center rounded-md hover:bg-gray-100"
			>
				<FeatherIcon @click="prevMonth" name="chevron-left" class="h-5 w-5" />
			</div>
			<div class="flex-1 text-center text-lg font-medium text-blue-500">
				{{ formatMonth }}
			</div>
			<div
				class="flex h-6 w-6 cursor-pointer items-center justify-center rounded-md hover:bg-gray-100"
			>
				<FeatherIcon @click="nextMonth" name="chevron-right" class="h-5 w-5" />
			</div>
		</div>
		<div class="flex space-x-2">
			<Input type="text" :value="value" @change="selectDate(getDate($event))"></Input>
			<Button class="h-7" @click="selectDate(getDate())"> Today </Button>
		</div>
		<div class="mt-2 flex flex-col items-center justify-center text-base">
			<div class="flex w-full items-center space-x-1 text-gray-600">
				<div
					class="flex h-[30px] w-[30px] items-center justify-center text-center"
					v-for="(d, i) in ['S', 'M', 'T', 'W', 'T', 'F', 'S']"
					:key="i"
				>
					{{ d }}
				</div>
			</div>
			<div v-for="(week, i) in datesAsWeeks" :key="i" class="mt-1">
				<div class="flex w-full items-center">
					<div
						v-for="date in week"
						:key="toValue(date)"
						class="mr-1 flex h-[30px] w-[30px] cursor-pointer items-center justify-center rounded-md last:mr-0 hover:bg-blue-50 hover:text-blue-500"
						:class="{
							'text-gray-500': date.getMonth() !== currentMonth - 1,
							'text-blue-500': toValue(date) === toValue(today),
							'bg-blue-50 font-semibold text-blue-500': toValue(date) === value,
						}"
						@click="() => selectDate(date)"
					>
						{{ date.getDate() }}
					</div>
				</div>
			</div>
		</div>

		<div class="mt-1 flex w-full justify-end">
			<div
				class="cursor-pointer rounded-md px-2 py-1 hover:bg-gray-100"
				@click="selectDate('')"
			>
				Clear
			</div>
		</div>
	</div>
</template>

<script>
export default {
	name: 'DatePicker',
	props: ['value', 'placeholder', 'readonly', 'inputClass'],
	emits: ['change'],
	data() {
		return {
			currentYear: null,
			currentMonth: null,
		}
	},
	created() {
		this.selectCurrentMonthYear()
	},
	computed: {
		today() {
			return this.getDate()
		},
		datesAsWeeks() {
			let datesAsWeeks = []
			let dates = this.dates.slice()
			while (dates.length) {
				let week = dates.splice(0, 7)
				datesAsWeeks.push(week)
			}
			return datesAsWeeks
		},
		dates() {
			if (!(this.currentYear && this.currentMonth)) {
				return []
			}
			let monthIndex = this.currentMonth - 1
			let year = this.currentYear

			let firstDayOfMonth = this.getDate(year, monthIndex, 1)
			let lastDayOfMonth = this.getDate(year, monthIndex + 1, 0)
			let leftPaddingCount = firstDayOfMonth.getDay()
			let rightPaddingCount = 6 - lastDayOfMonth.getDay()

			let leftPadding = this.getDatesAfter(firstDayOfMonth, -leftPaddingCount)
			let rightPadding = this.getDatesAfter(lastDayOfMonth, rightPaddingCount)
			let daysInMonth = this.getDaysInMonth(monthIndex, year)
			let datesInMonth = this.getDatesAfter(firstDayOfMonth, daysInMonth - 1)

			let dates = [...leftPadding, firstDayOfMonth, ...datesInMonth, ...rightPadding]
			if (dates.length < 42) {
				const finalPadding = this.getDatesAfter(dates.at(-1), 42 - dates.length)
				dates = dates.concat(...finalPadding)
			}
			return dates
		},
		formatMonth() {
			let date = this.getDate(this.currentYear, this.currentMonth - 1, 1)
			return date.toLocaleString('en-US', {
				month: 'short',
				year: 'numeric',
			})
		},
	},
	methods: {
		selectDate(date) {
			this.$emit('change', this.toValue(date))
		},
		selectCurrentMonthYear() {
			let date = this.value ? this.getDate(this.value) : this.getDate()
			if (date === 'Invalid Date') {
				date = this.getDate()
			}
			this.currentYear = date.getFullYear()
			this.currentMonth = date.getMonth() + 1
		},
		prevMonth() {
			this.changeMonth(-1)
		},
		nextMonth() {
			this.changeMonth(1)
		},
		changeMonth(adder) {
			this.currentMonth = this.currentMonth + adder
			if (this.currentMonth < 1) {
				this.currentMonth = 12
				this.currentYear = this.currentYear - 1
			}
			if (this.currentMonth > 12) {
				this.currentMonth = 1
				this.currentYear = this.currentYear + 1
			}
		},
		getDatesAfter(date, count) {
			let incrementer = 1
			if (count < 0) {
				incrementer = -1
				count = Math.abs(count)
			}
			let dates = []
			while (count) {
				date = this.getDate(
					date.getFullYear(),
					date.getMonth(),
					date.getDate() + incrementer
				)
				dates.push(date)
				count--
			}
			if (incrementer === -1) {
				return dates.reverse()
			}
			return dates
		},

		getDaysInMonth(monthIndex, year) {
			let daysInMonthMap = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
			let daysInMonth = daysInMonthMap[monthIndex]
			if (monthIndex === 1 && this.isLeapYear(year)) {
				return 29
			}
			return daysInMonth
		},

		isLeapYear(year) {
			if (year % 400 === 0) return true
			if (year % 100 === 0) return false
			if (year % 4 === 0) return true
			return false
		},

		toValue(date) {
			if (!date) {
				return ''
			}

			// toISOString is buggy and reduces the day by one
			// this is because it considers the UTC timestamp
			// in order to circumvent that we need to use luxon/moment
			// but that refactor could take some time, so fixing the time difference
			// as suggested in this answer.
			// https://stackoverflow.com/a/16084846/3541205
			date.setHours(0, -date.getTimezoneOffset(), 0, 0)
			return date.toISOString().slice(0, 10)
		},

		getDate(...args) {
			let d = new Date(...args)
			return d
		},
	},
}
</script>
