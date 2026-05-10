<head>
    <title>Game Board</title>
</head>

<script lang="ts">
	
    import {toast} from '@zerodevx/svelte-toast';

    const size = 10;
    const cells = Array.from({ length:size*size},(_,i)=>{
        return{
            row: Math.floor(i/size),
            col: i%size
        };
    });
    
    let startloading = $state(true);
    let resetloading = $state(false);
    let rollLoading = $state(false);
    let showIconPicker = $state(false);
    let rollResult = $state("");
    let startMessage = $state("");
    let endMessage = $state("");
    let resetMessage = $state("");
    let iconMessage = $state("");
    let moveMessage = $state("");
    let game_id = $state("");
    let player_name = $state(0);
    let nextTurn = $state("");
    let playerPositions: Record<string, number> = $state({
        1:0,
        2:0,
        3:0,
        4:0
    });

    let activePlayer = $state(0);
    let selectedIcon: Record<string, string> = $state({
        1:"",
        2:"",
        3:"",
        4:""
    });
    const icons = ['red','blue','green','yellow'];

    function moveIcon(newIndex:number) {
        playerPositions[activePlayer]= newIndex;
        
        // playerPositions = {
        //     ...playerPositions,
        //     [activePlayer]: newIndex
        // }
    }

    async function start(numPlayers:number) {
        startloading = true;

        try {
            const response = await fetch("http://127.0.0.1:5000/start", {
                method:'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({numPlayers:numPlayers})
            });

            const data = await response.json();
            toast.push(data.message);
            startMessage = data.message;
            game_id = data.game_id;
            showIconPicker = true;
        } catch(err){
            startMessage = "Error starting game";
        } finally {
            startloading = false;
        }
    }

    async function submitIcons(game_id:string,selectedIcon:Record<string, string>) {

        try {
            const response = await fetch("http://127.0.0.1:5000/choose_icon", {
                method:'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({game_id:game_id,icons:selectedIcon})
            });

            const data = await response.json();
            toast.push(data.message);
            // selectedIcon = selectedIcon;
            // iconMessage = data.message;
        } catch(err) {
            console.error("Error submitting icons:", err);
        }
    }

    async function end(game_id:string) {
        try {
            const response = await fetch("http://127.0.0.1:5000/end", {
                method:'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({game_id:game_id})
            });

            const data = await response.json();
            // endMessage = data.message;
            toast.push(data.message);
        } catch(err) {
            // endMessage = "Error ending game";
            toast.push("Error ending game");
        }
    }

    async function details(url:string) {

        try {
            const response = await fetch(url);

            if(!response.ok){
                throw new Error("Network response was not ok");
            }

            const data = await response.json();
            player_name = data.current_turn
            activePlayer = player_name;
        } catch(err) {
            console.error("Error fetching game details:", err);
        }
    }
        
    async function rollDice(game_id:string,player_name:number){
        rollLoading = true;

        try {
            const response = await fetch("http://127.0.0.1:5000/roll", {
                method:'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({game_id:game_id,player_name:player_name})
            });

            const data = await response.json();

            rollResult = data.message;
            toast.push(data.message);
        } catch (err) {
            rollResult = "Error rolling dice";
            toast.push(rollResult);
        } finally {
            rollLoading = false;
        }   
    }

    async function movePlayer(game_id:string,player_name:number) {
        try {
            const response = await fetch("http://127.0.0.1:5000/move", {
                method:'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({game_id:game_id,player_name:player_name})
            });

            const data = await response.json();
            toast.push(data.message);
        } catch (err) {
            console.error("Error moving player:", err);
        }

    }

    async function endTurn(game_id:string) {
        try {
            const response = await fetch("http://127.0.0.1:5000/end_turn",{
                method:'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({game_id:game_id})
            });
            const data = await response.json();
            nextTurn = data.message;
            toast.push(data.message);

        } catch (err) {
            console.error("Error ending turn:", err);
        }
    }

    async function reset(game_id:string) {
        resetloading = true;
        
        try {
            const response = await fetch("http://127.0.0.1:5000/reset", {
                method:'POST',
                headers:{
                    'Content-Type':'application/json'
                },
                body: JSON.stringify({game_id:game_id})
            }); 

            const data = await response.json();
            // resetMessage = data.message;
            toast.push(data.message);
        } catch(err){
            // resetMessage = "Error resetting game";
            toast.push("Error resetting game");
        } finally {
            resetloading = false;
        }
    }

    async function handleRoll() {
        await details(`http://127.0.0.1:5000/game/details?game_id=${game_id}`);
        await rollDice(game_id, player_name);
        await movePlayer(game_id, player_name);
        await endTurn(game_id);
    }
</script>

{#if showIconPicker}
    <div class = "divider-section">
        <h3>Select Player Icons</h3>
        {#each Object.keys(selectedIcon) as player}
            <div class="divider-section">
                <p>Player {player}</p>
                {#each icons as icon}
                    <button 
                        class="button"
                        disabled = {Object.values(selectedIcon).includes(icon)  && selectedIcon[player] != icon}
                        onclick={() => selectedIcon[player] = icon}
                        style="background-color: {icon}; color: black;"
                        
                    >
                    {icon}
                    </button>
                {/each}
            </div>
        {/each}

            <button class="button" onclick = {() => submitIcons(game_id, selectedIcon)}>Submit Icons</button>
            {#if iconMessage}
                <p>{iconMessage}</p>
            {/if}
    </div>
{/if}

<div class="divider-section">
    <button class="button" 
        onclick={() => start(4)} disabled = {!startloading}>
        {#if startloading}
            Start Game
        {:else}
            Game in Progress
        {/if}
    </button>

    <!-- {#if startMessage}
        <p>toast.push("{startMessage}")</p>
    {/if} -->

    <button class="button" 
        onclick={() => handleRoll()} disabled = {!game_id}>
        {#if rollLoading}
            Rolling....
        {:else}
            Roll Dice
        {/if}
    </button>

    {#if rollResult}
        <p>{rollResult}</p>
    {/if}
    {#if moveMessage}
        <p>{moveMessage}</p>
    {/if}
    {#if nextTurn}
        <p>{nextTurn}</p>
    {/if}

    <button class="button" 
        onclick={() => reset(game_id)} disabled = {!game_id}>
        {#if resetloading}
            Resetting....
        {:else}
            Reset Game
        {/if}
    </button>

    <button class="button"
        onclick={() => end(game_id)} disabled={!game_id}>
        End Game
    </button>

    {#if resetMessage}
        <p>{resetMessage}</p>
    {/if}

    {#if endMessage}
        <p>{endMessage}</p>
    {/if}
</div>

<div>
    <p>Game ID: {game_id}</p>
    <p>Current Player: {activePlayer}</p>
    <p>Player Positions: {JSON.stringify(playerPositions)}</p>
    <p>Selected Icons: {JSON.stringify(selectedIcon)}</p>
</div>

<div class="grid">
    {#each cells as cell}
        
        <button 
            class="cells"
            onclick = {() => moveIcon(cell.row*size + cell.col)}
        >
            <div class="tile-icons">
                {#each Object.entries(playerPositions) as [player, position]}
                    {#if position === cell.row*size + cell.col}
                        <div class={selectedIcon[player]}></div>
                    {/if}
                {/each}
            </div>
        </button>
    {/each}
</div>

