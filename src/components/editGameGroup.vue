<script setup lang="ts">
import { assertExpressionStatement } from '@babel/types';
import $ from 'jquery';
import popup from './popup.vue';
import SelectMultiple from './selectMultiple.vue';
</script>

<script lang="ts">
export default {
	data() {
		return {
			permissions: [
				{ id: 'balance', name: 'Balance' },
				{ id: 'ban', name: 'Ban' },
				{ id: 'cameraman', name: 'Cameraman' },
				{ id: 'canseeadminchat', name: 'Can see Admin Chat' },
				{ id: 'changemap', name: 'Change Map' },
				{ id: 'chat', name: 'Chat' },
				{ id: 'cheat', name: 'Cheat' },
				{ id: 'clientdemos', name: 'Client Demos' },
				{ id: 'config', name: 'Config' },
				{ id: 'debug', name: 'Debug' },
				{ id: 'demos', name: 'Demos' },
				{ id: 'featuretest', name: 'Feature Test' },
				{ id: 'forceteamchange', name: 'Force Team Change' },
				{ id: 'immune', name: 'Immune' },
				{ id: 'kick', name: 'Kick' },
				{ id: 'manageserver', name: 'Manage Server' },
				{ id: 'pause', name: 'Pause' },
				{ id: 'private', name: 'Private' },
				{ id: 'reserve', name: 'Reserve' },
				{ id: 'startvote', name: 'Start Vote' },
				{ id: 'teamchange', name: 'Team Change' },
			],
			discord_roles: [] as Array<any>,
			extRet: {
				discord_roles: [] as Array<any>,
				group_permissions: [] as Array<any>,
			},
		};
	},
	props: {
		group_data: null as any,
	},
	methods: {
		confirmBtnClick(dt: any) {
			console.log(dt);
			const createUrl = '/api/gameGroups/write/newGroup';
			const updateUrl = '/api/gameGroups/write/editGroup'
			const data = { ...dt }
			if (this.group_data)
				data._id = this.group_data._id
			$.ajax({
				url: this.group_data ? updateUrl : createUrl,
				type: 'post',
				dataType: 'json',
				data: JSON.stringify(data),
				contentType: 'application/json',
				success: (dt) => {
					console.log(dt);
					if (this.group_data)
						this.$emit('edited', dt);
					else
						this.$emit('new_game_group', dt);
					this.$emit('cancelBtnClick');
				},
				error: (err) => {
					console.error(err);
					const compPopup: any = this.$refs.popupComp;
					compPopup.blinkAll();
					//.blinkBgColor(this.$refs.popupLogin.getInputs());
				},
			});
		},
		getAllDiscordRoles: function () {
			fetch('/api/discord/read/getRoles')
				.then((res) => res.json())
				.then((dt) => {
					this.discord_roles = dt;
				});
		},
		getCustomPermissions: async function () {
			fetch('/api/custom_permissions/read/getAll')
				.then((res) => res.json())
				.then((dt) => {
					this.permissions = this.permissions.concat(dt.map((e: any) => ({ id: e.permission, name: e.name })))
				});
		}
	},
	created() {
		this.getCustomPermissions();
		this.getAllDiscordRoles();
		console.log(this.group_data);
	},
	components: { popup, SelectMultiple },
};
</script>

<template>
	<popup ref="popupComp" :title="`${group_data ? 'Edit' : 'Add'} Group`" @cancelBtnClick="$emit('cancelBtnClick', $event)" @confirmBtnClick="confirmBtnClick" :extRetProp="extRet">
		<input name="group_name" type="text" placeholder="Group Name" :value="group_data?.group_name" regex="^[a-zA-Z\d]{2,}$" />
		<!-- <select name="group_permissions" multiple>
			<option v-for="p in permissions.sort()" :value="p" :selected="group_data.group_permissions.includes(p)">{{ p }}</option>
		</select> -->
		<SelectMultiple :elements="permissions" oIdKey="id" oTitleKey="name" title="Permissions" :preselect="group_data?.group_permissions" @selectChanged="extRet.group_permissions = $event" />
		<SelectMultiple :elements="discord_roles" oIdKey="id" oTitleKey="name" title="Discord Roles" :preselect="group_data?.discord_roles" @selectChanged="extRet.discord_roles = $event" />

		<label>Require Approval<input name="require_appr" type="checkbox" placeholder="Require Approval" :checked="group_data?.require_appr" /></label>
	</popup>
</template>

<style scoped></style>
