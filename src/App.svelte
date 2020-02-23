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
		strike = 'line-through';
	} else {
		strike = 'none';
	}

	$: title = (time == 0) ? "Pomdoro! Time's up!" : `Pomdoro ${getTime(time)}`;
</script>

<svelte:head>
	<title>{title}</title>
</svelte:head>

<h1>
	{getTime(time)}
</h1>
<p style="text-decoration: {strike};">
	{#if selected.id != 1} 
		It's
	{:else} 
		You should 
	{/if}
	<strong>{getByKey(config, (key) => config[key].id == selected.id)[0]}</strong>
	{#if selected.id != 1} 
		break now
	{/if}
</p>

{#if time == 0}
	<h3>Your time is up!</h3>
{/if}

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
	20 min.
</button>
<button on:click={() => start("short")}>
	5 min.
</button>
<button on:click={() => start("long")}>
	10 min.
</button>
<p>
	<strong>Legend:</strong> <br>
	&nbsp;&nbsp; <strong>o</strong> &emsp; focus time <br>
	&nbsp;&nbsp; <strong>|</strong> &emsp; short break <br>
	&nbsp;&nbsp; <strong>_</strong> &emsp; long break
</p>
<h2>
	{#if history.length || isOn}done: <br>{/if}
	{ history.join("") }{#if isOn}<span style="opacity: 0.5">{selected.sign}</span>{/if}
</h2>