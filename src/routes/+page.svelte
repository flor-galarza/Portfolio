<script lang="ts">
	import { onMount } from 'svelte';

	const projectFiles = import.meta.glob('../lib/projects/*.json', { eager: true });
	const projects = Object.entries(projectFiles)
		.filter(([path]) => !path.includes('_TEMPLATE'))
		.map(([_, module]: any) => module.default || module)
		.sort((a, b) => (a.order || 99) - (b.order || 99));

	let expandedProjectTitle = $state<string | null>(null);
	function toggleExpand(title: string) {
		expandedProjectTitle = expandedProjectTitle === title ? null : title;
	}

	function handleCardClick(e: Event, title: string) {
		const target = e.target as HTMLElement;
		if (
			target.closest(
				'.card-links, .project-details-expanded, .article-card__banner-btn, .article-card__details'
			)
		) {
			return;
		}
		toggleExpand(title);
	}

	function handleCardKeydown(e: KeyboardEvent, title: string) {
		if (e.key === 'Enter' || e.key === ' ') {
			const target = e.target as HTMLElement;
			if (
				target.closest(
					'.card-links, .project-details-expanded, .article-card__banner-btn, .article-card__details, a, button, video'
				)
			) {
				return;
			}
			e.preventDefault();
			toggleExpand(title);
		}
	}

	let avatarError = $state(false);
	let themeLit = $state(10);

	function setThemeLit(value: number) {
		themeLit = value;
		if (typeof document !== 'undefined') {
			const el = document.querySelector('.presentation-theme') as HTMLElement;
			if (el) el.style.setProperty('--theme-lit', String(value));
			localStorage.setItem('theme-lit', String(value));
		}
	}

	let lightboxOpen = $state(false);
	let lightboxImages = $state<string[]>([]);
	let lightboxIndex = $state(0);

	function openLightbox(images: string[], index: number) {
		lightboxImages = images;
		lightboxIndex = index;
		lightboxOpen = true;
	}

	function closeLightbox() {
		lightboxOpen = false;
	}

	function nextImage() {
		if (lightboxImages.length > 0) {
			lightboxIndex = (lightboxIndex + 1) % lightboxImages.length;
		}
	}

	function prevImage() {
		if (lightboxImages.length > 0) {
			lightboxIndex = (lightboxIndex - 1 + lightboxImages.length) % lightboxImages.length;
		}
	}

	function handleLightboxKeydown(e: KeyboardEvent) {
		if (!lightboxOpen) return;
		if (e.key === 'Escape') {
			closeLightbox();
		} else if (e.key === 'ArrowRight') {
			nextImage();
		} else if (e.key === 'ArrowLeft') {
			prevImage();
		}
	}

	onMount(() => {
		const saved = localStorage.getItem('theme-lit');
		if (saved !== null) {
			themeLit = Number(saved);
		} else {
			const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
			themeLit = prefersDark ? 10 : 80;
		}
		const el = document.querySelector('.presentation-theme') as HTMLElement;
		if (el) el.style.setProperty('--theme-lit', String(themeLit));

		console.log(
			`%cHi! I'm Flor. Welcome to my portfolio.`,
			'color: #e8b75c; font-size: 18px; font-weight: bold; font-family: sans-serif;'
		);
		console.log(
			`%cI'm currently living in Germany, building technology that creates a positive impact on people's lives.\nExplore my projects here, or reach out to me: flor.cursos20@gmail.com`,
			'color: #9ba3ae; font-size: 14px; font-family: sans-serif; line-height: 1.5;'
		);
	});
</script>

<svelte:head>
	<title>Flor · Portfolio</title>
	<meta
		name="description"
		content="Portfolio of Flor, Systems Engineer. From Argentina to Germany."
	/>
</svelte:head>

<div class="presentation-theme">
	<header class="navbar">
		<div class="logo">Flor<span>.</span></div>
		<div class="brightness-control">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				width="16"
				height="16"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
				class="brightness-icon"
			>
				<path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z" />
			</svg>
			<input
				type="range"
				min="5"
				max="90"
				step="1"
				value={themeLit}
				oninput={(e) => setThemeLit(Number((e.target as HTMLInputElement).value))}
				class="brightness-slider"
				aria-label="Brightness"
			/>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				width="16"
				height="16"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
				class="brightness-icon"
			>
				<circle cx="12" cy="12" r="5" /><line x1="12" y1="1" x2="12" y2="3" /><line
					x1="12"
					y1="21"
					x2="12"
					y2="23"
				/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64" /><line
					x1="18.36"
					y1="18.36"
					x2="19.78"
					y2="19.78"
				/><line x1="1" y1="12" x2="3" y2="12" /><line x1="21" y1="12" x2="23" y2="12" /><line
					x1="4.22"
					y1="19.78"
					x2="5.64"
					y2="18.36"
				/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22" />
			</svg>
		</div>
	</header>

	<main class="presentation-main">
		<div class="presentation">
			<div class="presentation__head">
				<div class="presentation__image-container">
					<h2 class="presentation__greeting">Hi</h2>
					<div class="avatar-wrapper">
						{#if !avatarError}
							<img
								src="/profile.jpg"
								alt="Flor"
								class="avatar-img"
								onerror={() => (avatarError = true)}
							/>
						{:else}
							<svg class="figure inline-avatar" viewBox="0 0 200 200">
								<circle class="glow" cx="100" cy="100" r="95" />
								<path
									class="blazer"
									d="M38,178 C40,150 50,132 75,140 C85,150 92,158 100,158 C108,158 115,150 125,140 C150,132 160,150 162,178 L162,200 L38,200 Z"
								/>
								<path
									class="blazer-shade"
									d="M75,140 C85,150 92,158 100,158 L100,200 L60,200 C58,178 63,155 75,140 Z"
									opacity="0.55"
								/>
								<rect class="skin" x="90" y="118" width="20" height="30" rx="7" />
								<ellipse class="skin" cx="100" cy="92" rx="30" ry="34" />
								<path
									class="hair"
									d="M68,78 C64,50 78,28 100,27 C122,28 136,50 132,78 C145,88 150,118 143,148 C139,166 129,172 124,163 C127,136 124,108 118,92 Q100,102 82,92 C76,108 73,136 76,163 C71,172 61,166 57,148 C50,118 55,88 68,78 Z"
								/>
								<path
									class="hair"
									d="M72,68 C78,50 88,40 100,40 C112,40 122,50 128,68 C120,58 110,54 100,54 C90,54 80,58 72,68 Z"
								/>
								<circle class="earring" cx="70" cy="108" r="3" />
								<circle class="earring" cx="130" cy="108" r="3" />
							</svg>
						{/if}
					</div>
				</div>

				<ul class="presentation__list">
					<li class="presentation__list-entry">I build software systems.</li>
					<li class="presentation__list-entry">I do academic research.</li>
					<li class="presentation__list-entry">I work on AI &amp; healthcare products.</li>
					<li class="presentation__list-entry">I am currently living in Germany.</li>
				</ul>
			</div>

			<hr class="presentation__divider" />

			<h2>Find me here:</h2>
			<ul class="presentation__networks">
				<li>
					<a
						class="presentation__networks-link"
						href="https://www.linkedin.com/in/florencia-galarza-maumary-49925131a/"
						target="_blank"
						rel="noopener noreferrer"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							width="24"
							height="24"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
							class="icon"
						>
							<path
								d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"
							/>
							<rect x="2" y="9" width="4" height="12" />
							<circle cx="4" cy="4" r="2" />
						</svg>
						<span class="is--visually-hidden">LinkedIn</span>
					</a>
				</li>
				<li>
					<a
						class="presentation__networks-link"
						href="https://github.com/flor-galarza"
						target="_blank"
						rel="noopener noreferrer"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							width="24"
							height="24"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
							class="icon"
						>
							<path
								d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"
							/>
						</svg>
						<span class="is--visually-hidden">GitHub</span>
					</a>
				</li>
				<li>
					<a
						class="presentation__networks-link"
						href="https://www.instagram.com/florg.maumary/"
						target="_blank"
						rel="noopener noreferrer"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							width="24"
							height="24"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
							class="icon"
						>
							<rect x="2" y="2" width="20" height="20" rx="5" ry="5" />
							<path d="M16 11.37A4 4 0 1 1 12.63 8 4 4 0 0 1 16 11.37z" />
							<line x1="17.5" y1="6.5" x2="17.51" y2="6.5" />
						</svg>
						<span class="is--visually-hidden">Instagram</span>
					</a>
				</li>
				<li>
					<a class="presentation__networks-link" href="mailto:flor.cursos20@gmail.com">
						<svg
							xmlns="http://www.w3.org/2000/svg"
							width="24"
							height="24"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
							class="icon"
						>
							<path
								d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"
							/>
							<polyline points="22,6 12,13 2,6" />
						</svg>
						<span class="is--visually-hidden">Email</span>
					</a>
				</li>
			</ul>

			<p class="love-letter-wrap">
				Or send me a <a class="love-letter-btn" href="mailto:flor.cursos20@gmail.com">love letter</a
				>
			</p>

			<hr class="presentation__divider" />

			<h2>Things I've made:</h2>
			<div class="projects-grid">
				{#each projects as project}
					<div
						class="article__card"
						class:expanded={expandedProjectTitle === project.title}
						onclick={(e) => handleCardClick(e, project.title)}
						onkeydown={(e) => handleCardKeydown(e, project.title)}
						onmousemove={(e) => {
							const card = e.currentTarget;
							const rect = card.getBoundingClientRect();
							const x = e.clientX - rect.left;
							const y = e.clientY - rect.top;
							const xc = rect.width / 2;
							const yc = rect.height / 2;
							const dx = x - xc;
							const dy = y - yc;
							card.style.setProperty('--rx', `${-dy / 25}deg`);
							card.style.setProperty('--ry', `${dx / 25}deg`);
						}}
						onmouseleave={(e) => {
							const card = e.currentTarget;
							card.style.setProperty('--rx', '0deg');
							card.style.setProperty('--ry', '0deg');
						}}
						role="button"
						tabindex="0"
						aria-expanded={expandedProjectTitle === project.title}
					>
						{#if project.images && project.images.length > 0}
							<button
								class="article-card__banner-btn"
								onclick={(e) => {
									e.stopPropagation();
									openLightbox(project.images, 0);
								}}
								aria-label="Open cover image in full view"
							>
								<img
									src={project.images[0]}
									alt={project.title}
									class="article-card__banner"
									loading="lazy"
								/>
							</button>
						{:else}
							<div class="article-card__banner fallback-gradient">
								<span class="fallback-icon">
									{#if project.title.includes('API')}
										&#x1F50C;
									{:else if project.title.includes('Insurance')}
										&#x1F4CA;
									{:else}
										&#x1F4BB;
									{/if}
								</span>
							</div>
						{/if}

						<div class="article-card__content-wrap">
							<aside class="article-card__meta">
								<div class="tech-tags">
									{#each project.tech as t}
										<span>#{t.toLowerCase().replace(/\s+/g, '-')}</span>
									{/each}
								</div>
							</aside>

							<h3 class="article-card__title">
								<button
									class="title-toggle-btn"
									onclick={(e) => {
										e.stopPropagation();
										toggleExpand(project.title);
									}}
								>
									{project.title}
								</button>
							</h3>

							<p class="article-card__summary">{project.description}</p>

							{#if expandedProjectTitle === project.title}
								<div class="article-card__details">
									<div class="card-links">
										{#if project.repoUrl}
											<a
												href={project.repoUrl}
												target="_blank"
												rel="noopener noreferrer"
												class="link-btn github"
											>
												Repository
											</a>
										{/if}
										{#if project.liveUrl}
											<a
												href={project.liveUrl}
												target="_blank"
												rel="noopener noreferrer"
												class="link-btn live"
											>
												Live Demo
											</a>
										{/if}
									</div>

									{#if project.videoUrl}
										<div class="video-container">
											<video
												src={project.videoUrl}
												controls
												class="project-video"
												preload="metadata"
											>
												<track kind="captions" />
												Your browser does not support the video tag.
											</video>
										</div>
									{/if}

									{#if project.presentationUrl || project.reportUrl}
										<div class="presentation-download">
											{#if project.presentationUrl}
												<a
													href={project.presentationUrl}
													target="_blank"
													rel="noopener noreferrer"
													class="download-link"
												>
													View Presentation (PDF)
												</a>
											{/if}
											{#if project.reportUrl}
												<a
													href={project.reportUrl}
													target="_blank"
													rel="noopener noreferrer"
													class="download-link"
												>
													View Release Report (PDF)
												</a>
											{/if}
										</div>
									{/if}

									{#if project.images && project.images.length > 1}
										<div class="carousel-wrap">
											<h4>More screenshots</h4>
											<div class="carousel-track">
												{#each project.images.slice(1) as img, i}
													<button
														class="carousel-slide-btn"
														onclick={(e) => {
															e.stopPropagation();
															openLightbox(project.images, i + 1);
														}}
														aria-label="Open screenshot in full view"
													>
														<img
															src={img}
															alt="App Screenshot"
															class="carousel-img"
															loading="lazy"
														/>
													</button>
												{/each}
											</div>
										</div>
									{/if}
								</div>
							{/if}

							<button
								class="expand-indicator-btn"
								onclick={(e) => {
									e.stopPropagation();
									toggleExpand(project.title);
								}}
							>
								{expandedProjectTitle === project.title ? 'Show less' : 'View details'}
							</button>
						</div>
					</div>
				{/each}
			</div>

			<hr class="presentation__divider" />
			<p class="console-easter-egg">
				Open the JavaScript Console (F12) to play a text adventure game!
			</p>
		</div>
	</main>

	<footer class="footer">
		<p>&copy; 2026 Flor Galarza &middot; Made with SvelteKit</p>
	</footer>
</div>

<svelte:window onkeydown={handleLightboxKeydown} />

{#if lightboxOpen}
	<div
		class="lightbox-backdrop"
		onclick={(e) => {
			if (e.target === e.currentTarget) closeLightbox();
		}}
		onkeydown={(e) => {
			if (e.key === 'Enter' || e.key === ' ') closeLightbox();
		}}
		role="button"
		tabindex="0"
		aria-label="Close image viewer"
	>
		<div class="lightbox-content">
			<button class="lightbox-close" onclick={closeLightbox} aria-label="Close image viewer">
				&times;
			</button>

			{#if lightboxImages.length > 1}
				<button class="lightbox-nav prev" onclick={prevImage} aria-label="Previous image">
					&#10094;
				</button>
			{/if}

			<img src={lightboxImages[lightboxIndex]} alt="Enlarged project view" class="lightbox-img" />

			{#if lightboxImages.length > 1}
				<button class="lightbox-nav next" onclick={nextImage} aria-label="Next image">
					&#10095;
				</button>
			{/if}

			{#if lightboxImages.length > 1}
				<div class="lightbox-counter">
					{lightboxIndex + 1} / {lightboxImages.length}
				</div>
			{/if}
		</div>
	</div>
{/if}

<style>
	/* ===== THEME CORE ===== */
	.presentation-theme {
		--theme-lit: 10;
		--hue: 200;
		--accent-hue: 330;
		--base-saturation: 8%;
		--fg-color-saturation: 25%;

		--base-lightness: calc(var(--theme-lit) + 2%);
		--fg-lightness: calc(100% - var(--theme-lit) - 8%);

		--base-color: hsl(var(--hue), var(--base-saturation), var(--base-lightness));
		--fg-color: hsl(var(--hue), var(--fg-color-saturation), var(--fg-lightness));
		--accent-color: hsl(var(--accent-hue), 100%, 50%);
		--grey: hsl(var(--hue), 12%, calc(50% + (50% - var(--fg-lightness)) * 0.2));
		--border-width: 3px;

		background-color: var(--base-color);
		color: var(--fg-color);
		min-height: 100vh;
		font-family: sans-serif;
		font-size: 1.05rem;
		line-height: 1.6em;
		letter-spacing: 0.018em;
		transition:
			background-color 0.4s ease-out,
			color 0.4s ease-out;
		display: flex;
		flex-direction: column;
		padding: 24px;
	}

	.presentation-main {
		width: 100%;
		max-width: 64rem;
		margin: 0 auto;
		flex: 1;
		padding: 40px 0;
	}

	/* ===== HEADER / NAVBAR ===== */
	.navbar {
		width: 100%;
		max-width: 64rem;
		margin: 0 auto 24px;
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-bottom: var(--border-width) dashed var(--grey);
		padding-bottom: 16px;
	}

	.logo {
		font-family: 'Share', sans-serif;
		font-size: 24px;
		font-weight: 700;
		color: var(--fg-color);
	}
	.logo span {
		color: var(--accent-color);
	}

	/* ===== BRIGHTNESS SLIDER ===== */
	.brightness-control {
		display: flex;
		align-items: center;
		gap: 10px;
	}
	.brightness-icon {
		color: var(--grey);
		flex-shrink: 0;
		transition: color 0.25s ease;
	}
	.brightness-slider {
		-webkit-appearance: none;
		appearance: none;
		width: 100px;
		height: 4px;
		background: var(--grey);
		border-radius: 2px;
		outline: none;
		cursor: pointer;
		transition: background 0.25s ease;
	}
	.brightness-slider::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 16px;
		height: 16px;
		border-radius: 50%;
		background: var(--fg-color);
		border: 2px solid var(--base-color);
		cursor: pointer;
		transition: background 0.25s ease;
	}
	.brightness-slider::-moz-range-thumb {
		width: 16px;
		height: 16px;
		border-radius: 50%;
		background: var(--fg-color);
		border: 2px solid var(--base-color);
		cursor: pointer;
	}
	.brightness-slider:hover {
		background: var(--accent-color);
	}
	.brightness-slider:hover::-webkit-slider-thumb {
		background: var(--accent-color);
	}
	.brightness-slider:hover::-moz-range-thumb {
		background: var(--accent-color);
	}

	/* ===== PRESENTATION GRID ===== */
	.presentation__head {
		display: grid;
		place-items: center;
		grid-template-columns: 1fr;
		gap: 40px;
		margin-bottom: 40px;
		padding: 20px 0;
	}

	@media (min-width: 768px) {
		.presentation__head {
			grid-template-columns: 1fr 1fr;
			min-height: 40vh;
			gap: 64px;
		}
	}

	.presentation__image-container {
		position: relative;
		width: 100%;
		max-width: 22rem;
		margin: auto;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.presentation__image-container:hover .presentation__greeting {
		transform: scale(1.08) rotate(-8deg);
		background: var(--accent-color);
	}

	.presentation__greeting {
		position: absolute;
		left: 0px;
		top: -10px;
		background: var(--fg-color);
		color: var(--base-color);
		font-family: 'Share', sans-serif;
		font-size: 24px;
		font-weight: 700;
		padding: 2px 16px;
		z-index: 10;
		box-shadow: 4px 4px 0 rgba(0, 0, 0, 0.15);
		transition: transform 0.3s ease, background 0.3s ease;
		animation: greetingPop 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
	}
	@keyframes greetingPop {
		from {
			opacity: 0;
			transform: scale(0.6) rotate(-10deg);
		}
		to {
			opacity: 1;
			transform: scale(1) rotate(-5deg);
		}
	}
	.presentation__greeting::after {
		content: '';
		position: absolute;
		right: -6px;
		top: 50%;
		margin-top: -6px;
		display: block;
		width: 12px;
		height: 12px;
		background: var(--fg-color);
		transform: rotate(45deg);
	}

	.avatar-wrapper {
		width: 200px;
		height: 200px;
		border-radius: 50%;
		overflow: hidden;
		border: var(--border-width) solid var(--fg-color);
		box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
		background: radial-gradient(circle, var(--grey) 60%, transparent 60.1%);
		display: flex;
		align-items: center;
		justify-content: center;
		transition: transform 0.4s ease;
	}
	.avatar-wrapper:hover {
		transform: scale(1.03);
	}
	.avatar-img {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}
	.inline-avatar {
		width: 100%;
		height: 100%;
	}
	.figure .glow {
		fill: none;
	}
	.figure .blazer {
		fill: var(--fg-color);
	}
	.figure .blazer-shade {
		fill: var(--base-color);
	}
	.figure .skin {
		fill: var(--grey);
	}
	.figure .hair {
		fill: var(--accent-color);
	}
	.figure .earring {
		fill: var(--fg-color);
	}

	.inline-avatar path,
	.inline-avatar circle,
	.inline-avatar ellipse,
	.inline-avatar rect {
		transition: fill 0.3s ease, stroke 0.3s ease;
	}
	.inline-avatar path:hover,
	.inline-avatar circle:hover,
	.inline-avatar ellipse:hover,
	.inline-avatar rect:hover {
		fill: var(--accent-color) !important;
		stroke: var(--accent-color) !important;
	}

	.presentation__list {
		list-style: none;
		padding: 0;
		margin: 0;
		width: 100%;
		display: flex;
		flex-direction: column;
		gap: 16px;
	}
	.presentation__list-entry {
		font-family: 'Share', sans-serif;
		font-size: 22px;
		font-weight: 700;
		color: var(--fg-color);
		display: flex;
		align-items: center;
		gap: 12px;
		opacity: 0;
		animation: listFadeIn 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
	}
	.presentation__list-entry::before {
		content: '\2192';
		color: var(--accent-color);
		font-weight: 900;
	}
	.presentation__list-entry:nth-child(1) {
		animation-delay: 0.3s;
	}
	.presentation__list-entry:nth-child(2) {
		animation-delay: 0.5s;
	}
	.presentation__list-entry:nth-child(3) {
		animation-delay: 0.7s;
	}
	.presentation__list-entry:nth-child(4) {
		animation-delay: 0.9s;
	}
	@keyframes listFadeIn {
		from {
			opacity: 0;
			transform: translateX(-15px);
		}
		to {
			opacity: 1;
			transform: translateX(0);
		}
	}

	.presentation__divider {
		border: none;
		height: 3px;
		background: repeating-linear-gradient(90deg, var(--grey), var(--grey) 10px, transparent 10px, transparent 20px);
		margin: 40px 0;
		transition: background 0.3s ease;
	}
	.presentation__divider:hover {
		animation: slideDashes 0.8s linear infinite;
		background: repeating-linear-gradient(90deg, var(--accent-color), var(--accent-color) 10px, transparent 10px, transparent 20px);
	}
	@keyframes slideDashes {
		from { background-position: 0 0; }
		to { background-position: 20px 0; }
	}

	h2 {
		font-family: 'Share', sans-serif;
		font-size: 32px;
		font-weight: 700;
		margin-top: 0;
		margin-bottom: 24px;
		color: var(--fg-color);
	}

	/* ===== SOCIAL NETWORKS ===== */
	.presentation__networks {
		display: flex;
		flex-wrap: wrap;
		gap: 24px;
		padding: 0;
		margin: 0 0 24px 0;
		list-style: none;
	}
	.presentation__networks-link {
		color: var(--grey);
		font-size: 2.2rem;
		transition: all 0.25s ease;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		border: var(--border-width) solid var(--grey);
		border-radius: 50%;
		width: 60px;
		height: 60px;
		background: transparent;
	}
	.presentation__networks-link:hover {
		color: var(--accent-color);
		border-color: var(--accent-color);
		background: rgba(255, 255, 255, 0.03);
		transform: translateY(-5px) scale(1.08);
		box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
	}
	.presentation__networks-link :global(.icon) {
		width: 26px;
		height: 26px;
	}

	.love-letter-wrap {
		margin-top: 16px;
	}
	.love-letter-btn {
		display: inline-block;
		padding: 6px 16px;
		background: var(--fg-color);
		color: var(--base-color);
		font-family: 'Share', sans-serif;
		font-size: 16px;
		font-weight: 700;
		text-decoration: none;
		transition: all 0.25s ease;
	}
	.love-letter-btn:hover {
		color: var(--accent-color);
		transform: translateY(-2px);
		box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
	}

	.is--visually-hidden {
		position: absolute !important;
		clip: rect(1px, 1px, 1px, 1px);
		padding: 0 !important;
		border: 0 !important;
		height: 1px !important;
		width: 1px !important;
		overflow: hidden;
	}

	/* ===== PROJECTS ===== */
	.projects-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
		gap: 32px;
	}

	.article__card {
		background: transparent;
		border: var(--border-width) solid var(--grey);
		border-radius: 0px;
		transition: transform 0.15s ease-out, border-color 0.3s ease, box-shadow 0.3s ease;
		display: flex;
		flex-direction: column;
		overflow: hidden;
		position: relative;
		transform: perspective(1000px) rotateX(var(--rx, 0deg)) rotateY(var(--ry, 0deg));
		transform-style: preserve-3d;
	}
	.article__card:hover {
		border-color: var(--fg-color);
		box-shadow: 6px 6px 0 var(--grey);
	}
	.article__card:focus-within {
		outline: 2px solid var(--accent-color);
		outline-offset: 4px;
	}

	.article-card__banner-btn {
		width: 100%;
		height: 180px;
		padding: 0;
		border: none;
		background: transparent;
		cursor: pointer;
		overflow: hidden;
		border-bottom: var(--border-width) dashed var(--grey);
		display: block;
		transform-style: preserve-3d;
	}
	.article-card__banner {
		width: 100%;
		height: 100%;
		object-fit: cover;
		transition: transform 0.15s ease-out;
	}
	.article__card:hover .article-card__banner {
		transform: translateZ(20px) scale(1.04);
	}

	.fallback-gradient {
		display: flex;
		align-items: center;
		justify-content: center;
		background: linear-gradient(135deg, var(--grey) 0%, var(--base-color) 100%);
		font-size: 44px;
		height: 180px;
		border-bottom: var(--border-width) dashed var(--grey);
	}
	.fallback-icon {
		opacity: 0.8;
	}

	.article-card__content-wrap {
		padding: 24px;
		display: flex;
		flex-direction: column;
		flex: 1;
		gap: 12px;
		transition: transform 0.15s ease-out;
		transform-style: preserve-3d;
	}
	.article__card:hover .article-card__content-wrap {
		transform: translateZ(10px);
	}

	.article-card__meta {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.tech-tags {
		display: flex;
		flex-wrap: wrap;
		gap: 8px;
		font-family: monospace;
		font-size: 13px;
		color: var(--accent-color);
	}

	.article-card__title {
		font-family: 'Share', sans-serif;
		font-size: 22px;
		font-weight: 700;
		margin: 0;
		color: var(--fg-color);
	}
	.title-toggle-btn {
		background: transparent;
		border: none;
		padding: 0;
		color: inherit;
		font-family: inherit;
		font-size: inherit;
		font-weight: inherit;
		cursor: pointer;
		text-align: left;
		text-decoration: underline;
		text-decoration-color: transparent;
		transition: text-decoration-color 0.25s ease;
	}
	.title-toggle-btn:hover {
		text-decoration-color: var(--accent-color);
	}

	.article-card__summary {
		font-size: 14px;
		line-height: 1.6em;
		color: var(--grey);
		margin: 0;
		flex: 1;
	}

	.expand-indicator-btn {
		background: transparent;
		border: 1px dashed var(--grey);
		color: var(--fg-color);
		font-family: 'Share', sans-serif;
		font-size: 13px;
		font-weight: 700;
		padding: 4px 10px;
		cursor: pointer;
		width: max-content;
		margin-top: 8px;
		transition: all 0.25s ease;
	}
	.expand-indicator-btn:hover {
		background: var(--fg-color);
		color: var(--base-color);
		border-style: solid;
	}

	.article__card.expanded {
		grid-column: 1 / -1;
	}

	.article-card__details {
		margin-top: 12px;
		border-top: var(--border-width) dashed var(--grey);
		padding-top: 16px;
		display: flex;
		flex-direction: column;
		gap: 20px;
		width: 100%;
	}

	.card-links {
		display: flex;
		flex-wrap: wrap;
		gap: 12px;
	}
	.link-btn {
		display: inline-block;
		padding: 6px 14px;
		font-family: 'Share', sans-serif;
		font-size: 14px;
		font-weight: 700;
		text-decoration: none;
		border: 2px solid var(--fg-color);
		background: transparent;
		color: var(--fg-color);
		transition: all 0.25s ease;
	}
	.link-btn:hover {
		background: var(--fg-color);
		color: var(--base-color);
	}

	.presentation-download {
		display: flex;
		flex-wrap: wrap;
		gap: 12px;
	}
	.download-link {
		display: inline-flex;
		align-items: center;
		gap: 8px;
		font-size: 13px;
		color: var(--fg-color);
		background: transparent;
		border: 1px solid var(--grey);
		padding: 6px 12px;
		text-decoration: none;
		font-weight: 500;
		transition: all 0.25s ease;
	}
	.download-link:hover {
		border-color: var(--accent-color);
		color: var(--accent-color);
	}

	.video-container {
		width: 100%;
		border-radius: 0px;
		overflow: hidden;
		border: var(--border-width) solid var(--fg-color);
		background: #000;
		max-width: 48rem;
		margin-top: 8px;
	}
	.project-video {
		width: 100%;
		display: block;
		aspect-ratio: 16 / 9;
	}

	.carousel-wrap {
		width: 100%;
		display: flex;
		flex-direction: column;
		gap: 10px;
	}
	.carousel-wrap h4 {
		font-family: 'Share', sans-serif;
		font-size: 14px;
		color: var(--accent-color);
		margin: 0;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}
	.carousel-track {
		display: flex;
		gap: 12px;
		overflow-x: auto;
		scroll-snap-type: x mandatory;
		padding-bottom: 8px;
		scrollbar-width: thin;
		scrollbar-color: var(--grey) transparent;
	}
	.carousel-slide-btn {
		background: transparent;
		border: none;
		padding: 0;
		cursor: pointer;
		display: block;
		flex: 0 0 160px;
		scroll-snap-align: start;
		aspect-ratio: 9 / 19.5;
	}
	.carousel-img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		transition: transform 0.3s ease;
		border: var(--border-width) solid var(--grey);
	}
	.carousel-slide-btn:hover .carousel-img {
		transform: scale(1.05);
	}

	.console-easter-egg {
		font-family: monospace;
		font-size: 13px;
		color: var(--grey);
		text-align: center;
		margin-top: 40px;
	}

	/* ===== FOOTER ===== */
	.footer {
		width: 100%;
		max-width: 64rem;
		margin: 40px auto 0;
		border-top: var(--border-width) dashed var(--grey);
		padding-top: 24px;
		display: flex;
		justify-content: center;
		align-items: center;
		color: var(--grey);
		font-size: 14px;
		font-family: 'Share', sans-serif;
	}

	/* ===== LIGHTBOX ===== */
	.lightbox-backdrop {
		position: fixed;
		inset: 0;
		z-index: 3000;
		background: rgba(3, 5, 9, 0.85);
		backdrop-filter: blur(8px);
		display: flex;
		align-items: center;
		justify-content: center;
		animation: fadeIn 0.3s ease;
	}
	@keyframes fadeIn {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}
	.lightbox-content {
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
		max-width: 90%;
		max-height: 90%;
	}
	.lightbox-img {
		max-width: 100%;
		max-height: 85vh;
		border-radius: 8px;
		border: 1px solid rgba(255, 255, 255, 0.1);
		box-shadow: 0 20px 50px rgba(0, 0, 0, 0.7);
		object-fit: contain;
		animation: zoomIn 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
	}
	@keyframes zoomIn {
		from {
			transform: scale(0.95);
			opacity: 0;
		}
		to {
			transform: scale(1);
			opacity: 1;
		}
	}
	.lightbox-close {
		position: absolute;
		top: -40px;
		right: 0;
		background: transparent;
		border: none;
		color: var(--grey);
		font-size: 32px;
		cursor: pointer;
		transition: color 0.25s ease;
		padding: 4px;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.lightbox-close:hover {
		color: var(--accent-color);
	}
	.lightbox-nav {
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		background: rgba(255, 255, 255, 0.03);
		border: 1px solid rgba(255, 255, 255, 0.1);
		backdrop-filter: blur(4px);
		color: var(--fg-color);
		width: 48px;
		height: 48px;
		border-radius: 50%;
		font-size: 18px;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.25s ease;
		z-index: 3010;
	}
	.lightbox-nav:hover {
		background: rgba(232, 183, 92, 0.1);
		border-color: var(--accent-color);
		color: var(--accent-color);
	}
	.lightbox-nav.prev {
		left: -64px;
	}
	.lightbox-nav.next {
		right: -64px;
	}
	.lightbox-counter {
		position: absolute;
		bottom: -32px;
		left: 50%;
		transform: translateX(-50%);
		background: rgba(255, 255, 255, 0.05);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 99px;
		padding: 4px 12px;
		font-size: 12px;
		color: var(--grey);
		font-weight: 500;
	}

	@media (max-width: 992px) {
		.lightbox-nav.prev {
			left: 10px;
		}
		.lightbox-nav.next {
			right: 10px;
		}
		.lightbox-close {
			top: 10px;
			right: 10px;
			z-index: 3020;
			background: rgba(0, 0, 0, 0.5);
			width: 36px;
			height: 36px;
			border-radius: 50%;
			font-size: 24px;
		}
	}

	@media (prefers-reduced-motion: reduce) {
		.presentation__greeting,
		.presentation__list-entry {
			animation: none;
			opacity: 1;
		}
	}
</style>
