<!-- Tile.svelte -->
<script lang="ts">
	import { Crown } from 'lucide-svelte';
	import { isBlue, isRed, isKing } from './+page.svelte';

	export let piece;
</script>

<div class="aspect-square">
	{#if isRed(piece)}
		<div class="h-full w-full rounded-full bg-red-500">
			<div class="h-full w-full scale-75 rounded-full bg-red-600/60">
				{#if isKing(piece)}
					<Crown class="h-full w-full p-[1vw] text-background/20" />
				{/if}
			</div>
		</div>
	{:else if isBlue(piece)}
		<div class="h-full w-full rounded-full bg-cyan-600">
			<div class="h-full w-full scale-75 rounded-full bg-cyan-700/60">
				{#if isKing(piece)}
					<Crown class="h-full w-full p-[1vw] text-background/20" />
				{/if}
			</div>
		</div>
	{:else if piece == 'move-preview'}
		<button transition:scale class="h-full w-full scale-50 rounded-full bg-zinc-500/50" />
	{/if}
</div>
