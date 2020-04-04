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

	$: {
		sound.volume = volume / 100;
	}
	
	$: if (time <= 0) {
		time = 0;
		stopCountdown();
		history = [...history, selected.sign];
		timeNotification();
	}
	
	$: if (!isOn && time <= 0) {
		playSound();
	}

	$: title = (time == 0) ? "Pomdoro! Time's up!" : `Pomdoro ${getTime(time)}`;
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
		<div class="flex justify-between">
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

	<fieldset>
		<legend>Sound volume {volume}</legend>
		0 <input type="range" bind:value={volume} step="10">100
		<br>
		<br>
		<button on:click={decrementVolume}>-</button>
		<button on:click={incrementVolume}>+</button>
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
	
<style>
	button {
		outline: 0;
		border: 0;
		border-radius: 16px;
		min-width: 40px;
		padding: 10px 20px;
		font: 400 16px "SF Pro Display", sans-serif;
		color: #334669;
		background: #E4F0FA;
		box-shadow: 19px 21px 50px rgba(176, 195, 210, 0.727846), -8px 0px 8px rgba(244, 248, 251, 0.50254), -8px -40px 22px rgba(246, 251, 255, 0.384288), -11px -11px 20px rgba(255, 255, 255, 0.272044), inset 1px 1px 0px rgba(255, 255, 255, 0.5);
	}

	button:active {
		background: #DDECF9;
	}

	.button-selected {
		color: #6E81A0;
		background: #E3EDF7;
		box-shadow: inset 0px 1.96396px 3.92793px #C8D4E2, inset 3.92793px 1.96396px 5.89189px #B1C5D5, inset 2.94595px 5.89189px 5.89189px #C3D7E7, inset -5.89189px -2.94595px 4.90991px rgba(255, 255, 255, 0.750601);
	}

	fieldset {
		margin-bottom: 16px;
		max-width: 360px;
	}

	.app {
		box-sizing: border-box;
		border-radius: 30px;
		margin: 56px auto;
		max-width: 360px;
		min-height: 640px;
		padding: 24px;
		background: #E4F0FA;
		box-shadow: 50px 80px 80px rgba(0, 0, 0, 0.18);
	}

	.countdown {
		margin-bottom: 14px;
		font: 900 28px "SF Pro Display", sans-serif;
		text-align: center;
		color: #334669;
	}

	.hint {
		margin-bottom: 36px;
		font: 100 16px/1.3 "SF Pro Display", sans-serif;
		text-align: center;
		color: #334669;
	}

</style>