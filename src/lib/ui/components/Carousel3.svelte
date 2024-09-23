<script lang="ts">
	import { onMount } from 'svelte';

	const itemHeight = 100; //px
	const itemGap = 10; //px
	const nItemsAboveTop = 3;
	const items = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10'];
	let itemsWPosition = $state(
		items.map((item, index) => ({
			item,
			position: -nItemsAboveTop * itemHeight + itemGap + index * itemHeight + itemGap
		}))
	);

	const containerHeight = 500; //px
	const maxY = containerHeight + itemHeight * 3; //px
	const minY = -itemHeight * 3; //px

	let isDragging = $state(false);
	let selectedN = $state(Math.round(items.length / 2));
	let lastScreenY = 0;

	const moveCarousel = (deltaY: number) => {
		let adjustedItems = itemsWPosition.map(({ item, position }) => {
			return { item, position: position + deltaY };
		});

		const movedDownwards = deltaY > 0;

		// Re-arrange items that have gone outside the bounds, so that they 'wrap' to the top or bottom of array
		const itemsOutOfBounds = adjustedItems.filter(({ position }) =>
			movedDownwards ? position > maxY : position < minY
		);

		const sortedItems = itemsOutOfBounds
			.map(({ item, position }) => ({
				item,
				position
			}))
			.sort((a, b) => (movedDownwards ? b.position - a.position : a.position - b.position));

		sortedItems.forEach(({ item }) => {
			const index = adjustedItems.findIndex(({ item: item2 }) => item === item2);

			if (index !== -1) {
				if (movedDownwards) {
					const minPos = Math.min(...adjustedItems.map(({ position }) => position));
					adjustedItems[index].position = minPos - itemHeight;
				} else {
					const maxPos = Math.max(...adjustedItems.map(({ position }) => position));
					adjustedItems[index].position = maxPos + itemHeight;
				}
			}
		});

		itemsWPosition = adjustedItems;
	};

	const handleMouseDown = (event: MouseEvent) => {
    event.preventDefault();
    event.stopPropagation();

		isDragging = true;
	};

	const handleMouseUp = (event: MouseEvent) => {
		isDragging = false;
	};

	const handleDrag = (event: MouseEvent) => {
    event.preventDefault();
    event.stopPropagation();
    
		const deltaY = event.screenY - lastScreenY;
		lastScreenY = event.screenY;

		if (!isDragging) return;

		moveCarousel(deltaY);
	};

	onMount(() => {
    const interval = setInterval(() => {
      moveCarousel(0.5);
    }, 1);

    return () => clearInterval(interval);
  });

	$effect(() => {
		// The selected item is the one closest to the center of the container
		selectedN = itemsWPosition.reduce((acc, { position }, index) => {
			const currentDiff = Math.abs(position - containerHeight / 2);
			const accDiff = Math.abs(itemsWPosition[acc].position - containerHeight / 2);
			return currentDiff < accDiff ? index : acc;
		}, 0);
	});
</script>

<div
	class="relative w-full flex flex-col items-center overflow-hidden {isDragging
		? 'cursor-grabbing'
		: 'cursor-grab'}"
	style={`
    height: ${containerHeight}px;
    mask-image: linear-gradient(
			transparent,
			rgba(255, 255, 255, 0) 0%,
			rgba(255, 255, 255, 1) 25%,
			rgba(255, 255, 255, 1) 50%,
			rgba(255, 255, 255, 1) 75%,
			rgba(255, 255, 255, 0) 100%
		);
    `}
	onmousedown={handleMouseDown}
	onmouseup={handleMouseUp}
	onmouseleave={handleMouseUp}
	onmousemove={handleDrag}
>
	{#each itemsWPosition as { item, position }, index}
		<div
			class="absolute p-5 rounded-lg shadow-lg w-full max-w-[200px] flex justify-center"
			style="
        top: {position}px;
        transform: scale({1 - Math.abs(position - containerHeight / 2) / containerHeight});
        border: {index === selectedN ? '2px solid #000' : 'none'};
        "
		>
			{item}
		</div>
	{/each}
</div>
