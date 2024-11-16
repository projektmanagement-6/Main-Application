<script lang="ts">
	import Pause from 'lucide-svelte/icons/pause';
	import Play from 'lucide-svelte/icons/play';
	import RotateCcw from 'lucide-svelte/icons/rotate-ccw';
	import Forward from 'lucide-svelte/icons/forward';
	import Save from 'lucide-svelte/icons/save-all';

	import { sound } from "svelte-sound";
	import { Sound } from "svelte-sound";

	import playStop_mp3 from "../SFX/Play_Stop_SFX.mp3";
	import reset_mp3 from "../SFX/Reset_SFX.mp3";
	import restart_mp3 from "../SFX/Restart_SFX.mp3";
	import save_mp3 from "../SFX/Save_SFX.mp3";
	import breakStart_mp3 from "../SFX/Break_Start_SFX.mp3";
	import sessionStart_mp3 from "../SFX/Session_Start_SFX.mp3"

	const reset_sound = new Sound(reset_mp3);
	const restart_sound = new Sound(restart_mp3);
	const break_sound = new Sound(breakStart_mp3);
	const session_sound = new Sound(sessionStart_mp3);

	function playSound(sfx: Sound) {
		sfx.play();
	}

	let time_work: number = 25*60;
	let time_pause: number = 5*60;
	let remaining: number = time_work;
	let duration: number = time_work;
	let isWorkSession: boolean = true;
	let isRunning: boolean = false;
	let interval: number | undefined = 10;
	let radius: number = 230;
	const stroke: number = 20;
	let circumference = 2 * Math.PI * radius;

	function changeTime(minutes: number) {
		const secondsToChange = minutes * 60;
		remaining = Math.max(0, remaining + secondsToChange);
		duration = remaining;
	}

	function saveTime() {
		if (isWorkSession){
			time_work = duration;
		}
		else {
			time_pause = duration;
		}
	}

	function resetTime() {
		if (isWorkSession){
			time_work = 25*60;
		}
		else {
			time_pause = 5*60;
		}
	}
	
	function switchSession() {
		remaining = (isWorkSession) ? time_pause : time_work;
		duration = remaining;
		isWorkSession = !isWorkSession; 
	}

	function toggleTimer() {
		if (isRunning) {
			clearInterval(interval);
			interval = undefined;
		} else
			interval = window.setInterval(() => {
				if (remaining > 0) remaining--;
				else {
					if (isWorkSession){playSound(break_sound);}
					else {playSound(session_sound);}
					switchSession();
				}
			}, 1000);
		isRunning = !isRunning;
	}

	function formatTime(seconds: number) {
		const minutes = Math.floor(seconds / 60);
		const secondsLeft = seconds % 60;
		return `${String(minutes).padStart(2, "0")}:${String(secondsLeft).padStart(2, "0")}`;
	}

	$: offset = circumference - (remaining / duration) * circumference;
</script>

<div class="container h-full mx-auto flex  justify-center items-center">
	<div class="h-screen justify-center items-center flex ">
		<div class="justify-center items-center flex variant-glass-surface p-10 rounded-xl shadow-2xl">
			<svg width="{(radius + stroke/2) * 2}" height="{(radius + stroke/2) * 2}">	
				<circle
					cx="{radius + stroke / 2}"
					cy="{radius + stroke / 2}"
					r={radius}
					fill="none"
					stroke="rgba(var(--color-surface-800)"
					stroke-width={stroke}
				/>
				<circle
					cx="{radius + stroke / 2}"
					cy="{radius + stroke / 2}"
					r={radius}
					fill="none"
					stroke="rgba(var({(isWorkSession) ? "--color-primary-500" : "--color-error-500"})"
					stroke-width={stroke}
					stroke-dasharray={circumference}
					stroke-dashoffset={offset}
					stroke-linecap="round"
					style="transition: stroke-dashoffset 0.1s linear;"
				/> 
			</svg>
			<div class="absolute text-center">
				<p class="lining-nums font-solid font-semibold text-8xl">{formatTime(remaining)}</p>
				<div>
					<div class="mt-6" style="display: flex; align-items: center;">
					<button style="font-size: 12px; font-weight: bold;" class="shadow-md m-2 variant-ghost-tertiary rounded-full w-14 h-14 p-3" on:click={() => changeTime(-5)}>
						- 5m
					</button>
					<button style="font-size: 12px; font-weight: bold;" class="shadow-md m-2 variant-ghost-tertiary rounded-full w-14 h-14 p-3" on:click={() => changeTime(-1)}>
						- 1m
					</button>
					<button class="shadow-md m-2 variant-ghost-tertiary text-surface-100 rounded-full p-5" use:sound={{ src: playStop_mp3, events: ["click"]}}  on:click={toggleTimer} disabled={isRunning && remaining === 0}>
						{#if !isRunning}
							<Play size={32}/>
						{:else}
							<Pause size={32}/>
						{/if}
					</button>
					<button style="font-size: 12px; font-weight: bold;" class="shadow-md m-2 variant-ghost-tertiary rounded-full w-14 h-14 p-3" on:click={() => changeTime(1)}>
						+ 1m
					</button>
					<button style="font-size: 12px; font-weight: bold;" class="shadow-md m-2 variant-ghost-tertiary rounded-full w-14 h-14 p-3" on:click={() => changeTime(5)}>
						+ 5m
					</button>
				</div>
					<div class="mt-3 space-x-3">
						<button id ="reset-btn" class="shadow-md variant-ghost-surface text-surface-100 rounded-full p-2" on:click={(event) => {
							if (event.detail === 1) {
								switchSession();
								switchSession();
								playSound(restart_sound);
							} else if (event.detail === 2) {
								resetTime();
								playSound(reset_sound);
							}
						}}>
							<RotateCcw/>
						</button> 
						<button class="shadow-md variant-ghost-surface text-surface-100 rounded-full p-2" on:click={switchSession}>
							<Forward/>
						</button> 
						<button class="shadow-md variant-ghost-surface text-surface-100 rounded-full p-2" use:sound={{ src: save_mp3, events: ["click"]}} on:click={saveTime}>
							<Save />
						</button>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
