<script lang="ts">
	import { Button } from '../common'

	import { getNonStreamingCompletion } from './lib'
	import { sendUserToast } from '$lib/toast'
	import Popup from '../common/popup/Popup.svelte'
	import { copilotInfo } from '$lib/stores'

	import { WindmillIcon } from '../icons'

	import { autoPlacement } from '@floating-ui/core'
	import { ExternalLink, HistoryIcon, Wand2 } from 'lucide-svelte'
	import { fade } from 'svelte/transition'
	import { createEventDispatcher } from 'svelte'

	// state
	let funcDesc: string = ''
	let genLoading: boolean = false
	let input: HTMLInputElement | undefined
	let abortController: AbortController | undefined = undefined

	let button: HTMLButtonElement | undefined

	const dispatch = createEventDispatcher()
	async function onGenerate(closePopup: () => void) {
		if (funcDesc.length <= 0) {
			return
		}
		savePrompt()
		try {
			genLoading = true
			abortController = new AbortController()
			const res = await getNonStreamingCompletion(
				[
					{
						role: 'system',
						content:
							'Generate a regex pattern that one can use in a javascript Regex object. Output only the regex itself. The regex should match the following:'
					},
					{
						role: 'user',
						content: funcDesc
					}
				],
				abortController
			)
			dispatch('gen', { res: res, prompt: funcDesc })
			closePopup()
			funcDesc = ''
		} catch (err) {
			if (err?.message) {
				sendUserToast('Failed to generate regex: ' + err.message, true)
			} else {
				sendUserToast('Failed to generate regex', true)
				console.error(err)
			}
		} finally {
			genLoading = false
		}
	}

	$: input && setTimeout(() => input?.focus(), 100)

	let promptHistory: string[] = JSON.parse(localStorage.getItem('prompts-regex') || '[]')

	function savePrompt() {
		if (promptHistory.includes(funcDesc)) {
			return
		}
		promptHistory.unshift(funcDesc)
		while (promptHistory.length > 5) {
			promptHistory.pop()
		}
		localStorage.setItem('prompts-regex', JSON.stringify(promptHistory))
	}

	function clearPromptHistory() {
		promptHistory = []
		localStorage.setItem('prompts-regex', JSON.stringify(promptHistory))
	}
</script>

{#if genLoading}
	<div transition:fade class="fixed z-[4999] inset-0 bg-gray-500/75" />
{/if}

<Popup
	floatingConfig={{
		middleware: [
			autoPlacement({
				allowedPlacements: ['bottom-start', 'bottom-end', 'top-start', 'top-end', 'top', 'bottom']
			})
		]
	}}
	let:close
>
	<svelte:fragment slot="button">
		<Button
			title="Generate regexes from prompt"
			btnClasses={'!font-medium ' + (genLoading ? 'z-[5000]' : '')}
			size="xs"
			color={genLoading ? 'red' : 'light'}
			spacingSize="md"
			startIcon={genLoading ? undefined : { icon: Wand2 }}
			propagateEvent
			on:click={genLoading ? () => abortController?.abort() : () => {}}
			bind:element={button}
		>
			{#if genLoading}
				<WindmillIcon
					white={true}
					class="mr-1 text-white"
					height="16px"
					width="20px"
					spin="veryfast"
				/>
				Stop
			{:else}
				AI
			{/if}
		</Button>
	</svelte:fragment>
	<div class="block text-primary z-[2000]">
		{#if $copilotInfo.exists_openai_resource_path}
			<div class="flex flex-col gap-4">
				<div class="flex w-96">
					<input
						type="text"
						bind:this={input}
						bind:value={funcDesc}
						on:keypress={({ key }) => {
							if (key === 'Enter' && funcDesc.length > 0) {
								onGenerate(() => close(input || null))
							}
						}}
						placeholder={'Describe what the regex should do'}
					/>
					<Button
						size="xs"
						color="blue"
						buttonType="button"
						btnClasses="!p-1 !w-[38px] !ml-2"
						aria-label="Generate"
						on:click={() => {
							onGenerate(() => close(input || null))
						}}
						disabled={funcDesc.length <= 0}
						iconOnly
						startIcon={{ icon: Wand2 }}
					/>
				</div>
				{#if promptHistory.length > 0}
					<div class="w-96 flex flex-col gap-1">
						{#each promptHistory as p}
							<Button
								size="xs2"
								color="light"
								btnClasses="justify-start overflow-x-scroll no-scrollbar"
								startIcon={{ icon: HistoryIcon, classes: 'shrink-0' }}
								on:click={() => {
									funcDesc = p
								}}>{p}</Button
							>
						{/each}
						<button
							class="underline text-xs text-start px-2 text-secondary font-normal"
							on:click={clearPromptHistory}>clear history</button
						>
					</div>
				{/if}
			</div>
		{:else}
			<p class="text-sm">
				Enable Windmill AI in the <a
					href="/workspace_settings?tab=openai"
					target="_blank"
					class="inline-flex flex-row items-center gap-1"
				>
					workspace settings <ExternalLink size={16} />
				</a>
			</p>
		{/if}
	</div>
</Popup>
