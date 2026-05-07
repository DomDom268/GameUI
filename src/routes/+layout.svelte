<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
	import { SvelteToast } from '@zerodevx/svelte-toast';

	const options = {
        duration: 3000,
        position: 'top-right',
        theme: {
            '--toastBackground': '#1f2937',
            '--toastColor': '#ffffff',
            '--toastBarBackground': '#22c55e'
        }
    }

	let { children } = $props();
</script>

<SvelteToast {options}/>

<nav class= navbar>
	<div class="logo">Staplers and Scanners</div>
	<ul class="navlinks">
		<li><a href ="/">Home</a></li>
		<li><a href = "/rules">Rules</a></li>
	</ul>
</nav>

{@render children()}
