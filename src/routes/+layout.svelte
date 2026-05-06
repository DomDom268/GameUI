<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
	import { SvelteToast } from '@zerodevx/svelte-toast';

	let { children } = $props();
</script>

<SvelteToast/>

<nav class= navbar>
	<div class="logo">Staplers and Scanners</div>
	<ul class="navlinks">
		<li><a href ="/">Home</a></li>
		<li><a href = "/rules">Rules</a></li>
	</ul>
</nav>

{@render children()}
