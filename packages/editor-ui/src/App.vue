<template>
	<div :class="[$style.app, 'root-container']">
		<LoadingView v-if="loading" />
		<div
			v-else
			id="app"
			:class="{
				[$style.container]: true,
				[$style.sidebarCollapsed]: uiStore.sidebarMenuCollapsed,
			}"
		>
			<div id="banners" :class="$style.banners">
				<BannerStack v-if="!isDemoMode" />
			</div>
			<div id="header" :class="$style.header">
				<router-view name="header"></router-view>
			</div>
			<div v-if="usersStore.currentUser" id="sidebar" :class="$style.sidebar">
				<router-view name="sidebar"></router-view>
			</div>
			<div id="content" :class="$style.content">
				<router-view v-slot="{ Component }">
					<keep-alive v-if="$route.meta.keepWorkflowAlive" include="NodeViewSwitcher" :max="1">
						<component :is="Component" />
					</keep-alive>
					<component :is="Component" v-else />
				</router-view>
			</div>
			<AskAssistantChat />
			<Modals />
			<Telemetry />
			<AskAssistantFloatingButton v-if="showAssistantButton" />
		</div>
	</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { mapStores } from 'pinia';

import BannerStack from '@/components/banners/BannerStack.vue';
import Modals from '@/components/Modals.vue';
import LoadingView from '@/views/LoadingView.vue';
import Telemetry from '@/components/Telemetry.vue';
import AskAssistantChat from '@/components/AskAssistant/AskAssistantChat.vue';
import AskAssistantFloatingButton from '@/components/AskAssistant/AskAssistantFloatingButton.vue';
import { HIRING_BANNER, VIEWS } from '@/constants';

import { loadLanguage } from '@/plugins/i18n';
import { useGlobalLinkActions } from '@/composables/useGlobalLinkActions';
import { useExternalHooks } from '@/composables/useExternalHooks';
import { useToast } from '@/composables/useToast';
import { useCloudPlanStore } from '@/stores/cloudPlan.store';
import { useNodeTypesStore } from '@/stores/nodeTypes.store';
import { useRootStore } from '@/stores/root.store';
import { useSettingsStore } from '@/stores/settings.store';
import { useSourceControlStore } from '@/stores/sourceControl.store';
import { useTemplatesStore } from '@/stores/templates.store';
import { useUIStore } from '@/stores/ui.store';
import { useUsageStore } from '@/stores/usage.store';
import { useUsersStore } from '@/stores/users.store';
import { useHistoryHelper } from '@/composables/useHistoryHelper';
import { useRoute } from 'vue-router';
import { useAssistantStore } from './stores/assistant.store';

export default defineComponent({
	name: 'App',
	components: {
		BannerStack,
		LoadingView,
		Telemetry,
		Modals,
		AskAssistantChat,
		AskAssistantFloatingButton,
	},
	setup() {
		return {
			...useGlobalLinkActions(),
			...useHistoryHelper(useRoute()),
			...useToast(),
			externalHooks: useExternalHooks(),
		};
	},
	computed: {
		...mapStores(
			useAssistantStore,
			useNodeTypesStore,
			useRootStore,
			useSettingsStore,
			useTemplatesStore,
			useUIStore,
			useUsersStore,
			useSourceControlStore,
			useCloudPlanStore,
			useUsageStore,
		),
		defaultLocale(): string {
			return this.rootStore.defaultLocale;
		},
		isDemoMode(): boolean {
			return this.$route.name === VIEWS.DEMO;
		},
		showAssistantButton(): boolean {
			return this.assistantStore.canShowAssistantButtons;
		},
	},
	data() {
		return {
			loading: true,
		};
	},
	watch: {
		defaultLocale(newLocale) {
			void loadLanguage(newLocale);
		},
	},
	async mounted() {
		this.logHiringBanner();
		void useExternalHooks().run('app.mount');
		this.loading = false;
	},
	methods: {
		logHiringBanner() {
			if (this.settingsStore.isHiringBannerEnabled && !this.isDemoMode) {
				console.log(HIRING_BANNER);
			}
		},
	},
});
</script>

<style lang="scss" module>
.app {
	height: 100vh;
	overflow: hidden;
}

.container {
	display: grid;
	grid-template-areas:
		'banners banners rightsidebar'
		'sidebar header rightsidebar'
		'sidebar content rightsidebar';
	grid-auto-columns: minmax(0, max-content) minmax(100px, auto) minmax(0, max-content);
	grid-template-rows: auto fit-content($header-height) 1fr;
	height: 100vh;
}

.banners {
	grid-area: banners;
	z-index: 999;
}

.content {
	display: flex;
	grid-area: content;
	position: relative;
	overflow: auto;
	height: 100%;
	width: 100%;
	justify-content: center;

	main {
		width: 100%;
		height: 100%;
	}
}

.header {
	grid-area: header;
	z-index: 99;
}

.sidebar {
	grid-area: sidebar;
	height: 100%;
	z-index: 999;
}
</style>
