<!--
  - @copyright Copyright (c) 2018 René Gieling <github@dartcafe.de>
  -
  - @author René Gieling <github@dartcafe.de>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<div class="user_settings">
		<h3>
			{{ t('polls', 'A poll with many options and voters can have a heavy impact on client performance.') }}
			{{ t('polls', 'Set the amount of voting cells (options x participants) up to which all voting cells should be displayed.') }}
			{{ t('polls', 'If this threshold gets tresspasses only the current user will be displayed, to avoid a performance breakdown.') }}
			{{ t('polls', 'The default threshold of 1000 should be a good and safe value.') }}
		</h3>
		<InputDiv v-model="threshold"
			type="number"
			inputmode="numeric"
			use-num-modifiers
			:placeholder="'1000'"
			:modifier-step-value="100"
			:modifier-min="200" />
	</div>
</template>

<script>

import { mapState } from 'vuex'
import InputDiv from '../../Base/InputDiv.vue'

export default {
	name: 'PerformanceSettings',

	components: {
		InputDiv,
	},

	computed: {
		...mapState({
			settings: (state) => state.settings.user,
		}),

		threshold: {
			get() {
				return this.settings.performanceThreshold
			},
			set(value) {
				if (value < 1) {
					value = 1000
				}
				this.writeValue({ performanceThreshold: +value })
			},
		},
	},

	methods: {
		async writeValue(value) {
			await this.$store.commit('settings/setPreference', value)
			this.$store.dispatch('settings/write')
		},
	},
}
</script>
