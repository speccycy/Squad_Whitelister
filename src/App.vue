<script setup lang="ts">
import levenshtein from 'js-levenshtein';

import login from './components/login.vue';
import registration from './components/registration.vue';
import tabBrowser from './components/tabBrowser.vue';
import tabBrowserBtn from './components/tabBrowserButton.vue';
import tab from './components/tab.vue';
import AddNewClan from './components/addNewClan.vue';
import EditClan from './components/editClan.vue';
import blackoutBackground from './components/blackoutBackground.vue';
import ClanCard from './components/clanCard.vue';
import popup from './components/popup.vue';
import confirmPopup from './components/confirmPopup.vue';
import gameGroupCard from './components/gameGroupCard.vue';
import editGameGroup from './components/editGameGroup.vue';
import editClanUsers from './components/editClanUsers.vue';
import whitelistTab from './components/whitelistTab.vue';
import addNewWhitelistUser from './components/addNewWhitelistUser.vue';
import thanksCard from './components/thanksCard.vue';
import approvalsTab from './components/approvalsTab.vue';
import importWhitelist from './components/importWhitelist.vue';
import userCard from './components/userCard.vue';
import changepassword from './components/changepassword.vue';
import addNewList from './components/addNewList.vue';
import editList from './components/editList.vue';
import ConfigTab from './components/configTab.vue';
import SeedingTab from './components/seedingTab.vue';
import ApiTab from './components/apiTab.vue';
import addEditApiKey from './components/addEditApiKey.vue';

import bia_logo from './assets/bia_logo.png';
import jd_logo from './assets/jd_logo.png';
// import oasis_logo from './assets/oasishostinglogo.svg';
</script>

<script lang="ts">
import MobileDetect from 'mobile-detect';
var md = new MobileDetect(window.navigator.userAgent);
export default {
	data() {
		return {
			mainRef: null as any,
			user_session: null as any,
			loginRequired: true,
			app_title: 'Squad Whitelister',
			header_title_hidden: false,
			accent_color: '#ffc40b',
			currentTab: 'Home',
			logo_url: '',
			logo_border_radius: 10,
			version: '',
			tabs: [],
			tabBtns: [],
			popups: {
				addingNewClan: false,
				addingNewGameGroup: false,
				creatingNewRole: false,
				confirm: false,
				registration: false,
				login: false,
				editClan: false,
				editGameGroup: false,
				editClanUsers: false,
				addNewWhitelistUser: false,
				importWhitelist: false,
				changepassword: false,
				addNewList: false,
				editList: false,
				addEditApiKey: false,
			},
			favicon: '',
			clans: [] as Array<any>,
			inEditingClan: -1,
			game_groups: [] as Array<any>,
			inEditingGroup: -1,
			inUserEditingClan: -1,
			pointers: {
				confirmPopup: {} as any,
				editClanUsers: {} as any,
			},
			roles: [] as any[],
			tabData: {
				Whitelist: {
					add_data: {} as any,
					new_list_data: {} as any,
					edit_list_data: {} as any,
				},
				Groups: {
					editor: false,
				},
				UsersAndRoles: {
					users: [] as Array<any>,
					roles: [] as Array<any>,
					userSearch: '',
				},
				api_keys: {
					add_data: [] as any,
				},
			},
		};
	},
	methods: {
		getAppPersonalization: function () {
			fetch('/api/getAppPersonalization')
				.then((res) => res.json())
				.then((dt) => {
					this.app_title = dt.name;
					this.accent_color = dt.accent_color;
					this.logo_url = dt.logo_url;
					this.header_title_hidden = dt.title_hidden_in_header;
					this.logo_border_radius = dt.logo_border_radius;
					this.favicon = dt.favicon;
				});
		},
		checkSession: async function () {
			await fetch('/api/checkSession')
				.then((res) => res.json())
				.then((dt) => {
					this.user_session = dt.userSession;
					this.setLoginRequired(dt.status == 'login_required');
					console.log('User session', this.user_session);
				});
		},
		getTabs: async function () {
			await fetch('/api/getTabs')
				.then((res) => res.json())
				.then((dt) => {
					console.log(dt);
					this.tabs = dt.tabs;
					this.tabs.forEach((e) => this.tabBtns.push(e[ 'name' ]));
					// this.$nextTick(() => {
					// 	this.setCurrentTab(this.tabs[0]['name']);
					// });1
					// this.setCurrentTab(this.tabs[ 0 ][ 'name' ]);
				});
		},
		logout: function () {
			fetch('/api/logout')
				.then((res) => res.json())
				.then((dt) => {
					this.setLoginRequired(dt.status == 'logout_ok');
				});
		},
		getVersion: function () {
			fetch('/api/getVersion')
				.then((res) => res.text())
				.then((dt) => {
					this.version = dt;
				});
		},
		setLoginRequired: function (required: boolean) {
			if (this.currentTab != 'Root User Registration') {
				this.loginRequired = required;
				this.popups.login = required;
				this.popups.registration = required;
				if (required)
					setTimeout(() => {
						const elm = document.getElementsByTagName('input')[ 0 ];
						console.log(elm);
						elm.focus();
					}, 10);
				if (required)
					this.setCurrentTab('Login');
			}
		},
		setCurrentTab: function (ct: string) {
			if (this.currentTab == ct)
				return;
			this.currentTab = ct;
			const path = '/' + ct.toLowerCase().replace(/ /g, '-');
			window.history.pushState({}, '', path);
		},
		appendNewClan: function (dt: any) {
			console.log('Appending new clan', dt.clan_data);
			this.clans.push(dt.clan_data);
			// this.clans.sort((a, b) => (a.full_name - b.full_name));
		},
		appendNewGroup: function (dt: any) {
			console.log('Appending new group', dt.clan_data);
			this.game_groups.push(dt.data);
			this.popups.addingNewGameGroup = false;
			// this.clans.sort((a, b) => (a.full_name - b.full_name));
		},
		getUsers: function () {
			fetch('/api/users/read/getAll')
				.then((res) => res.json())
				.then((dt) => {
					console.log('All users', dt);
					this.tabData.UsersAndRoles.users = dt;
				});
		},
		getRoles: function () {
			fetch('/api/roles/read/getAll')
				.then((res) => res.json())
				.then((dt) => {
					console.log('All roles', dt);
					this.tabData.UsersAndRoles.roles = dt;
				});
		},
		getRolesRet: async function () {
			return await (await fetch('/api/roles/read/getAll')).json();
		},
		getClans: function () {
			fetch('/api/clans/getAllClans')
				.then((res) => res.json())
				.then((dt) => {
					console.log('All clans', dt);
					this.clans = dt;
				});
		},
		getGameGroups: function () {
			fetch('/api/gameGroups/read/getAllGroups')
				.then((res) => res.json())
				.then((dt) => {
					console.log('All groups', dt);
					this.game_groups = dt;
				});
		},
		confirmEvt: function (e: any) {
			this.confirmShow(e.title, e.text || e.message, e.callback);
		},
		confirmShow: function (title = '', text = '', callback: any) {
			this.popups.confirm = true;
			const cbData = {
				closePopup: () => {
					this.popups.confirm = false;
				},
			};
			if (this.pointers.confirmPopup != null) {
				this.pointers.confirmPopup.setProps(title, text);
				this.pointers.confirmPopup.setCallback(() => {
					callback(cbData);
				});
			}
		},
		clearAdminList: function (dt: any) {
			console.log('clearing entire admin list');

			this.confirmShow('Confirm clearing?', 'Do you really want to delete entire list for this clan?', () => {
				dt.callback();
				this.popups.confirm = false;
			});
		},
		removeClan: function (dt: any) {
			this.confirmShow('Confirm deletion?', 'Do you really want to delete ' + dt.clan_data.tag + ' clan?', () => {
				dt.callback(() => {
					this.clans = this.clans.filter((a) => a._id != dt.clan_data._id);
					this.popups.confirm = false;
				});
			});
		},
		removeWhitelistPlayer: function (dt: any) {
			this.confirmShow('Confirm deletion?', 'Do you really want to delete ' + dt.wl_data.username + '?', () => {
				dt.callback(() => {
					//this.clans = this.clans.filter((a) => a._id != dt.clan_data._id)
					//dt.callback();
					this.popups.confirm = false;
				});
			});
		},
		removeGroup: function (dt: any) {
			this.confirmShow('Confirm deletion?', 'Do you really want to delete ' + dt.group_data.group_name + ' group?', () => {
				dt.callback(() => {
					this.game_groups = this.game_groups.filter((a) => a._id != dt.group_data._id);
					this.popups.confirm = false;
				});
			});
		},
		removeUser: function (e: any) {
			this.confirmShow('Delete User?', 'Do you really want to delete ' + e.username + '?', () => {
				e.callback(() => {
					this.popups.confirm = false;
					this.tabData.UsersAndRoles.users = this.tabData.UsersAndRoles.users.filter((r) => r._id != e._id);
				});
			});
		},
		updateJson: function (config: any, emptyConfFile: any) {
			for (let k in emptyConfFile) {
				const objType = Object.prototype.toString.call(emptyConfFile[ k ]);
				const parentObjType = Object.prototype.toString.call(emptyConfFile);
				if (config[ k ] == undefined || (config[ k ] && parentObjType == '[object Array]' && !config[ k ].includes(emptyConfFile[ k ]))) {
					switch (objType) {
						case '[object Object]':
							config[ k ] = {};
							break;
						case '[object Array]':
							config[ k ] = [];
							break;

						default:
							//console.log("CONFIG:", config, "\nKEY:", k, "\nCONFIG_K:", config[k], "\nEMPTY_CONFIG_K:", emptyConfFile[k], "\nPARENT_TYPE:",parentObjType,"\n");
							if (parentObjType == '[object Array]') config.push(emptyConfFile[ k ]);
							else config[ k ] = emptyConfFile[ k ];
							break;
					}
				}
				if (typeof emptyConfFile[ k ] === 'object') {
					this.updateJson(config[ k ], emptyConfFile[ k ]);
				}
			}
		},
		checkPerms: function (url: string, callback = (dt: any) => { }) {
			fetch(url)
				.then((res) => res.json())
				.then((dt) => {
					console.log('editor?', dt);
					callback(dt);
				});
		},
		evtTest: function (e: any) {
			console.log('evt test: ', e);
		},
		log: console.log,
	},
	beforeUnmount() {
		window.removeEventListener('popstate', this.handlePopState);
	},
	async mounted() {
		this.getVersion();
		await this.getTabs();

		const path = window.location.pathname.slice(1);
		if (path) {
			console.log('tabs', this.tabs)
			const matchingTab: any = this.tabs.filter((tab: any) => tab != null && Object.keys(tab).length > 0).find((tab: any) => tab.name.toLowerCase().replace(/ /g, '-') === path);
			console.log(matchingTab)
			if (matchingTab)
				this.currentTab = matchingTab.name;
			else
				this.currentTab = this.tabs[ 0 ][ 'name' ];
		} else
			this.currentTab = this.tabs[ 0 ][ 'name' ];

		window.addEventListener('popstate', (event) => {
			const path = window.location.pathname.slice(1);
			if (path) {
				console.log('tabs', this.tabs)
				const matchingTab: any = this.tabs.filter((tab: any) => tab != null && Object.keys(tab).length > 0).find((tab: any) => tab.name.toLowerCase().replace(/ /g, '-') === path);
				console.log(matchingTab)
				if (matchingTab)
					this.currentTab = matchingTab.name;
				else
					this.currentTab = this.tabs[ 0 ][ 'name' ];
			} else
				this.currentTab = this.tabs[ 0 ][ 'name' ];
		});

		console.log("this.currentTab", this.currentTab)

		this.checkSession();
		this.getAppPersonalization();
		// console.log('levenshtein', levenshtein, 'mobile', md.mobile());
		if (md.mobile() != null) {
			document.body.classList.add('mobile');
		}
		this.roles = await this.getRolesRet();
		this.log('Roles', this.roles);
		// this.getClans();
	},
};
</script>

<template>
	<title>{{ currentTab ? currentTab + ' | ' : '' }}{{ app_title }}</title>
	<header>
		<img alt="Squad Whitelister Logo" class="logo" :src="logo_url" :style="{ 'border-radius': logo_border_radius + 'px' }" />
		<h1 v-if="!header_title_hidden">{{ app_title }}</h1>
		<div id="hdBtnContainer">
			<button v-if="!loginRequired" @click="popups.changepassword = true">Change Password</button>
			<button v-if="!loginRequired" @click="logout">Logout</button>
			<button v-if="loginRequired" @click="popups.login = true">Login</button>
			<button v-if="loginRequired" @click="popups.registration = true">Sign Up</button>
		</div>
		<tabBrowser :visible="!loginRequired">
			<tabBrowserBtn v-for="t in tabBtns" :key="t" @updateTab="setCurrentTab" :title="t" :currentTab="currentTab" />
		</tabBrowser>
	</header>
	<main>
		<blackoutBackground
			v-show="popups.addingNewClan ||
				popups.addingNewGameGroup ||
				popups.confirm ||
				popups.login ||
				popups.registration ||
				popups.editClan ||
				popups.editGameGroup ||
				popups.editClanUsers ||
				popups.addNewWhitelistUser ||
				popups.importWhitelist ||
				popups.changepassword ||
				popups.addNewList ||
				popups.editList ||
				popups.addEditApiKey
				">
			<login
				v-if="popups.login"
				@cancelBtnClick="popups.login = false"
				@login_done="
					loginRequired = false;
				popups.login = false;
				" />
			<registration v-if="popups.registration" @cancelBtnClick="popups.registration = false" />
			<changepassword v-if="popups.changepassword" @cancelBtnClick="popups.changepassword = false" />
			<editGameGroup v-if="popups.editGameGroup || popups.addingNewGameGroup" @cancelBtnClick="popups.editGameGroup = false; popups.addingNewGameGroup = false; inEditingGroup = -1" :group_data="game_groups[ inEditingGroup ]" @new_game_group="appendNewGroup" @edited="game_groups[ inEditingGroup ] = $event" />
			<confirmPopup
				:ref="(el: any) => {
					pointers.confirmPopup = el;
				}
					"
				v-show="popups.confirm"
				@cancelBtnClick="popups.confirm = false" />
			<AddNewClan v-if="popups.addingNewClan" @cancelBtnClick="popups.addingNewClan = false" @new_clan="appendNewClan" />
			<editClan v-if="popups.editClan" @cancelBtnClick="popups.editClan = false" :clan_data="clans[ inEditingClan ]" @clan_edited="clans[ inEditingClan ] = $event" />
			<editClanUsers v-if="popups.editClanUsers" :clan_data="clans[ inUserEditingClan ]" @cancelBtnClick="popups.editClanUsers = false" />
			<addNewWhitelistUser v-if="popups.addNewWhitelistUser" @cancelBtnClick="popups.addNewWhitelistUser = false" :add_data="tabData.Whitelist.add_data" />
			<importWhitelist v-if="popups.importWhitelist" @cancelBtnClick="popups.importWhitelist = false" :add_data="tabData.Whitelist.add_data" />
			<addNewList v-if="popups.addNewList" @cancelBtnClick="popups.addNewList = false" :add_data="tabData.Whitelist.new_list_data" />
			<editList v-if="popups.editList" @cancelBtnClick="popups.editList = false" :data="tabData.Whitelist.edit_list_data" />
			<addEditApiKey v-if="popups.addEditApiKey" @cancelBtnClick="popups.addEditApiKey = false" :add_data="tabData.api_keys.add_data" :roles="roles" />
		</blackoutBackground>
		<!--<button @click="setLoginRequired(!loginRequired)">Toggle</button>-->
		<tab v-if="currentTab == 'Home'" :currentTab="currentTab"></tab>
		<tab v-else-if="currentTab == 'Root User Registration'" :currentTab="currentTab">
			<registration :root_user_registration="true"></registration>
		</tab>
		<tab v-else-if="currentTab == 'Updating'" :currentTab="currentTab">
			<popup ref="popupLogin" title="Update in progress" :hide-confirm="true" :hide-cancel="true" style="margin: auto" />
		</tab>
		<tab v-else-if="currentTab == 'Clans'" :currentTab="currentTab" :horizontal="true" @vue:mounted="getClans">
			<button class="addNewClan clanCard" @click="popups.addingNewClan = true"></button>
			<ClanCard
				@confirm="removeClan"
				v-for="c in clans"
				:key="c"
				class="clanCard shadow"
				:clan_data="c"
				@edit_clan="
					popups.editClan = true;
				inEditingClan = clans.indexOf(c);
				"
				@edit_clan_users="
					popups.editClanUsers = true;
				inUserEditingClan = clans.indexOf(c);
				" />
		</tab>
		<tab
			v-else-if="currentTab == 'Groups'"
			:currentTab="currentTab"
			@vue:mounted="() => {
				getGameGroups();
				checkPerms('/api/gameGroups/write/checkPerm', (dt) => {
					tabData.Groups.editor = dt ? true : false;
				});
			}
				">
			<button v-if="tabData.Groups.editor" class="addNewGameGroup" @click="popups.addingNewGameGroup = true"></button>
			<gameGroupCard
				@confirm="removeGroup"
				v-for="g in game_groups"
				:key="g"
				class="gameGroupCard shadow"
				:group_data="g"
				@edit="
					popups.editGameGroup = true;
				inEditingGroup = game_groups.indexOf(g);
				"
				:hoverMenuVisible="tabData.Groups.editor" />
		</tab>
		<tab v-else-if="currentTab == 'Roles'" :currentTab="currentTab">
			<button class="createRole" @click="popups.creatingNewRole = true"></button>
		</tab>
		<tab v-else-if="currentTab == 'Whitelist'" :currentTab="currentTab">
			<whitelistTab
				:user_session="user_session"
				@addNewWhitelistUser="
					popups.addNewWhitelistUser = true;
				tabData.Whitelist.add_data = $event;
				"
				@confirm="removeWhitelistPlayer"
				@import_whitelist="
					popups.importWhitelist = true;
				tabData.Whitelist.add_data = $event;
				"
				@confirm_clearing="clearAdminList"
				@addNewList="
					popups.addNewList = true;
				tabData.Whitelist.new_list_data = $event;
				"
				@confirm_standard="confirmEvt"
				@editList="
					popups.editList = true;
				tabData.Whitelist.edit_list_data = $event;
				" />
		</tab>
		<tab v-else-if="currentTab == 'Approvals'" :currentTab="currentTab">
			<approvalsTab />
		</tab>
		<tab v-else-if="currentTab == 'Seeding'" :currentTab="currentTab">
			<SeedingTab />
		</tab>
		<tab
			v-else-if="currentTab == 'Users and Roles'"
			:currentTab="currentTab"
			@vue:mounted="() => {
				getUsers();
				getRoles();
			}
				">
			<input type="search" placeholder="Search User" name="usrSearch" id="" v-model="tabData.UsersAndRoles.userSearch" />
			<userCard
				v-for="u in tabData.UsersAndRoles.users"
				:key="u"
				v-show="u.username.toLowerCase().startsWith(tabData.UsersAndRoles.userSearch.toLowerCase()) ||
					levenshtein(u.username.toLowerCase(), tabData.UsersAndRoles.userSearch.toLowerCase()) <= 2 ||
					tabData.UsersAndRoles.userSearch == ''
					"
				:user_data="u"
				:roles="tabData.UsersAndRoles.roles"
				@delete-record="removeUser" />
		</tab>
		<tab v-else-if="currentTab == 'API'" :currentTab="currentTab" :roles="roles" complex>
			<ApiTab
				:roles="roles"
				@confirm="confirmEvt"
				@addNewApiKey="
					popups.addEditApiKey = true;
				tabData.api_keys.add_data = $event;
				" />
		</tab>
		<tab v-else-if="currentTab == 'Configuration'" :currentTab="currentTab" complex>
			<ConfigTab @confirm="confirmEvt" />
		</tab>
	</main>

	<footer>
		<h2>Official Support</h2>
		<div style="display: flex; flex-wrap: wrap; justify-content: center">
			<thanksCard title="JetDave Development | Fantino Davide" :src="jd_logo" discord="https://discord.com/invite/5hfcjNYdCP" github="https://github.com/fantinodavide/Squad_Whitelister" />
		</div>
		<h2>Special Thanks</h2>
		<div style="display: flex; flex-wrap: wrap; justify-content: center">
			<thanksCard title="Brigata Italiana Ariete" :src="bia_logo" website="https://biaclan.it/" discord="https://discord.gg/dXHVfQZcxJ" />
			<thanksCard
				title="Offworld Industries"
				website="https://www.offworldindustries.com/"
				src="https://www.offworldindustries.com/wp-content/themes/owitheme/img/logo_white.svg"
				discord="https://discord.com/invite/kRkqJgXW" />
			<!-- <thanksCard title="GameDash" website="https://gamedash.io/" src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4KPCEtLSBHZW5lcmF0b3I6IEFkb2JlIElsbHVzdHJhdG9yIDI0LjEuMiwgU1ZHIEV4cG9ydCBQbHVnLUluIC4gU1ZHIFZlcnNpb246IDYuMDAgQnVpbGQgMCkgIC0tPgo8c3ZnIHZlcnNpb249IjEuMSIgaWQ9IkxheWVyXzEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHg9IjBweCIgeT0iMHB4IgoJIHZpZXdCb3g9IjAgMCA5NDggOTQ4IiBzdHlsZT0iZW5hYmxlLWJhY2tncm91bmQ6bmV3IDAgMCA5NDggOTQ4OyIgeG1sOnNwYWNlPSJwcmVzZXJ2ZSI+CjxzdHlsZSB0eXBlPSJ0ZXh0L2NzcyI+Cgkuc3Qwe2ZpbGwtcnVsZTpldmVub2RkO2NsaXAtcnVsZTpldmVub2RkO2ZpbGw6I0ZGRkZGRjt9Cjwvc3R5bGU+CjxwYXRoIGlkPSJDb21wb3VuZF9TaGFwZSIgY2xhc3M9InN0MCIgZD0iTTQxNy45LDc5OS43TDM4Myw4MzQuNmwzNC45LDM0LjlsMCwwbDM0LjktMzQuOUw0MTcuOSw3OTkuN0w0MTcuOSw3OTkuN3ogTTg1NC4zLDI4My4xCglsLTQuMy0xNmwtMjAuNS03Ni41bC01LjctMjEuMWwtMTMuMy00OS42TDcxMyw5My44TDU4OS44LDIxN2wtMTkuNSwxOS41TDQ0Ny4xLDM1OS43bDE1LDU2LjFsNS40LDIwLjNsOTEuOCwyNC42bDU2LTU2bDkwLjktOTAuOQoJbDIuMiw4LjNsMjYuOCwxMDAuMmwtMzcuMSwzNy4xbC0xNC45LDE0LjlsLTk0LjYsOTQuN2wtNzMuNC0xOS43bC0xOC44LTVsLTgtMi4xbC03MC42LTE4LjlsLTI5LjYtNy45bC0yNi44LTEwMC4yTDMzNC43LDMxNQoJbDc5LjItNzkuMmw0LjEtNC4xTDYwNC41LDQ1LjFsLTkyLjEtMjQuN0w0MzYuMiwwbC0xOC4zLDE4LjNsLTQ5LjIsNDkuMmwtNzkuMiw3OS4ybC00Mi44LDQyLjhsLTEuMiwxLjJsLTEsMWwtMjguOCwyOC44CglsLTQ5LjUsNDkuNUwxODQsMzM2bC02NS44LDY1LjhsLTAuOCwwLjhsMC44LDAuOGwzNCwzNGw0Ni41LTQ2LjVsMy40LTMuNGwxMy42LTEzLjZsOS45LTkuOWwtNi4xLDYuMWwtMy45LDMuOWwtMTMuNSwxMy41CgljOS42LTkuNSwyNS05LjYsMzQuNy0wLjJsMC4zLDAuM2M5LjQsOS43LDkuNCwyNS4xLTAuMSwzNC43bDE3LjQtMTcuNGwtMTcuNSwxNy41bC0yMS4zLDIxLjNsLTIuMywyLjNsLTg3LjMsODcuM2wzNC45LDM0LjlsMzUtMzUKCWwxOS43LTE5LjdsMi42LTIuNmw5LjUtOS41bDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xbDAuMS0wLjFsOS4yLTkuMmM5LjYtOS42LDI1LjItOS42LDM0LjksMHM5LjYsMjUuMiwwLDM0LjlsLTAuMSwwLjFsLTAuMSwwLjEKCWwtMC4xLDAuMWwtOS4xLDlsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTkuMSw5LjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFMMjQzLDU1Ni4xbC0xMiwxMgoJbC0xNS4yLDE1LjJsLTg0LjEsODQuMWwzNC45LDM0LjlsNDkuMi00OS4ybDQxLjktNDEuOWwyNi43LTI2LjdsMCwwbDItMmwwLjEtMC4xbDAuMS0wLjFsMC4xLTAuMWM5LjYtOS42LDI1LjItOS42LDM0LjksMAoJYzguOCw4LjgsOS43LDIyLjksMS45LDMyLjdsMS44LTEuOGwwLjEsMGwwLjEtMC4xbDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xbDkuMS05LjFsMC4xLTAuMWwwLjEtMC4xbDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xCglsMC4xLTAuMWwzLjgtMy44bDAuMSwwLjFsLTcuMSw3LjFsLTkuOCw5LjhsLTAuNSwwLjVsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTksOS4xbC0wLjEsMC4xbC0wLjEsMC4xbC0wLjEsMC4xbC0wLjEsMC4xCglsLTAuMSwwLjFsLTAuMSwwLjFsLTkuMSw5LjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTYuMSw2LjFsMCwwbC0zLjgsMy44bC0zNC45LDM0LjlsMzQuOSwzNC45bDIxLjItMjEuMnYwCglsMTAuMi0xMC4ybDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xbDktOS4xbDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xbDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xbDUuMy01LjNsMy0zbDAuMS0wLjFsMCwwbDAuMS0wLjFsMC0wLjEKCWwwLjEtMC4xbDAuMS0wLjFsOS4xLTlsMC4xLTAuMWwwLjEtMC4xbDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xbDAuMS0wLjFsOS4xLTkuMWwwLjEtMC4xbDAuMS0wLjFsMC4xLTAuMQoJYzkuNi05LjcsMjUuMi05LjgsMzQuOS0wLjJjOS43LDkuNiw5LjgsMjUuMiwwLjIsMzQuOWMtMC4xLDAuMS0wLjEsMC4xLTAuMiwwLjJsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTkuMSw5LjFsLTAuMSwwLjEKCWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtOCw4bC0wLjQsMC40bC0wLjEsMC4xbC0wLjEsMC4xbDAsMGwtMC41LDAuNWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMQoJbC0wLjEsMGwtNi4xLDYuMWwtMi4yLDIuMmwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtOS4xLDkuMWwtMC4xLDAuMWwtMC4xLDAuMWwtMC4xLDAuMWwtMTAuMiwxMC4yCglMMjQxLjksODM2LjJsMzQuOSwzNC45bDE0MS4xLTE0MS4xbDMxLjUtMzEuNWw4LjItOC4ybDEwLjQtMTAuNGwwLjEtMC4xbDAuMS0wLjFsMC4xLTAuMWw5LjEtOWwwLjEtMC4xbDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xCglsMC4xLTAuMWwwLjEtMC4xbDkuMS05bDAuMS0wLjFsMC4xLTAuMWwwLjEtMC4xYzkuNy05LjYsMjUuMy05LjUsMzQuOSwwLjJjOS41LDkuNiw5LjUsMjUuMSwwLDM0LjdsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjEKCWwtOSw5LjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTAuMSwwLjFsLTksOS4xbC0wLjEsMC4xbC0wLjEsMC4xbC0wLjEsMC4xbC0xOC42LDE4LjZMNDQ4LDc2OS42bDM0LjksMzQuOQoJbDg0LjgtODQuOGw2Ni4zLDE3LjhsNjQtNjRsMCwwYzkuNC05LjIsMTkuMS0xOS4zLDI5LjYtMjkuOGwyOS40LTI5LjRsOTQuNy05NC43bDUyLTUyTDg1NC4zLDI4My4xeiBNMTk5LjksODc4LjJMMTY1LjEsOTEzCglsMzQuOSwzNC45bDE1LjgtMTUuOGwxOS0xOWwtMTktMTlMMTk5LjksODc4LjJ6IE0yMTUuOCw3OTIuNmwzNC45LTM0LjlsLTM0LjktMzQuOWwwLDBsLTM0LjksMzQuOUwyMTUuOCw3OTIuNkwyMTUuOCw3OTIuNnoKCSBNMTE1LjksNDczLjlMODEsNDM5bC0zNC45LDM0LjlMODEsNTA4LjhMMTE1LjksNDczLjl6IE00Myw3NTYuMUw3Ny45LDc5MWwzNC45LTM0LjlsLTM0LjktMzQuOUw0Myw3NTYuMXogTTg0LjIsNTc1LjRsLTM0LjksMzQuOQoJbDM0LjksMzQuOWwzNC0zNGwwLjgtMC44bC0wLjgtMC44TDg0LjIsNTc1LjR6Ii8+Cjwvc3ZnPgo=" /> -->
			<!-- <thanksCard title="Oasis Hosting" website="https://oasis-hosting.net/" :src="oasis_logo" /> -->
		</div>
		<span class="copyright"> &copy; {{ new Date().getFullYear() }} Squad Whitelister, V{{ version }}, JetDave | Fantino Davide </span>
	</footer>
</template>

<style>
@import './assets/base.css';

#app * {
	--accent-color: v-bind(accent_color);
}

main {
	display: flex;
	align-items: center;
	justify-content: center;
	flex-direction: column;
	align-items: center;
	/* padding: 20px; */
	display: flex;
	flex-direction: column;
	align-items: stretch;
	justify-content: flex-start;
	flex-grow: 1;
}

header .logo {
	max-height: 50px;
	margin-right: 10px;
}

#hdBtnContainer {
	display: flex;
	flex-grow: 100;
	justify-content: end;
	align-items: center;
}

#tabBrowser>button {
	white-space: nowrap;
}

.popupsArea {
	align-self: center;
	display: flex;
	flex-direction: column;
}

.addNewClan,
.clanCard {
	width: 250px;
	height: 250px;
	border: 5px solid #fff2;
	border-radius: 15px;
	margin: 10px;
	background: #0000;
	display: flex;
	flex-direction: column;
	overflow: hidden;
	min-width: 200px;
}

.addNewClan::after,
.addNewClan::before {
	content: '';
	position: absolute;
	height: 75px;
	width: 5px;
	border-radius: 1px;
	background: #444;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%) rotate(90deg);
}

.addNewClan::before {
	transform: translate(-50%, -50%) rotate(0deg);
}

.popupContainer {
	position: fixed;
	top: 0;
	left: 0;
	display: flex;
	width: 100%;
	height: 100%;
	justify-content: center;
	align-items: baseline;
	padding-top: 100px;
}

button.addHorizontal,
.addNewGameGroup,
.gameGroupCard {
	/* width: ; */
	height: 65px;
	border: 5px solid #fff2;
	border-radius: 15px;
	margin: 10px;
	background: #0000;
	display: flex;
	flex-direction: row;
	overflow: hidden;
	align-items: center;
}

button.addHorizontal::after,
button.addHorizontal::before,
.addNewGameGroup::after,
.addNewGameGroup::before {
	content: '';
	position: absolute;
	height: 35px;
	width: 5px;
	border-radius: 1px;
	background: #444;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%) rotate(90deg);
}

button.addHorizontal::before,
.addNewGameGroup::before {
	transform: translate(-50%, -50%) rotate(0deg);
}

.gameGroupCard:hover .hoverMenu,
.hoverMenu.vis {
	bottom: calc(-100% + 25px);
	/* animation: scrollTopDown 200ms ease-in-out alternate infinite; */
}

body.mobile .gameGroupCard:hover .hoverMenu,
body.mobile .hoverMenu.vis {
	bottom: 0px !important;
	/* animation: scrollTopDown 200ms ease-in-out alternate infinite; */
}

.hoverMenu:hover {
	bottom: 0px !important;
}

@keyframes scrollTopDown {
	0% {
		bottom: calc(-100% + 18px);
	}

	100% {
		bottom: calc(-100% + 23px);
	}
}
</style>
