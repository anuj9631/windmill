<script lang="ts">
	import { createEventDispatcher, setContext } from 'svelte'
	import type { ListInputs, ListContext } from '../../types'
	import { writable } from 'svelte/store'

	export let index: number
	export let value: any
	export let disabled = false

	const ctx = writable({ index, value, disabled })

	$: $ctx = { index, value, disabled }

	const dispatch = createEventDispatcher()
	setContext<ListContext>('RowWrapperContext', ctx)
	setContext<ListInputs>('RowInputs', {
		set: (id: string, value: any) => {
			dispatch('set', { id, value })
		},
		remove(id) {
			dispatch('remove', id)
		}
	})
</script>

<slot />
