<script lang="ts">
	import { getContext } from 'svelte'
	import { initConfig, initOutput } from '../../editor/appUtils'
	import type { AppViewerContext, ComponentCustomCSS, RichConfigurations } from '../../types'
	import { initCss } from '../../utils'
	import { components } from '../../editor/component'
	import ResolveConfig from '../helpers/ResolveConfig.svelte'
	import ResolveStyle from '../helpers/ResolveStyle.svelte'
	import { ArrowDown } from 'lucide-svelte'
	import { twMerge } from 'tailwind-merge'
	import { loadIcon } from '../icon'

	export let id: string
	export let configuration: RichConfigurations
	export let customCss: ComponentCustomCSS<'statcomponent'> | undefined = undefined
	export let render: boolean

	const { app, worldStore } = getContext<AppViewerContext>('AppViewerContext')

	let resolvedConfig = initConfig(
		components['statcomponent'].initialData.configuration,
		configuration
	)

	initOutput($worldStore, id, {})

	let css = initCss($app.css?.statcomponent, customCss)

	let iconComponent: any

	$: resolvedConfig.icon && handleIcon()

	async function handleIcon() {
		if (resolvedConfig.icon) {
			iconComponent = await loadIcon(resolvedConfig.icon)
		}
	}
</script>

{#each Object.keys(components['statcomponent'].initialData.configuration) as key (key)}
	<ResolveConfig
		{id}
		{key}
		bind:resolvedConfig={resolvedConfig[key]}
		configuration={configuration[key]}
	/>
{/each}

{#each Object.keys(css ?? {}) as key (key)}
	<ResolveStyle
		{id}
		{customCss}
		{key}
		bind:css={css[key]}
		componentStyle={$app.css?.imagecomponent}
	/>
{/each}

{#if render}
	<div class="p-6 border rounded-md h-full flex flex-row gap-6 w-full items-center">
		{#if resolvedConfig.icon && iconComponent}
			<div class="bg-blue-500 rounded-md p-2">
				<svelte:component this={iconComponent} size={24} color="white" />
			</div>
		{/if}
		<div class="w-full">
			<div class="text-base font-normal text-primary">{resolvedConfig.title}</div>
			<div class="mt-1 flex items-baseline justify-between">
				<div class="flex items-baseline text-2xl font-semibold text-blue-600 dark:text-blue-200">
					{resolvedConfig?.value}
				</div>

				{#if resolvedConfig?.progress !== undefined && resolvedConfig?.progress !== null && resolvedConfig?.progress !== 0}
					<div
						class={twMerge(
							'flex items-center flex-row gap-2 rounded-full px-2.5 py-0.5 text-sm font-medium',
							resolvedConfig?.progress > 0
								? 'bg-green-100 text-green-800'
								: 'bg-red-100 text-red-800'
						)}
					>
						<ArrowDown
							size={16}
							class={resolvedConfig?.progress > 0 ? 'transform rotate-180' : 'transform rotate-0'}
						/>
						{resolvedConfig?.progress}%
					</div>
				{/if}
			</div>
		</div>
	</div>
{/if}
