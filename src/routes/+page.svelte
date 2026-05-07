<head>
    <title>Home</title>
</head>

<div class="pagetitle">
    <p>Welcome to Staplers and Scanners!</p>
</div>

<a href="/board" class="button">Play Game</a>

<script lang="ts">
    import { toast } from '@zerodevx/svelte-toast';

    function showToast() {
        toast.push("This is a toast notification!");
    }
</script>

<button class="button" onclick = {showToast}>Test Toast</button>