<script>
	import { onMount } from 'svelte';
	import VisualComparison from '$lib/VisualComparison.svelte';

	let selectedFormat = $state('gendyn');
	let serumCanvas = $state();
	let waveEditCanvas = $state();
	let chart1;
	let chart2;

	// Icon data for self-contained rendering
	const iconPaths = {
		FileCode: '<path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"/><polyline points="14 2 14 8 20 8"/><path d="m10 13-2 2 2 2"/><path d="m14 17 2-2-2-2"/>',
		Zap: '<polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/>',
		Cpu: '<rect x="4" y="4" width="16" height="16" rx="2" ry="2"/><rect x="9" y="9" width="6" height="6"/><line x1="9" x2="9" y1="1" y2="4"/><line x1="15" x2="15" y1="1" y2="4"/><line x1="9" x2="9" y1="20" y2="23"/><line x1="15" x2="15" y1="20" y2="23"/><line x1="20" x2="23" y1="9" y2="9"/><line x1="20" x2="23" y1="14" y2="14"/><line x1="1" x2="4" y1="9" y2="9"/><line x1="1" x2="4" y1="14" y2="14"/>',
		Layers: '<polygon points="12 2 2 7 12 12 22 7 12 2"/><polyline points="2 17 12 22 22 17"/><polyline points="2 12 12 17 22 12"/>',
		Keys: '<path d="M20 2H4C2.9 2 2 2.9 2 4V20C2 21.11 2.9 22 4 22H20C21.11 22 22 21.11 22 20V4C22 2.9 21.11 2 20 2M14.74 14H15V20H9V14H9.31C9.86 14 10.3 13.56 10.3 13V4H13.75V13C13.75 13.56 14.19 14 14.74 14M4 4H6.8V13C6.8 13.56 7.24 14 7.79 14H8V20H4V4M20 20H16V14H16.26C16.81 14 17.25 13.56 17.25 13V4H20V20Z" />',
		FileAudio: '<path d="M17.5 22h.5c.5 0 1-.2 1.4-.6.4-.4.6-.9.6-1.4V7.5L14.5 2H6c-.5 0-1 .2-1.4.6C4.2 3 4 3.5 4 4v3"/><polyline points="14 2 14 8 20 8"/><path d="M10 20v-6"/><path d="M6 20v-4"/><path d="M14 20v-4"/>',
		Music: '<path d="M9 18V5l12-2v13"/><circle cx="6" cy="18" r="3"/><circle cx="18" cy="16" r="3"/>',
		ArrowRight: '<line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/>',
		Info: '<circle cx="12" cy="12" r="10"/><line x1="12" y1="16" x2="12" y2="12"/><line x1="12" y1="8" x2="12.01" y2="8"/>',
		Settings: '<path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.1a2 2 0 0 1-1-1.74v-.5a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"/><circle cx="12" cy="12" r="3"/>',
		Activity: '<polyline points="2 12 4 17 6 7 8 19 10 5 12 18 14 6 16 16 18 9 20 14 22 12"/>'
	};

	const formats = {
		waldorf: {
			id: 'waldorf',
			title: 'Legacy Waldorf / PPG',
			subtitle: 'The 80s Wavetable Synth',
			iconName: 'Keys',
			color: 'text-red-400',
			bg: 'bg-red-900/20',
			border: 'border-red-500/50',
			image: 'ppg.png',
			specs: {
				frameSize: '128 Samples (PPG)',
				maxFrames: '64 Waves',
				bitDepth: '8-bit / 12-bit',
				fileType: '.SYX / EPROM Data'
			},
			description:
				'The grandfather of wavetable synthesisers. The original PPG Wave stored 64 waves per table, each only 128 samples per cycle. It was very expensive at the time, competing with the Fairlight system in top studios. Its low resolution (8/12-bit) was fresh in 80s synth pop. Notable users Thomas Dolby, Trevor Horn, Depeche Mode, Tangerine Dream.',
			compatible: ['PPG Wave 2.2/2.3', 'Waldorf Microwave 1', 'Waldorf Blofeld']
		},

		gendyn: {
			id: 'gendyn',
			title: 'Xenakis GENDYN',
			subtitle: 'Dynamic Stochastic',
			iconName: 'Activity',
			color: 'text-purple-400',
			bg: 'bg-purple-900/20',
			border: 'border-purple-500/50',
			specs: {
				frameSize: 'None (Calculated)',
				maxFrames: 'Infinite',
				bitDepth: 'Floating Point',
				fileType: 'Executable / Code'
			},
			image: 'Xenakis-in-front-of-the-upic-board.jpg',
			description:
				'Un-repeatable anti-pop. Iannis Xenakis used the medium of digital audio in a radical way, to produce a music without cycles. Instead of reading a stored table, the waveform is generated live via a "random walk" algorithm. The sample points in the waveform move stochastically, creating constantly evolving, unstable timbres. He favoured non-interpolation which made for noisy, aggresive, aliasing waves.',
			compatible: ['GENDYN (Original)', 'iatro (Max/MSP)', 'Kyma', 'CDP']
		},


		kyma: {
			id: 'kyma',
			title: 'Symbolic Sound Kyma',
			subtitle: 'The 90s',
			iconName: 'Cpu',
			color: 'text-orange-400',
			bg: 'bg-orange-900/20',
			border: 'border-black-500/50',
			specs: {
				frameSize: '4096 Samples',
				maxFrames: 'Variable',
				bitDepth: '8/16/24-bit Linear PCM',
				fileType: 'mono AIF/WAV'
			},
			description:
				'Kyma and it\'s hardware was the most powerful realtime sound programming environment in the mid-90s and is still state-of-the-art today. The digital wavetable oscillators and realtime stochastic synthesis has been inspired by Iannis Xenakis pioneering work with stochastic synthesis.',
			image: 'kyma.jpg',
			compatible: ['The music of Cristián Vogel', 'Carla Scaletti (co-founder of Symbolic Sound)', 'any PCM based format']
		},



		serum: {
			id: 'serum',
			title: 'Serum Format',
			subtitle: 'The Modern Software Standard',
			iconName: 'Zap',
			color: 'text-green-400',
			bg: 'bg-green-900/20',
			border: 'border-green-500/50',
			specs: {
				frameSize: '2048 Samples',
				maxFrames: '256 Frames',
				bitDepth: '32-bit Float (Typical)',
				fileType: '.WAV (Metadata Header)'
			},
			description:
				'The de-facto standard for modern software synthesizers (Vital, Phase Plant, etc.). Uses a specific header to map the audio file into chunks of 2048 samples.',
			compatible: ['Xfer Serum', 'Vital', 'Kilohearts Phase Plant', 'Parawave Rapid']
		},
		waveedit: {
			id: 'waveedit',
			title: 'WaveEdit Format',
			subtitle: 'The Hardware / Eurorack Standard',
			iconName: 'Cpu',
			color: 'text-orange-400',
			bg: 'bg-orange-900/20',
			border: 'border-orange-500/50',
			specs: {
				frameSize: '256 Samples',
				maxFrames: '64 Frames (1 Bank)',
				bitDepth: '16-bit PCM',
				fileType: '.WAV (Standard)'
			},
			image: 'wave-edit.jpg',
			description:
				'Optimized for hardware with limited memory. Commonly used in Eurorack modules (Synthesis Technology E352/E370) and boutique digital synths.',
			compatible: ['WaveEdit App', 'SynthTech E352/E370', 'Korg modwave (Import)', 'Piston Honda Mk3']
		},
		bitwig: {
			id: 'bitwig',
			title: 'Bitwig .WT',
			subtitle: 'The Explicit Container',
			iconName: 'FileCode',
			color: 'text-amber-400',
			bg: 'bg-amber-900/20',
			border: 'border-amber-500/50',
			specs: {
				frameSize: 'Variable (Header Defined)',
				maxFrames: 'Variable',
				bitDepth: '32-bit Float / 16-bit',
				fileType: '.WT (WAV + Metadata)'
			},
			image: 'bw-wt-browser.jpg',
			description:
				'A specialized format (adapted from Dune soft synth) that explicitly declares frame size in powers of two and other strict metadata. Used in the Bitwig Studio IDE and the open source Surge Synth XT. Bitwig users apply these WT tables to Oscillators, LFOs and even Waveshaping. The WT and WT Curves formats are use in the fantastic 3D morphing native synth instruments, the Polymer, the Wavetable LFO modulator and the Grid.',
			compatible: ['Bitwig Studio (Polymer/Grid)', 'Synapse Dune 2/3', 'Surge XT']
		},
		ableton: {
			id: 'ableton',
			title: 'Ableton Wavetable',
			subtitle: 'The Flexible Adapter',
			iconName: 'Layers',
			color: 'text-blue-400',
			bg: 'bg-blue-900/20',
			border: 'border-blue-500/50',
			specs: {
				frameSize: '1024 Samples (Internal)',
				maxFrames: 'Flexible (Resamples)',
				bitDepth: '32-bit Float',
				fileType: 'Reads .WAV / .AIFF'
			},
			description:
				'Ableton\'s Wavetable synth is an "omnivores". It can import Serum files or raw audio, but internally downsamples everything to 1024 samples per frame for processing.',
			compatible: ['Ableton Live 10/11/12 Suite']
		}
	};

	onMount(async () => {
		// Dynamically load Chart.js if not present
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
		// Simulating 2048 samples with a smooth sine wave
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
		// Simulating 256 samples with a stepped/quantized look
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

<!-- Reusable Snippets -->
{#snippet icon(name, size = 24, className = '')}
	<svg
		xmlns="http://www.w3.org/2000/svg"
		width={size}
		height={size}
		viewBox="0 0 24 24"
		fill="none"
		stroke="currentColor"
		stroke-width="2"
		stroke-linecap="round"
		stroke-linejoin="round"
		class={className}
	>
		{@html iconPaths[name] || ''}
	</svg>
{/snippet}

{#snippet specCard(label, value, iconName, color)}
	<div class="bg-slate-950/50 p-4 rounded-xl border border-slate-800/50 flex items-start gap-3">
		<div class="mt-1 {color}">
			{@render icon(iconName, 18)}
		</div>
		<div>
			<div class="text-xs text-slate-500 uppercase font-bold mb-1">{label}</div>
			<div class="text-slate-200 font-medium">{value}</div>
		</div>
	</div>
{/snippet}

<!-- Main Layout -->
<div
	class="min-h-screen bg-slate-950 text-slate-100 p-4 md:p-8 font-sans selection:bg-indigo-500 selection:text-white"
>
	<!-- Header -->
	<header class="mb-12 text-center max-w-4xl mx-auto">
		<div
			class="inline-flex items-center justify-center p-3 h-10"
		>
			<img src="nel_logo_text_nobg.svg" alt="NeverEngineLabs logo"/>
		</div>
		<h1
			class="text-4xl md:text-6xl font-bold mb-4 bg-gradient-to-r from-white via-slate-200 to-slate-400 bg-clip-text text-transparent"
		>
			Digital Wavetable Synthesis
		</h1>
		<p class="text-lg text-slate-400 max-w-2xl mx-auto">
			A Partial history and data guide for digital sound designers.
		</p>
	</header>

	<div class="grid grid-cols-1 lg:grid-cols-12 gap-8 max-w-7xl mx-auto">
		<!-- Left Column: Selector -->
		<div class="lg:col-span-4 space-y-4">
			{#each Object.values(formats) as format (format.id)}
				<button
					onclick={() => (selectedFormat = format.id)}
					class="w-full text-left p-4 rounded-xl border transition-all duration-300 group relative overflow-hidden {selectedFormat ===
					format.id
						? `${format.bg} ${format.border} shadow-lg`
						: 'bg-slate-900/50 border-slate-800 hover:bg-slate-800 hover:border-slate-700'}"
				>
					<div class="flex items-center justify-between relative z-10">
						<div class="flex items-center gap-4">
							<div
								class="p-3 rounded-lg bg-slate-950 {selectedFormat === format.id
									? format.color
									: 'text-slate-500 group-hover:text-slate-300'}"
							>
								{@render icon(format.iconName, 32)}
							</div>
							<div>
								<h3
									class="font-bold text-lg {selectedFormat === format.id
										? 'text-white'
										: 'text-slate-300'}"
								>
									{format.title}
								</h3>
								<p class="text-xs text-slate-500 uppercase tracking-wider font-semibold">
									{format.subtitle}
								</p>
							</div>
						</div>
						{#if selectedFormat === format.id}
							{@render icon('ArrowRight', 20, format.color)}
						{/if}
					</div>
				</button>
			{/each}
		</div>

		<!-- Right Column: Details -->
		<div class="lg:col-span-8">
			<div
				class="bg-slate-900/50 backdrop-blur-sm border border-slate-800 rounded-2xl p-6 md:p-10 h-full relative overflow-hidden"
			>
				<!-- Decorative background gradient -->
				<div
					class="absolute top-0 right-0 w-96 h-96 bg-gradient-to-br {formats[selectedFormat].color
						.replace('text-', 'from-')
						.replace('400', '900')} to-transparent opacity-10 blur-3xl pointer-events-none rounded-full -mr-20 -mt-20"
				></div>

				<div class="relative z-10">
					<div class="flex items-center gap-4 mb-8">
						<div
							class="p-4 rounded-2xl bg-slate-950 border border-slate-800 shadow-xl {formats[
								selectedFormat
							].color}"
						>
							{@render icon(formats[selectedFormat].iconName, 32)}
						</div>
						<div>
							<h2 class="text-3xl font-bold text-white mb-1">{formats[selectedFormat].title}</h2>
							<div class="flex items-center gap-2 text-slate-400">
								{@render icon('Info', 14)}
								<span class="text-sm">{formats[selectedFormat].subtitle}</span>
							</div>
						</div>
					</div>

					<!-- Specs Grid -->
					<div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-8">
						{@render specCard(
							'Frame Size (Cycle)',
							formats[selectedFormat].specs.frameSize,
							'Settings',
							formats[selectedFormat].color
						)}
						{@render specCard(
							'Max Table Length',
							formats[selectedFormat].specs.maxFrames,
							'Layers',
							formats[selectedFormat].color
						)}
						{@render specCard(
							'Bit Depth',
							formats[selectedFormat].specs.bitDepth,
							'Database',
							formats[selectedFormat].color
						)}
						{@render specCard(
							'File Container',
							formats[selectedFormat].specs.fileType,
							'FileAudio',
							formats[selectedFormat].color
						)}
					</div>

					<!-- Description -->
					<div class="mb-8">
						<h4 class="text-sm font-semibold text-slate-500 uppercase mb-3 tracking-widest">
							Technical Description
						</h4>
						<p class="text-slate-300 leading-relaxed text-lg">
							{formats[selectedFormat].description}
						</p>
						{#if formats[selectedFormat].image}
							<div class="mt-6">
								<img src={formats[selectedFormat].image} alt="Format Illustration" class="w-full rounded-xl" />
							</div>
							{/if}
					</div>

						<h4 class="text-sm font-semibold text-slate-500 uppercase mb-4 tracking-widest">
							Audio example
						</h4>
					<hr class="my-4 border-slate-700/50"/>
					{#if selectedFormat === 'kyma'}
						<iframe style="border: 0; width: 100%; height: 42px;" src="https://bandcamp.com/EmbeddedPlayer/album=1980001639/size=small/bgcol=111/linkcol=0687f5/track=2460169885/transparent=true/" seamless><a href="https://cristianvogel.bandcamp.com/album/the-assistenz-2016">The Assistenz (2016) by Cristian Vogel</a></iframe>
						{:else if selectedFormat === 'gendyn'}
						<iframe style="border: 0; width: 100%; height: 42px;" src="https://bandcamp.com/EmbeddedPlayer/album=1916547661/size=small/bgcol=333333/linkcol=0f91ff/track=3528246608/transparent=true/" seamless><a href="https://neumarecords.bandcamp.com/album/iannis-xenakis-a-s-gendy3-taurhiphanie-thalle-n">Iannis Xenakis: Aïs - Gendy3 - Taurhiphanie - Thalleïn by Iannis Xenakis</a></iframe>
						{:else if selectedFormat === 'bitwig'}
						<iframe width="100%" height="300" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/playlists/soundcloud%253Aplaylists%253A2020208517&color=%233d455c&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/bitwig" title="Bitwig" target="_blank" style="color: #cccccc; text-decoration: none;">Bitwig</a> · <a href="https://soundcloud.com/bitwig/sets/the-art-of-wavetables-demos" title="The Art of Wavetables by Cristian Vogel Demos" target="_blank" style="color: #cccccc; text-decoration: none;">The Art of Wavetables by Cristian Vogel Demos</a></div>
						{:else}
						<p class="text-gray-600 text-sm">Coming soon.</p>
					{/if}
					<!-- Compatibility -->
					<div class="mt-12">
						<h4 class="text-sm font-semibold text-slate-500 uppercase mb-4 tracking-widest">
							Commonly Used In
						</h4>
						<div class="flex flex-wrap gap-3">
							{#each formats[selectedFormat].compatible as item}
								<span
									class="px-4 py-2 rounded-lg bg-slate-950 border border-slate-800 text-slate-300 text-sm font-medium flex items-center gap-2"
								>
									{@render icon('Music', 14, formats[selectedFormat].color)}
									{item}
								</span>
							{/each}
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>

	{#if selectedFormat === 'serum' || selectedFormat === 'waveedit'}
		<VisualComparison />
	{/if}
</div>