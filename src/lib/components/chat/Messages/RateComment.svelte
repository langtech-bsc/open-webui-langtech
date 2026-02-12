<script lang="ts">
	import { toast } from 'svelte-sonner';

	import { createEventDispatcher, onMount, getContext } from 'svelte';
	import { config, models, tags as _tags } from '$lib/stores';
	import Tags from '$lib/components/common/Tags.svelte';
	import XMark from '$lib/components/icons/XMark.svelte';

	const i18n = getContext('i18n');

	const dispatch = createEventDispatcher();

	export let message;

	export let show = false;

	let PROMPT_REASONS = ['PROMPT-FLAWED', 'SUBJECT-CHANGED', 'LANGUAGE-AMBIGUOUS'];
	let LIKE_REASONS = [];
	let DISLIKE_REASONS = [
		'HALLUCINATION',
		'OUTDATED-INFO',
		'OVERGENERALIZATION',
		'LOGIC',
		'MATH',
		'PARTIAL-REASONING',
		'CONTRADICTION',
		'SEQUENCE-REPETITION',
		'TURN-REPETITION',
		'WORD-SALAD',
		'VA-LLENO',
		'BRIEFTY',
		'UNCLEAR-REFERENCE',
		'TONE-SHIFT',
		'LANGUAGE-SHIFT',
		'LANGUAGE-ERROR',
		'UNNECESSARY-TAGS',
		'WEIRD-SEQUENCE',
		'SOLILOQUY',
		'IGNORED-CONSTRAINT',
		'OVER-REFUSAL',
		'UNDER-REFUSAL',
		'PARTIAL-ANSWER',
		'PROMPT-IGNORED',
		'CODE',
		'TRANSLATION',
		'SUMMARIZATION',
		'FORMAT',
		'OVERCONFIDENCE',
		'CONTEXT-INCONSISTENCY',
		'CONTEXT-IGNORED',
		'NO-DISCLAIMING',
		'DIALECT',
		'CULTURE',
		'IDENTITY',
		'OVERREACHER',
		'OTHER'
	];

	let PROMPT_LANGUAGES = ['ES', 'CA', 'EN', 'EU', 'GL'];

	let tags = [];

	let reasons = [];
	let promptReasons = [];
	export let selectedPromptReasons = null;
	export let selectedReasons = null;
	export let selectedLangs = null;
	let langs = [];
	let comment = '';
	let warn = 0;
	let rating = null;
	let selectedModel = null;

	$: if (message?.annotation?.rating === 1) {
		reasons = LIKE_REASONS;
		rating = 1
	} else if (message?.annotation?.rating === -1) {
		reasons = DISLIKE_REASONS;
		rating = -1
	}
	langs = PROMPT_LANGUAGES;
	promptReasons = PROMPT_REASONS;

	$: if (message) {
		init();
	}

	const init = () => {
		if (!selectedPromptReasons) {
			selectedPromptReasons = message?.annotation?.promptReasons?.split(',').filter(Boolean) ?? [];
		}

		if (!selectedReasons) {
			selectedReasons = message?.annotation?.reasons?.split(',').filter(Boolean) ?? [];
		}

		if (!selectedLangs) {
			selectedLangs = message?.annotation?.langs?.split(',').filter(Boolean) ?? [];
		}

		if (!comment) {
			comment = message?.annotation?.comment ?? '';
		}

		tags = (message?.annotation?.tags ?? []).map((tag) => ({
			name: tag
		}));

	};

	onMount(() => {
		if (message?.arena) {
			selectedModel = $models.find((m) => m.id === message.selectedModelId);
		}
	});

	const saveHandler = () => {
		if (selectedLangs.length < 1 || ( (rating === -1) && (selectedReasons.length < 1))) {
			toast.error($i18n.t(selectedLangs.length < 1  ? 'Please select a Language' : 'Please select a reason'));
			warn = 1
		} else {
			warn = 0
		}

		selectedReasons = selectedReasons.filter(function(item){return reasons.indexOf(item) > -1;})

		dispatch('save', {
			// remove reasons inconsistent with the current thumb up/down
			reasons: selectedReasons.toString(),
			promptReasons: selectedPromptReasons.toString(),
			langs: selectedLangs.toString(),
			comment: comment,
			tags: tags.map((tag) => tag.name),
			warn: warn
		});

		if (warn === 0) {
			toast.success($i18n.t('Thanks for your feedback!'));
		}
		show = false;
	};
</script>

<div
	class=" my-2.5 rounded-xl px-4 py-3 border border-gray-100/30 dark:border-gray-850/30"
	id="message-feedback-{message.id}"
>

	<div>
		<div class="text-sm mt-1.5 font-medium">Languages</div>
		<div class="flex flex-wrap gap-1.5 text-sm mt-1.5">
			{#each langs as lang}
				<button
					class="px-3 py-0.5 border border-gray-100/30 dark:border-gray-850/30 hover:bg-gray-50 dark:hover:bg-gray-850 {selectedLangs.includes(
						lang
					)
						? 'bg-gray-100 dark:bg-gray-800'
						: ''} transition rounded-xl"
					on:click={() => {
						if (selectedLangs.includes(lang)) {
							selectedLangs.splice(selectedLangs.indexOf(lang), 1);
						} else {
							selectedLangs.push(lang);
						}
						// @jab make change so it is rective
						selectedLangs = selectedLangs;
					}}
				>
					{lang}
				</button>
			{/each}
		</div>
		<div class="text-sm mt-1.5 font-medium">Prompt Reasons</div>
		<div class="flex flex-wrap gap-1.5 text-sm mt-1.5">
			{#each promptReasons as promptReason}
				<button
					class="px-3 py-0.5 border border-gray-100/30 dark:border-gray-850/30 hover:bg-gray-50 dark:hover:bg-gray-850 {selectedPromptReasons.includes(
						promptReason
					)
						? 'bg-gray-100 dark:bg-gray-800'
						: ''} transition rounded-xl"
					on:click={() => {
						if (selectedPromptReasons.includes(promptReason)) {
							selectedPromptReasons.splice(selectedPromptReasons.indexOf(promptReason), 1);
						} else {
							selectedPromptReasons.push(promptReason);
						}
						// @jab make change so it is rective
						selectedPromptReasons = selectedPromptReasons;
					}}
				>
					{promptReason}
				</button>
			{/each}
		</div>
		{#if reasons.length > 0}
			<div class="text-sm mt-1.5 font-medium">{$i18n.t('Why?')}</div>

			<div class="flex flex-wrap gap-1.5 text-sm mt-1.5">
				{#each reasons as reason}
					<button
						class="px-3 py-0.5 border border-gray-100/30 dark:border-gray-850/30 hover:bg-gray-50 dark:hover:bg-gray-850 {selectedReasons.includes(
							reason
						)
							? 'bg-gray-100 dark:bg-gray-800'
							: ''} transition rounded-xl"
						on:click={() => {
							if (selectedReasons.includes(reason)) {
								selectedReasons.splice(selectedReasons.indexOf(reason), 1);
							} else {
								selectedReasons.push(reason);
							}
							// @jab make change so it is rective
							selectedReasons = selectedReasons;
						}}
					>
						{reason}
					</button>
				{/each}
			</div>
		{/if}
	</div>

	<div class="mt-2">
		<textarea
			bind:value={comment}
			class="w-full text-sm px-1 py-2 bg-transparent outline-hidden resize-none rounded-xl"
			placeholder={$i18n.t('Feel free to add specific details')}
			rows="3"
		/>
	</div>

	<div class="mt-2 gap-1.5 flex justify-between">
		<!-- @jab begin suppress tags -->
		<div class="flex items-end group"></div>
		<!-- @jab end suppress tags -->
		<button
			class="px-3.5 py-1.5 text-sm font-medium bg-black hover:bg-gray-900 text-white dark:bg-white dark:text-black dark:hover:bg-gray-100 transition rounded-full"
			on:click={() => {
				saveHandler();
			}}
		>
			{$i18n.t('Save')}
		</button>
	</div>
</div>
