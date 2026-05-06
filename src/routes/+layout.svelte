<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<nav class= navbar>
	<div class="logo">Staplers and Scanners</div>
	<ul class="navlinks">
		<li><a href ="/">Home</a></li>
		<li><a href = "/rules">Rules</a></li>
	</ul>
</nav>

{@render children()}
