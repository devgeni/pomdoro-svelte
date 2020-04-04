<script>
	import { padNum, getSeconds, getMinutes, getTime, getByKey } from "./helpers";

	const config = {
		focus: { id: 1, time: 20 * 60, sign: "o" },
		short: { id: 2, time: 5 * 60, sign: "|"},
		long: { id: 3, time: 10 * 60, sign: "_" }
	};

	Notification.requestPermission().then(result => {
		console.log("Notification permission", result);
	});

	const sound = new Audio("media/digital-alarm.mp3");
	
	// state
	let time = config.focus.time;
	let selected = config.focus;
	let isOn = false;
	let intervalId = null;
	let history = [];
	let strike = '';
	let volume = 50;

	const playSound = () => sound.play();
	
	const timeNotification = () => {
		const n = new Notification("Pomdoro! Time's up", {
			body: "Click to get back to your timer!"
		})
	};

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

	const incrementVolume = () => {
		volume = volume >= 100 ? 100 : volume + 10;
	};

	const decrementVolume = () => {
		volume = volume <= 0 ? 0 : volume - 10;
	};
	
	$: if (time <= 0) {
		time = 0;
		stopCountdown();
		history = [...history, selected.sign];
		timeNotification();
	}
	
	$: if (!isOn && time <= 0) {
		playSound();
	}

	$: sound.volume = volume / 100;
	$: title = (time == 0) ? "Pomdoro! Time's up!" : `Pomdoro ${getTime(time)}`;
	$: volumeHandle = (volume == 0 || volume == 100) ? `volume-handle-${volume}` : '';
</script>

<svelte:head>
	<title>{title}</title>
</svelte:head>

<div class="app">
	<h1 class="countdown">
		{getTime(time)}
	</h1>
	<h3 class:invisible={time !== 0} class="text-center font-bold">Your time is up!</h3>
	<p class="hint" style="text-decoration: { (time <= 0) ? 'line-through' : 'none' };">
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

	<div class="mb-5">
		<div class="flex justify-center mb-6">
			{#if !isOn}
			<button on:click={startCountdown}>
				start
			</button>
			{:else}
			<button on:click={stopCountdown}>
				pause
			</button>
			{/if}
			<button class="ml-6" on:click={reset}>
				reset
			</button>
		</div>
		<div class="flex justify-between mb-6">
			<button class:button-selected={selected.id === 1}  on:click={() => start("focus")}>
				20 min.
			</button>
			<button class:button-selected={selected.id === 2} on:click={() => start("short")}>
				5 min.
			</button>
			<button class:button-selected={selected.id === 3} on:click={() => start("long")}>
				10 min.
			</button>
		</div>
	</div>

	<p class="volume-text">Sound volume:</p>
	
	<div class="volume">
		<div class="volume-progress" style="width: {volume}%">
			<div class="volume-handle {volumeHandle}">
			{volume}</div>
		</div>
	</div>

	<div class="flex justify-center mt-6">
		<button on:click={decrementVolume}>-</button>
		<button class="ml-6" on:click={incrementVolume}>+</button>
	</div>

	<fieldset style="display: none;">
		0 <input type="range" bind:value={volume} step="10">100
		<br>
		<br>
	</fieldset>

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
</div>