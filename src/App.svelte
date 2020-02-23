<script>
	import { padNum, getSeconds, getMinutes, getTime, getByKey } from "./helpers";

	const config = {
		focus: { id: 1, time: 20 * 60, sign: "o" },
		short: { id: 2, time: 5 * 60, sign: "|"},
		long: { id: 3, time: 10 * 60, sign: "_" }
	};

	const sound = new Audio("media/digital-alarm.mp3");
	
	// state
	let time = config.focus.time;
	let selected = config.focus;
	let isOn = false;
	let intervalId = null;
	let history = [];
	let strike = '';

	const startCountdown = () => {
		clearInterval(intervalId);
		intervalId = setInterval(() => {
			time--;
		}, 1000);
		isOn = true;
	};
	const stopCountdown = () => {
		clearInterval(intervalId);
		isOn = false;
	};
	const start = (type) => {
		time = config[type].time;
		selected = config[type];
		startCountdown();
	};
	const reset = () => {
		stopCountdown(); 
		time = selected.time;
	};
	
	$: if (time <= 0) {
		time = 0;
		stopCountdown();
		history = [...history, selected.sign];
		sound.play();
		strike = 'text-decoration: line-through';
	} else {
		strike = '';
	}
</script>

<svelte:head>
	<title>Pomdoro {getTime(time)}</title>
</svelte:head>

<h1>
	{getTime(time)}
</h1>
<p>
	<span style={strike}>
		{#if selected.id != 1} 
			It's
		{:else} 
			You should 
		{/if}
		<strong>{getByKey(config, (key) => config[key].id == selected.id)[0]}</strong>
		{#if selected.id != 1} 
			break now
		{/if}
	</span>
	{#if time == 0}
		Your time is up!
	{/if}
</p>
{#if !isOn}
<button on:click={startCountdown}>
	start
</button>
{:else}
<button on:click={stopCountdown}>
	pause
</button>
{/if}
<button on:click={reset}>
	reset
</button>
<br/>
<button on:click={() => start("focus")}>
	20
</button>
<button on:click={() => start("short")}>
	5
</button>
<button on:click={() => start("long")}>
	10
</button>
<h2>
	{ history.join("") }{#if isOn}<span style="opacity: 0.5">{selected.sign}</span>{/if}
</h2>