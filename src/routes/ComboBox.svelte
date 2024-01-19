<script context="module" lang="ts">
	import { createCombobox, melt } from '@melt-ui/svelte';
	import type { FloatingConfig } from '@melt-ui/svelte/internal/actions';
	import { debounce } from 'lodash-es';
	import { createEventDispatcher, onDestroy, onMount } from 'svelte';
	import { type Writable, writable } from 'svelte/store';
</script>

<script lang="ts">
	export let positioning: FloatingConfig = {
		placement: 'top-start',
		fitViewport: true,
		sameWidth: true
	};

	type T = $$Generic<{ id: string; name: string }>;

	export let defaultValue: T | undefined = undefined;
	export let name: string;
	export let placeholder: string = '';
	
	const items = writable<Array<T>>([]);

	interface $$Events {
		selected: CustomEvent<{
			selected: T;
			store: ReturnType<typeof createCombobox<T>>["states"]["selected"]
			inputValue: ReturnType<typeof createCombobox<T>>["states"]["inputValue"]
		}>;
		change: Event;
		'before-options-update': CustomEvent<{
			store: typeof items;
			value: string;
		}>;
	}

	const dispatch = createEventDispatcher<{
		selected: {
			selected: T;
			store: ReturnType<typeof createCombobox<T>>["states"]["selected"]
			inputValue: ReturnType<typeof createCombobox<T>>["states"]["inputValue"]
		};
		'before-options-update': {
			store: typeof items;
			value: string;
		};
	}>();

	const {
		elements: { menu, input, option, label },
		states: { open, inputValue, selected, touchedInput },
		helpers: { isSelected }
	} = createCombobox<T>({
		forceVisible: true,
		multiple: false,
		preventScroll: true,
		positioning,
		onSelectedChange({ curr, next }) {
			if (next) {
				dispatch('selected', {
					selected: next.value,
					store: selected,
					inputValue
				});
			}
			return next;
		}
	});

	onMount(() => {
		if (typeof defaultValue !== 'undefined') {
			selected.set({
				value: defaultValue,
				label: defaultValue.name
			});
		}
	});

	$: $inputValue = $selected?.label ?? '';
</script>

<div>
	{#if $$slots.label}
		<label use:melt={$label}>
			<slot name="label" />
		</label>
	{/if}

	<input
		{name}
		autocomplete="off"
		{placeholder}
		type="text"
		on:input={debounce((e) => {
			const elem = e.target;
			if (!(elem instanceof HTMLInputElement)) return;

			dispatch('before-options-update', {
				store: items,
				value: elem.value
			});
		}, 250)}
		on:change
		use:melt={$input}
	/>
</div>

{#if $open}
	{#if $items.length > 0}
		<ul class="menu" use:melt={$menu}>
			{#each $items as item, index (index)}
				<li
					class="combobox-item"
					use:melt={$option({
						value: item,
						label: item.name,
						disabled: false
					})}
				>
					<span>
						{item.name}
					</span>

					<span class="tick" class:viz-hidden={!($isSelected(item) || $inputValue === item.name)}>
						X
					</span>
				</li>
			{/each}
		</ul>
	{/if}
{/if}