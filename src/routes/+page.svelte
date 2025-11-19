<script>
	let selectedFormat = $state('bitwig');

	// Icon data for self-contained rendering
	const iconPaths = {
		FileCode: '<path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"/><polyline points="14 2 14 8 20 8"/><path d="m10 13-2 2 2 2"/><path d="m14 17 2-2-2-2"/>',
		Zap: '<polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/>',
		Cpu: '<rect x="4" y="4" width="16" height="16" rx="2" ry="2"/><rect x="9" y="9" width="6" height="6"/><line x1="9" x2="9" y1="1" y2="4"/><line x1="15" x2="15" y1="1" y2="4"/><line x1="9" x2="9" y1="20" y2="23"/><line x1="15" x2="15" y1="20" y2="23"/><line x1="20" x2="23" y1="9" y2="9"/><line x1="20" x2="23" y1="14" y2="14"/><line x1="1" x2="4" y1="9" y2="9"/><line x1="1" x2="4" y1="14" y2="14"/>',
		Layers: '<polygon points="12 2 2 7 12 12 22 7 12 2"/><polyline points="2 17 12 22 22 17"/><polyline points="2 12 12 17 22 12"/>',
		Database: '<ellipse cx="12" cy="5" rx="9" ry="3"/><path d="M21 12c0 1.66-4 3-9 3s-9-1.34-9-3"/><path d="M3 5v14c0 1.66 4 3 9 3s9-1.34 9-3V5"/>',
		FileAudio: '<path d="M17.5 22h.5c.5 0 1-.2 1.4-.6.4-.4.6-.9.6-1.4V7.5L14.5 2H6c-.5 0-1 .2-1.4.6C4.2 3 4 3.5 4 4v3"/><polyline points="14 2 14 8 20 8"/><path d="M10 20v-6"/><path d="M6 20v-4"/><path d="M14 20v-4"/>',
		Music: '<path d="M9 18V5l12-2v13"/><circle cx="6" cy="18" r="3"/><circle cx="18" cy="16" r="3"/>',
		ArrowRight: '<line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/>',
		Info: '<circle cx="12" cy="12" r="10"/><line x1="12" y1="16" x2="12" y2="12"/><line x1="12" y1="8" x2="12.01" y2="8"/>',
		Settings: '<path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.1a2 2 0 0 1-1-1.74v-.5a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"/><circle cx="12" cy="12" r="3"/>'
	};

	const formats = {
		bitwig: {
			id: 'bitwig',
			title: 'Bitwig / Dune .WT',
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
			description:
				'A specialized format (originally from Dune) that explicitly declares frame size. Crucial for Bitwig, which defaults raw .WAV files to 256 samples/frame. Using .WT prevents Serum tables from sounding "broken" or pitched down.',
			compatible: ['Bitwig Studio (Polymer/Grid)', 'Synapse Dune 2/3', 'Surge XT']
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
			description:
				'Optimized for hardware with limited memory. Commonly used in Eurorack modules (Synthesis Technology E352/E370) and boutique digital synths.',
			compatible: ['WaveEdit App', 'SynthTech E352/E370', 'Korg modwave (Import)', 'Piston Honda Mk3']
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
		},
		waldorf: {
			id: 'waldorf',
			title: 'Legacy Waldorf',
			subtitle: 'The Classic Standard',
			iconName: 'Database',
			color: 'text-red-400',
			bg: 'bg-red-900/20',
			border: 'border-red-500/50',
			specs: {
				frameSize: '128 Samples (Legacy)',
				maxFrames: '64 Waves',
				bitDepth: '8-bit / 16-bit',
				fileType: '.SYX / Proprietary'
			},
			description:
				'The grandfather of wavetable. Older Blofeld/Microwave units used very small tables. Modern Waldorf synths (Quantum/Iridium) are much more flexible.',
			compatible: ['Waldorf Blofeld', 'Waldorf Microwave', 'PPG Wave']
		}
	};
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
<div class="min-h-screen bg-slate-950 text-slate-100 p-4 md:p-8 font-sans selection:bg-indigo-500 selection:text-white">

	<!-- Header -->
	<header class="mb-12 text-center max-w-4xl mx-auto">
		<div
			class="inline-flex items-center justify-center p-3 bg-indigo-600/20 rounded-full mb-4 border border-indigo-500/30"
		>
			{@render icon('FileAudio', 24, 'text-indigo-400 mr-2')}
			<span class="text-indigo-300 font-medium tracking-wide uppercase text-sm">Audio Engineering Reference</span>
		</div>
		<h1
			class="text-4xl md:text-6xl font-bold mb-4 bg-gradient-to-r from-white via-slate-200 to-slate-400 bg-clip-text text-transparent"
		>
			Wavetable Formats
		</h1>
		<p class="text-lg text-slate-400 max-w-2xl mx-auto">
			A compatibility guide for sound designers. Understanding the hidden structures inside your <code
			class="bg-slate-800 px-2 py-1 rounded text-slate-300">.wav</code
		> files.
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
							class="p-4 rounded-2xl bg-slate-950 border border-slate-800 shadow-xl {formats[selectedFormat]
								.color}"
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
					</div>

					<!-- Compatibility -->
					<div>
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

	<!-- Footer / Visual Comparison -->
	<div class="mt-12 max-w-7xl mx-auto border-t border-slate-800 pt-12">
		<h3 class="text-2xl font-bold text-slate-200 mb-6 text-center">Visualizing the Difference</h3>
		<div class="grid grid-cols-1 md:grid-cols-2 gap-8">
			<!-- Serum Visual -->
			<div class="bg-slate-900 p-6 rounded-xl border border-slate-800">
				<div class="flex justify-between items-end mb-4">
					<span class="text-green-400 font-bold">Serum / Modern</span>
					<span class="text-xs text-slate-500">High Resolution</span>
				</div>
				<div class="h-24 flex items-end justify-between gap-1">
					{#each Array(20) as _, i}
						<div class="w-full bg-green-500/20 rounded-t-sm relative group">
							<div
								class="absolute bottom-0 w-full bg-green-500"
								style="height: {30 + Math.sin(i) * 20 + 40}%"
							></div>
						</div>
					{/each}
				</div>
				<p class="mt-4 text-sm text-slate-400">
					<span class="font-bold text-slate-200">2048 samples</span> per wave allows for extremely smooth sine
					waves and complex harmonics without aliasing.
				</p>
			</div>

			<!-- WaveEdit Visual -->
			<div class="bg-slate-900 p-6 rounded-xl border border-slate-800">
				<div class="flex justify-between items-end mb-4">
					<span class="text-orange-400 font-bold">WaveEdit / Hardware</span>
					<span class="text-xs text-slate-500">Memory Efficient</span>
				</div>
				<div class="h-24 flex items-end justify-between gap-1">
					{#each Array(20) as _, i}
						<div class="w-full bg-orange-500/20 rounded-t-sm relative">
							<!-- Simulating lower resolution/steppier look -->
							<div
								class="absolute bottom-0 w-full bg-orange-500"
								style="height: {Math.round((30 + Math.sin(i) * 20 + 40) / 10) * 10}%"
							></div>
						</div>
					{/each}
				</div>
				<p class="mt-4 text-sm text-slate-400">
					<span class="font-bold text-slate-200">256 samples</span> per wave creates a "steppier" digital sound,
					often desired for "glitch" or "retro" aesthetics in Eurorack.
				</p>
			</div>
		</div>
	</div>
</div>