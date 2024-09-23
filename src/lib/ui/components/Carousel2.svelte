<script lang="ts">
	import { onMount } from 'svelte';

	const items = [
		{ label: 'Something' },
		// { label: 'Else' },
		// { label: 'Entirely' }
		// { label: 'Different' },
		// { label: 'Unique' },
		// { label: 'Special' },
		// { label: 'Extraordinary' },
		// { label: 'Remarkable' }
	];

	let containerHeight = 500;
	let itemHeight = 100;
	let visibleItems = Math.ceil(containerHeight / itemHeight) + 4; // Extra buffer
	let scrollPosition = 0;
	let containerRef: HTMLElement;

	$: startIndex = Math.floor(scrollPosition / itemHeight);
	$: visibleItemsArray = Array(visibleItems)
		.fill(null)
		.map((_, i) => {
			const index = (((startIndex + i) % items.length) + items.length) % items.length;
			return { ...items[index], y: (startIndex + i) * itemHeight };
		});

	function handleScroll(event: Event) {
		const target = event.target as HTMLElement;
		const newScrollPosition = target.scrollTop;

		// Check if we've scrolled more than one item height in either direction
		if (Math.abs(newScrollPosition - scrollPosition) > itemHeight) {
			// Adjust scroll position to create illusion of infinite scroll
			const adjustment = Math.sign(newScrollPosition - scrollPosition) * itemHeight * items.length;
			target.scrollTop = newScrollPosition - adjustment;
			scrollPosition = target.scrollTop;
		} else {
			scrollPosition = newScrollPosition;
		}
	}

	onMount(() => {
		containerRef.addEventListener('scroll', handleScroll);
		// Initialize scroll position to middle of the virtual list
		containerRef.scrollTop = items.length * itemHeight * 500;
		scrollPosition = containerRef.scrollTop;
		return () => containerRef.removeEventListener('scroll', handleScroll);
	});
</script>

<div class="carousel" bind:this={containerRef}>
	<div class="scroll-content" style="height: {itemHeight * 1000000}px;">
		{#each visibleItemsArray as item (item.y)}
			<div class="card" style="transform: translateY({item.y}px);">
				{item.label}
			</div>
		{/each}
	</div>
</div>

<style lang="postcss">
	.carousel {
		height: 500px;
		overflow-y: scroll;
		position: relative;
	}

	.scroll-content {
		position: relative;
	}

	.card {
		position: absolute;
		width: 100%;
		height: 100px;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: theme(borderRadius.lg);
		box-shadow: theme(boxShadow.lg);
	}
</style>
