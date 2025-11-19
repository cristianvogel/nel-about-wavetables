<script>
	import { onMount } from 'svelte';

	let serumCanvas = $state();
	let waveEditCanvas = $state();
	let chart1;
	let chart2;

	onMount(async () => {
		// Check if Chart is loaded, if not wait for it (parent loads it, or we can reload)
		if (typeof Chart === 'undefined') {
			const script = document.createElement('script');
			script.src = 'https://cdn.jsdelivr.net/npm/chart.js';
			document.head.appendChild(script);
			await new Promise((resolve) => (script.onload = resolve));
		}
		initCharts();
	});

	function initCharts() {
		if (!serumCanvas || !waveEditCanvas) return;

		// Destroy existing charts if re-initializing
		if (chart1) chart1.destroy();
		if (chart2) chart2.destroy();

		const commonOptions = {
			responsive: true,
			maintainAspectRatio: false,
			plugins: {
				legend: { display: false },
				tooltip: { enabled: false }
			},
			scales: {
				x: { display: false },
				y: { display: false, min: -1.2, max: 1.2 }
			},
			animation: {
				duration: 2000,
				easing: 'easeOutQuart'
			},
			elements: {
				point: { radius: 0 }
			}
		};

		// Chart 1: Serum (Smooth / High Resolution)
		chart1 = new Chart(serumCanvas, {
			type: 'line',
			data: {
				labels: Array.from({ length: 100 }, (_, i) => i),
				datasets: [
					{
						data: Array.from({ length: 100 }, (_, i) => Math.sin(i * 0.1) + Math.sin(i * 0.3) * 0.5),
						borderColor: '#4ade80', // green-400
						borderWidth: 2,
						backgroundColor: 'rgba(74, 222, 128, 0.1)',
						fill: true,
						tension: 0.4 // Smooth spline
					}
				]
			},
			options: commonOptions
		});

		// Chart 2: WaveEdit (Stepped / Low Resolution)
		chart2 = new Chart(waveEditCanvas, {
			type: 'line',
			data: {
				labels: Array.from({ length: 40 }, (_, i) => i),
				datasets: [
					{
						data: Array.from({ length: 40 }, (_, i) => Math.sin(i * 0.25) + Math.sin(i * 0.75) * 0.5),
						borderColor: '#fb923c', // orange-400
						borderWidth: 2,
						backgroundColor: 'rgba(251, 146, 60, 0.1)',
						fill: true,
						stepped: true // The visual "digital" step effect
					}
				]
			},
			options: commonOptions
		});
	}
</script>

<!-- Footer / Visual Comparison -->
<div class="mt-12 max-w-7xl mx-auto border-t border-slate-800 pt-12">
	<h3 class="text-2xl font-bold text-slate-200 mb-6 text-center">Visualizing the Difference</h3>
	<div class="grid grid-cols-1 md:grid-cols-2 gap-8">
		<!-- Serum Visual -->
		<div class="bg-slate-900 p-6 rounded-xl border border-slate-800">
			<div class="flex justify-between items-end mb-4">
				<span class="text-green-400 font-bold">Serum / Modern</span>
				<span class="text-xs text-slate-500">High Resolution (Smooth)</span>
			</div>
			<div class="h-40 w-full">
				<canvas bind:this={serumCanvas}></canvas>
			</div>
			<p class="mt-4 text-sm text-slate-400">
				<span class="font-bold text-slate-200">2048 samples</span> per wave allows for extremely smooth
				sine waves and complex harmonics without aliasing.
			</p>
		</div>

		<!-- WaveEdit Visual -->
		<div class="bg-slate-900 p-6 rounded-xl border border-slate-800">
			<div class="flex justify-between items-end mb-4">
				<span class="text-orange-400 font-bold">WaveEdit / Hardware</span>
				<span class="text-xs text-slate-500">Memory Efficient (Stepped)</span>
			</div>
			<div class="h-40 w-full">
				<canvas bind:this={waveEditCanvas}></canvas>
			</div>
			<p class="mt-4 text-sm text-slate-400">
				<span class="font-bold text-slate-200">256 samples</span> per wave creates a "steppier" digital
				sound, often desired for "glitch" or "retro" aesthetics in Eurorack.
			</p>
		</div>
	</div>
</div>