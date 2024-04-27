<script>
	import { goto } from '$app/navigation';

	let games = [
		{
			name: 'Four in a row',
			description: 'AKA: Connect 4.',
			href: '/four'
		},
		{
			name: 'Checkers',
			href: '/checkers',
			description: 'Like chess, but easier to code. (BROKEN RN)'
		}
	];
</script>

<div class="prose mx-auto dark:prose-invert">
	<h1 class="scroll-m-20 pt-5 text-center text-5xl font-extrabold tracking-tight lg:text-6xl">
		Games!
	</h1>

	{#each games as game}
		<button class="w-full" on:click={() => goto(game.href)}>
			<div class="rounded-2xl border p-5 hover:bg-zinc-500/5">
				<strong class="text-xl">{game.name}</strong>
				<br />
				{game.description}
			</div>
		</button>
		<br /><br />
	{/each}
</div>
