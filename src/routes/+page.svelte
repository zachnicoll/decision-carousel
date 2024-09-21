<script lang="ts">
	const items = [
		{ label: 'Something' },
		{ label: 'Else' },
		{ label: 'Entirely' },
		{ label: 'Something' },
		{ label: 'Something' },
		{ label: 'Something' },
		{ label: 'Something' },
		{ label: 'Something' }
	];

	const showNItems = 5;
	let selectedN = $state(Math.round(items.length / 2));
	let currentN = $state(selectedN);

  const halfHeight = 250;

	const calculateYTranslation = (index: number) => {
    // Calculate the translation for this item, based on the selected item
    // such that the selected item is always in the middle of the carousel
    if (index === currentN) {
      return halfHeight;
    }

    if (index < currentN) {
      return halfHeight - (currentN - index) * 75;
    }
    
    return halfHeight + (index - currentN) * 150;
	};

  const calculateZIndex = (index: number) => {
    if (index === currentN) {
      return items.length;
    }

    return items.length - Math.abs(currentN - index);
  };

	const calculateScale = (index: number) => {
		const diff = Math.abs(currentN - index);
		return 1 - diff / items.length;
	};

  const calculateDisplay = (index: number) => {
    return index < currentN - showNItems || index > currentN + showNItems ? 'none' : 'flex';
  };

	const handleClick = (index: number) => {
		selectedN = index;
	};

  const spin = () => {
    selectedN = Math.floor(Math.random() * items.length);
  };

  $effect(() => {
    const interval = setInterval(() => {
      if (currentN === selectedN) {
        return;
      }
      if (currentN < selectedN) {
        currentN++;
      } else {
        currentN--;
      }
    }, 100);

    return () => clearInterval(interval);
  })
</script>

<div class="w-full flex items-center justify-center gap-4">
	<div class="carousel border border-black">
		{#each items as item, i}
			<button
				class={`card transform-gpu`}
				style={`
        scale: ${calculateScale(i)};
        transform: translate(0, calc(-50% + ${calculateYTranslation(i)}px));
        z-index: ${calculateZIndex(i)};
        display: ${calculateDisplay(i)};
        `}
				onclick={() => handleClick(i)}
				tabindex={i}
			>
				{item.label}
			</button>
		{/each}
	</div>
</div>
<button onclick={spin}>Spin</button>


<style lang="postcss">
	.carousel {
		display: flex;
		flex-direction: column;
		gap: theme(gap.2);
		padding: theme(padding.4);
    height: 500px;
    position: relative;
    flex: 1;
    align-items: center;
    overflow: hidden;
	}

	.card {
		transition: all 0.3s;
		width: 300px;
		min-height: 75px;
    position: absolute;
    top: 0;
    background-color: white;


		border-radius: theme(borderRadius.lg);
		box-shadow: theme(boxShadow.lg);
		justify-content: center;
		align-items: center;
	}
</style>
