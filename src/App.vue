<script setup lang="ts">
import type { AccentName, FlavorName } from '@catppuccin/palette';

import rawUserstylesImport from '../import.json' with { type: 'json' };

import { get, set } from '@vueuse/core';
import { flavorEntries, flavors } from '@catppuccin/palette';

const accentEntries = flavors.latte.colorEntries.filter(
	([_, { accent }]) => accent,
);

const isDark = usePreferredDark();

const mode = useColorMode<FlavorName>({
	attribute: 'theme',
	modes: Object.fromEntries(
		Object.keys(flavors).map((flavor) => [flavor, flavor]),
	),
	initialValue: isDark ? 'mocha' : 'latte',
});
const { state, next } = useCycleList<FlavorName>(
	Object.keys(flavors) as FlavorName[],
	{
		initialValue: mode as Ref<FlavorName>,
	},
);
watchEffect(() => set(mode, get(state)));

const darkFlavor = useStorage<FlavorName>('darkFlavor', 'mocha');
const lightFlavor = useStorage<FlavorName>('lightFlavor', 'latte');
const accentColor = useStorage<AccentName>('accentColor', 'mauve');

const userstyles = rawUserstylesImport.slice(1) as UserstyleEntry[];

function createCustomUserstylesImport() {
	const customSelectedUserstyles = userstyles
		.filter(({ name }) => get(selectedUserstyles)[name])
		.map((userstyle) => {
			const updatedVars = {
				...userstyle.usercssData.vars,
				accentColor: { value: get(accentColor) },
				darkFlavor: { value: get(darkFlavor) },
				lightFlavor: { value: get(lightFlavor) },
			};

			return {
				...userstyle,
				usercssData: {
					...userstyle.usercssData,
					vars: updatedVars,
				},
			};
		});

	return JSON.stringify([
		rawUserstylesImport[0],
		...customSelectedUserstyles,
	]);
}

const searchText = ref('');
const selectedUserstyles = useStorage<Record<string, boolean>>(
	'selectedUserstyles',
	getAllSelected(),
);
function getAllSelected() {
	return userstyles.reduce((acc, { name }) => ({ ...acc, [name]: true }), {});
}
function getFilteredUserstyles() {
	return userstyles.filter(({ name }) =>
		name.toLowerCase().includes(searchText.value.toLowerCase()),
	);
}

const downloaded = ref(false);

function download() {
	const blob = new Blob([createCustomUserstylesImport()], {
		type: 'application/json',
	});
	const href = URL.createObjectURL(blob);
	const link = document.createElement('a');
	link.href = href;
	link.download = 'import.json';
	link.click();
	URL.revokeObjectURL(href);
	set(downloaded, true);
	setTimeout(() => {
		set(downloaded, false);
	}, 1000);
}
</script>

<template>
	<div class="h-100vh w-full flex flex-col px4 pt4 text-lg gap-24">
		<header class="flex justify-between">
			<h1 class="text-3xl">Catppuccin Userstyles Customizer</h1>
			<div class="flex flex-row gap-2 h-[min-content]">
				<a
					class="border border-surface0 border-rounded flex h-auto p2 hover:bg-mantle"
					aria-label="GitHub repository"
					href="https://github.com/uncenter/catppuccin-userstyles-customizer"
					target="_blank"
				>
					<div class="self-center i-carbon-logo-github" />
				</a>
				<button
					class="border border-surface0 border-rounded flex p2 hover:bg-mantle"
					:aria-label="`Toggle theme to ${flavors[state].name}`"
					@click="next()"
				>
					<span class="capitalize">{{ flavors[state].name }}</span>
				</button>
			</div>
		</header>
		<main class="flex flex-col gap-8 md:gap-4">
			<div
				class="flex flex-wrap justify-center md:flex-row gap-4 sm:gap-6 self-center mx-auto"
			>
				<div class="flex flex-col gap-2">
					<label for="lightFlavor">Light Flavor</label>
					<select
						id="lightFlavor"
						v-model="lightFlavor"
						class="border border-surface0 border-rounded bg-base p2"
						name="lightFlavor"
					>
						<option
							v-for="[flavor, { name }] in flavorEntries"
							:value="flavor"
						>
							{{ name }}
						</option>
					</select>
				</div>
				<div class="flex flex-col gap-2">
					<label for="darkFlavor">Dark Flavor</label>
					<select
						id="darkFlavor"
						v-model="darkFlavor"
						class="border border-surface0 border-rounded bg-base p2"
						name="darkFlavor"
					>
						<option
							v-for="[flavor, { name }] in flavorEntries"
							:value="flavor"
						>
							{{ name }}
						</option>
					</select>
				</div>
				<div class="flex flex-col gap-2">
					<label for="accentColor">Accent Color</label>
					<select
						id="accentColor"
						v-model="accentColor"
						class="border border-surface0 border-rounded bg-base p2"
						name="accentColor"
					>
						<option
							v-for="[color, { name }] in accentEntries"
							:value="color"
						>
							{{ name }}
						</option>
					</select>
				</div>
			</div>
			<button
				class="border-rounded mx-auto flex flex-row gap-2 bg-mauve text-base py-2 px-4 justify-center items-center hover:scale-105 transition-transform group"
				:style="{
					backgroundColor: `var(--ctp-${get(accentColor)})`,
				}"
				title="Download"
				@click="download()"
			>
				<span v-text="downloaded ? 'Downloaded!' : 'Download'" />
				<i v-if="downloaded" class="i-carbon-checkmark" />
				<i v-else class="i-carbon-download group-hover:animate-tada" />
			</button>
			<div class="mx-auto flex flex-col gap-2 w-full lg:w-3/4">
				<div class="flex flex-col sm:flex-row gap-2 self-center my-4">
					<input
						type="text"
						placeholder="Search"
						v-model="searchText"
						class="border border-surface0 border-rounded bg-base p2"
					/>
					<div class="flex flex-row gap-2">
						<button
							class="border-rounded flex p2 bg-surface0 hover:bg-surface1"
							@click="selectedUserstyles = {}"
						>
							Deselect All
						</button>
						<button
							class="border-rounded flex p2 bg-surface0 hover:bg-surface1"
							@click="selectedUserstyles = getAllSelected()"
						>
							Select All
						</button>
					</div>
				</div>
				<div
					class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4"
				>
					<template
						v-for="userstyle in userstyles"
						:key="userstyle.name"
					>
						<label
							v-if="getFilteredUserstyles().includes(userstyle)"
							:for="userstyle.name"
							class="flex flex-row gap-2 border border-surface0 border-rounded p2 items-center justify-between"
						>
							<h2 class="text-xl">
								{{ userstyle.name.replace(' Catppuccin', ' ') }}
							</h2>
							<input
								type="checkbox"
								v-model="selectedUserstyles[userstyle.name]"
								class="size-5"
								:name="userstyle.name"
								:id="userstyle.name"
							/>
						</label>
					</template>
				</div>
			</div>
		</main>
	</div>
</template>

<style></style>
