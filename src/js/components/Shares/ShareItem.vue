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
	<UserItem show-email v-bind="share" :icon="true">
		<template #status>
			<div v-if="hasVoted(share.userId)">
				<VotedIcon class="vote-status voted" :title="t('polls', 'Has voted')" />
			</div>
			<div v-else-if="['public', 'group'].includes(share.type)">
				<div class="vote-status empty" />
			</div>
			<div v-else>
				<UnvotedIcon class="vote-status unvoted" :title="t('polls', 'Has not voted')" />
			</div>
		</template>

		<NcActions>
			<NcActionInput v-if="share.type === 'public'"
				:show-trailing-button="false"
				:value.sync="label"
				@input="writeLabel()">
				<template #icon>
					<EditIcon />
				</template>
				{{ t('polls', 'Share label') }}
			</NcActionInput>

			<NcActionButton v-if="share.emailAddress || share.type === 'group'" @click="sendInvitation()">
				<template #icon>
					<SendEmailIcon />
				</template>
				{{ share.invitationSent ? t('polls', 'Resend invitation mail') : t('polls', 'Send invitation mail') }}
			</NcActionButton>

			<NcActionButton v-if="share.type === 'user' || share.type === 'admin'" @click="switchAdmin()">
				<template #icon>
					<GrantAdminIcon v-if="share.type === 'user'" />
					<WithdrawAdminIcon v-else />
				</template>
				{{ share.type === 'user' ? t('polls', 'Grant poll admin access') : t('polls', 'Withdraw poll admin access') }}
			</NcActionButton>

			<NcActionButton @click="copyLink()">
				<template #icon>
					<ClippyIcon />
				</template>
				{{ t('polls', 'Copy link to clipboard') }}
			</NcActionButton>

			<NcActionButton v-if="share.URL" @click="$emit('show-qr-code')">
				<template #icon>
					<QrIcon />
				</template>
				{{ t('polls', 'Show QR code') }}
			</NcActionButton>

			<NcActionCaption v-if="share.type === 'public'" :title="t('polls', 'Options for the registration dialog')" />

			<NcActionRadio v-if="share.type === 'public'"
				name="publicPollEmail"
				value="optional"
				:checked="share.publicPollEmail === 'optional'"
				@change="setPublicPollEmail({ share, value: 'optional' })">
				{{ t('polls', 'Email address is optional') }}
			</NcActionRadio>

			<NcActionRadio v-if="share.type === 'public'"
				name="publicPollEmail"
				value="mandatory"
				:checked="share.publicPollEmail === 'mandatory'"
				@change="setPublicPollEmail({ share, value: 'mandatory' })">
				{{ t('polls', 'Email address is mandatory') }}
			</NcActionRadio>

			<NcActionRadio v-if="share.type === 'public'"
				name="publicPollEmail"
				value="disabled"
				:checked="share.publicPollEmail === 'disabled'"
				@change="setPublicPollEmail({ share, value: 'disabled' })">
				{{ t('polls', 'Do not ask for an email address') }}
			</NcActionRadio>
		</NcActions>

		<ActionDelete :title="t('polls', 'Remove share')"
			@delete="removeShare({ share })" />
	</UserItem>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import { showSuccess, showError } from '@nextcloud/dialogs'
import { NcActions, NcActionButton, NcActionCaption, NcActionInput, NcActionRadio } from '@nextcloud/vue'
import ActionDelete from '../Actions/ActionDelete.vue'
import VotedIcon from 'vue-material-design-icons/CheckboxMarked.vue'
import UnvotedIcon from 'vue-material-design-icons/MinusBox.vue'
import SendEmailIcon from 'vue-material-design-icons/EmailArrowRight.vue'
import GrantAdminIcon from 'vue-material-design-icons/ShieldCrown.vue'
import EditIcon from 'vue-material-design-icons/Pencil.vue'
import WithdrawAdminIcon from 'vue-material-design-icons/ShieldCrownOutline.vue'
import ClippyIcon from 'vue-material-design-icons/ClipboardArrowLeftOutline.vue'
import QrIcon from 'vue-material-design-icons/Qrcode.vue'

export default {
	name: 'ShareItem',

	components: {
		WithdrawAdminIcon,
		GrantAdminIcon,
		ClippyIcon,
		EditIcon,
		QrIcon,
		SendEmailIcon,
		UnvotedIcon,
		VotedIcon,
		NcActions,
		NcActionButton,
		NcActionCaption,
		NcActionInput,
		NcActionRadio,
		ActionDelete,
	},

	props: {
		share: {
			type: Object,
			default: undefined,
		},
	},

	computed: {
		...mapGetters({
			hasVoted: 'votes/hasVoted',
		}),

		label: {
			get() {
				return this.share.displayName
			},
			set(value) {
				this.$store.commit('shares/setShareProperty', { id: this.share.id, displayName: value })
			},
		},

	},

	methods: {
		...mapActions({
			removeShare: 'shares/delete',
			switchAdmin: 'shares/switchAdmin',
			setPublicPollEmail: 'shares/setPublicPollEmail',
			setLabel: 'shares/writeLabel',
		}),

		async writeLabel() {
			this.setLabel({ token: this.share.token, displayName: this.share.displayName })
		},

		async sendInvitation() {
			const response = await this.$store.dispatch('shares/sendInvitation', { share: this.share })
			if (response.data?.sentResult?.sentMails) {
				response.data.sentResult.sentMails.forEach((item) => {
					showSuccess(t('polls', 'Invitation sent to {displayName} ({emailAddress})', { emailAddress: item.emailAddress, displayName: item.displayName }))
				})
			}
			if (response.data?.sentResult?.abortedMails) {
				response.data.sentResult.abortedMails.forEach((item) => {
					console.error('Mail could not be sent!', { recipient: item })
					showError(t('polls', 'Error sending invitation to {displayName} ({emailAddress})', { emailAddress: item.emailAddress, displayName: item.displayName }))
				})
			}
		},

		copyLink() {
			try {
				navigator.clipboard.writeText(this.share.URL)
				showSuccess(t('polls', 'Link copied to clipboard'))
			} catch {
				showError(t('polls', 'Error while copying link to clipboard'))
			}
		},
	},
}
</script>

<style lang="scss">
.vote-status {
	margin-left: 8px;
	width: 32px;

	&.voted {
		color: var(--color-polls-foreground-yes)
	}

	&.unvoted {
		color: var(--color-polls-foreground-no)
	}
}

</style>
