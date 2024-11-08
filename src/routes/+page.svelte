<script lang="ts">
	import Pause from 'lucide-svelte/icons/pause';
	import Play from 'lucide-svelte/icons/play';
	import RotateCcw from 'lucide-svelte/icons/rotate-ccw';
	import Forward from 'lucide-svelte/icons/forward';

	let time_work: number = 25*60;
	let time_pause: number = 5*60;
	let remaining: number = time_work;
	let duration: number = time_work;
	let isWorkSession: boolean = true;
	let isRunning: boolean = false;
	let interval: number | undefined = 10;
	let radius: number = 250;
	const stroke: number = 20;
	let circumference = 2 * Math.PI * radius;

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
				else switchSession();
			}, 1000);
		isRunning = !isRunning;
	}

	function formatTime(seconds: number) {
		const minutes = Math.floor(seconds / 60);
		const secondsLeft = seconds % 60;
		return `${String(minutes).padStart(2, "0")}m ${String(secondsLeft).padStart(2, "0")}s`;
	}

	$: offset = circumference - (remaining / duration) * circumference;
</script>

<div class="container h-full mx-auto flex justify-center items-center">
	<div class="h-screen justify-center items-center flex">
		<div class="justify-center items-center flex variant-glass-surface p-10 rounded-xl border-2 border-surface-500">
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
				<p class="font-bold text-7xl">{formatTime(remaining)}</p>
				<div>
					<button class="mt-8 bg-secondary-500 text-surface-100 rounded-full p-5" on:click={toggleTimer} disabled={isRunning && remaining === 0}>
						{#if !isRunning}
							<Play size={32}/>
						{:else}
							<Pause size={32}/>
						{/if}
					</button> 
					<div class="mt-3 space-x-3">
						<button class="bg-secondary-500 text-surface-100 rounded-full p-2" on:click={() => {switchSession();switchSession();}}>
							<RotateCcw/>
						</button> 
						<button class="bg-secondary-500 text-surface-100 rounded-full p-2" on:click={switchSession}>
							<Forward/>
						</button> 
					</div>
				</div>
			</div>
		</div>
	</div>
</div>