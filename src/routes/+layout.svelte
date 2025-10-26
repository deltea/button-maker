<script lang="ts">
	import "../app.css";
	import "iconify-icon";

	let { children } = $props();
</script>

<svelte:head>
	<title>button maker</title>
</svelte:head>

<main class="h-screen">
	{@render children?.()}
</main>
