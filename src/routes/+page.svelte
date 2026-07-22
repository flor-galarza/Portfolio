<script lang="ts">
	// Force reload of all project JSON configurations
	import { onMount } from 'svelte';
	import { fade } from 'svelte/transition';
	import worldMapRaw from '$lib/assets/world-map.svg?raw';

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
		if (target.closest('.card-links, .project-details-expanded')) {
			return;
		}
		toggleExpand(title);
	}

	function handleCardKeydown(e: KeyboardEvent, title: string) {
		if (e.key === 'Enter' || e.key === ' ') {
			const target = e.target as HTMLElement;
			if (target.closest('.card-links, .project-details-expanded, a, button, video')) {
				return;
			}
			e.preventDefault();
			toggleExpand(title);
		}
	}

	let phase = $state('globe');
	let introFinished = $state(false);
	let activeTab = $state('about');
	let avatarError = $state(false);

	let timers: any[] = [];

	function clearTimers() {
		timers.forEach(clearTimeout);
		timers = [];
	}

	function runSequence() {
		clearTimers();
		phase = 'globe';
		timers.push(
			setTimeout(() => {
				phase = 'text';
			}, 1800)
		);
		timers.push(
			setTimeout(() => {
				introFinished = true;
				document.body.classList.add('intro-done');
			}, 7000)
		);
	}

	function skipToEnd() {
		clearTimers();
		phase = 'text';
		introFinished = true;
		document.body.classList.add('intro-done');
	}

	function replayIntro() {
		introFinished = false;
		avatarError = false;
		document.body.classList.remove('intro-done');
		window.scrollTo(0, 0);
		setTimeout(runSequence, 100);
	}

	onMount(() => {
		const starsContainer = document.getElementById('stars');
		if (starsContainer) {
			starsContainer.innerHTML = '';
			for (let i = 0; i < 60; i++) {
				const s = document.createElement('div');
				s.className = 'star';
				const size = Math.random() * 2 + 1;
				s.style.width = size + 'px';
				s.style.height = size + 'px';
				s.style.left = Math.random() * 100 + '%';
				s.style.top = Math.random() * 100 + '%';
				s.style.animationDelay = Math.random() * 3 + 's';
				starsContainer.appendChild(s);
			}
		}
		const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
		if (reduced) {
			skipToEnd();
		} else {
			runSequence();
		}
		return () => clearTimers();
	});

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
</script>

<svelte:head>
	<title>Flor · Portfolio</title>
	<meta
		name="description"
		content="Portfolio of Flor, Systems Engineer. From Argentina to Germany."
	/>
</svelte:head>

<!-- ==================== INTRO OVERLAY ==================== -->
{#if !introFinished}
	<div class="intro-overlay phase-{phase}">
		<div class="stars" id="stars"></div>
		<button class="skip" onclick={skipToEnd}>Skip intro</button>

		<div class="globe-scene">
			<div class="globe-wrap">
				<div class="globe">
					<div class="map-strip-container">
						<div class="map-svg-wrapper">{@html worldMapRaw}</div>
						<div class="map-svg-wrapper">{@html worldMapRaw}</div>
					</div>
					<div class="globe-shade"></div>
				</div>
			</div>
			<div class="greeting">
				Hi! I'm Flor, nice to meet you.<br />Welcome to my site.
			</div>
		</div>
	</div>
{/if}

<!-- ==================== PORTFOLIO CONTENT ==================== -->
<div id="portfolio" class="portfolio">
	<div class="bg-light-blue"></div>
	<div class="bg-light-gold"></div>

	<header class="navbar">
		<div class="logo">Flor<span>.</span></div>
		<nav class="nav-links">
			<button class:active={activeTab === 'about'} onclick={() => (activeTab = 'about')}>
				About
			</button>
			<button class:active={activeTab === 'projects'} onclick={() => (activeTab = 'projects')}>
				Projects
			</button>
			<button class:active={activeTab === 'contact'} onclick={() => (activeTab = 'contact')}>
				Contact
			</button>
		</nav>
	</header>

	<main class="content-container">
		<div class="glass-card">
			{#if activeTab === 'about'}
				<div class="tab-content about-section" transition:fade={{ duration: 300 }}>
					<div class="profile-header">
						<div class="profile-avatar">
							{#if !avatarError}
								<img
									src="/profile.jpg"
									alt="Flor Galarza"
									class="inline-avatar profile-image"
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
						<div class="profile-title">
							<h1>I'm Flor.</h1>
							<p class="subtitle">Systems Engineer</p>
							<p class="location-tag">
								<svg
									xmlns="http://www.w3.org/2000/svg"
									viewBox="0 0 24 24"
									fill="currentColor"
									class="icon"
								>
									<path
										fill-rule="evenodd"
										d="M11.54 22.351l.07.04.028.016a.76.76 0 00.723 0l.028-.015.071-.041a16.975 16.975 0 001.144-.742 19.58 19.58 0 002.683-2.282c1.944-1.99 3.963-4.98 3.963-8.827a8.25 8.25 0 00-16.5 0c0 3.846 2.02 6.837 3.963 8.827a19.58 19.58 0 002.682 2.282 16.975 16.975 0 001.145.742zM12 13.5a3 3 0 100-6 3 3 0 000 6z"
										clip-rule="evenodd"
									/>
								</svg>
								Living in Germany
							</p>
						</div>
					</div>
					<div class="bio-text">
						<p class="bio-lead">
							I'm currently living in <strong>Germany</strong>, embracing a new chapter of my life while building technology that creates a positive impact on people's lives.
						</p>
						<p>
							I grew up believing that technology could solve problems far beyond a computer screen.
						</p>
						<p>
							That curiosity led me to software engineering, research, international competitions, and eventually to opportunities like representing Latin America in the <strong>Huawei ICT Competition</strong> and studying abroad in Germany through the <strong>DAAD scholarship</strong>.
						</p>
						<p>
							Today, I'm passionate about building products where software, artificial intelligence, and healthcare meet. I enjoy transforming ideas into tools that create real value for people.
						</p>
						<p>
							I'm still early in my journey, but I'm excited about tackling ambitious problems, learning every day, and collaborating with people who love building meaningful technology as much as I do.
						</p>
						<p class="bio-footer">
							I'm just getting started.
						</p>
						<div class="tech-pills">
							<span>TypeScript</span>
							<span>Svelte / SvelteKit</span>
							<span>Node.js</span>
							<span>SQL / NoSQL</span>
							<span>Git & CI/CD</span>
							<span>Software Architecture</span>
						</div>
					</div>
				</div>
			{:else if activeTab === 'projects'}
				<div class="tab-content projects-section" transition:fade={{ duration: 300 }}>
					<h2>Things I've made</h2>
					<p class="section-intro">Here is a brief selection of the systems I've worked on:</p>
					<div class="projects-grid">
						{#each projects as project}
							<div
								class="project-card"
								class:expanded={expandedProjectTitle === project.title}
								onclick={(e) => handleCardClick(e, project.title)}
								onkeydown={(e) => handleCardKeydown(e, project.title)}
								role="button"
								tabindex="0"
								aria-expanded={expandedProjectTitle === project.title}
							>
								{#if project.images && project.images.length > 0}
									<button
										class="card-cover-btn"
										onclick={(e) => {
											e.stopPropagation();
											openLightbox(project.images, 0);
										}}
										aria-label="Open cover image in full view"
									>
										<div class="card-cover">
											<img src={project.images[0]} alt="{project.title} Cover" class="cover-img" loading="lazy" />
										</div>
									</button>
								{:else}
									<div class="card-cover fallback-gradient {project.title.toLowerCase().replace(/[^a-z0-9]/g, '-') || ''}">
										<span class="fallback-icon">
											{#if project.title.includes('API')}
												🔌
											{:else if project.title.includes('Insurance')}
												📊
											{:else}
												💻
											{/if}
										</span>
									</div>
								{/if}

								<div class="card-content">
									<h3>{project.title}</h3>
									
									<div class="card-tech">
										{#each project.tech as t}
											<span>{t}</span>
										{/each}
									</div>

									{#if expandedProjectTitle === project.title}
										<div class="project-details-expanded">
											<div class="details-top-grid" class:has-carousel={project.images && project.images.length > 1 && project.layout !== 'stacked'}>
												<div class="details-left">
													<p class="project-description">{project.description}</p>
													
													<div class="card-links">
														{#if project.repoUrl}
															<a
																href={project.repoUrl}
																target="_blank"
																rel="noopener noreferrer"
																class="link-btn github"
															>
																<svg
																	xmlns="http://www.w3.org/2000/svg"
																	viewBox="0 0 24 24"
																	fill="none"
																	stroke="currentColor"
																	stroke-width="2"
																	class="icon"
																>
																	<path
																		stroke-linecap="round"
																		stroke-linejoin="round"
																		d="M14.25 9.75L16.5 12l-2.25 2.25m-4.5 0L7.5 12l2.25-2.25M6 20.25h12A2.25 2.25 0 0020.25 18V6A2.25 2.25 0 0018 3.75H6A2.25 2.25 0 003.75 6v12A2.25 2.25 0 006 20.25z"
																	/>
																</svg>
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
																<svg
																	xmlns="http://www.w3.org/2000/svg"
																	viewBox="0 0 24 24"
																	fill="none"
																	stroke="currentColor"
																	stroke-width="2"
																	class="icon"
																>
																	<path
																		stroke-linecap="round"
																		stroke-linejoin="round"
																		d="M13.19 8.688a4.5 4.5 0 011.242 7.244l-4.5 4.5a4.5 4.5 0 01-6.364-6.364l1.757-1.757m13.35-.622l1.757-1.757a4.5 4.5 0 00-6.364-6.364l-4.5 4.5a4.5 4.5 0 001.242 7.244"
																	/>
																</svg>
																Live Demo
															</a>
														{/if}
													</div>

													{#if project.presentationUrl || project.reportUrl}
														<div class="presentation-download">
															{#if project.presentationUrl}
																<a href={project.presentationUrl} target="_blank" rel="noopener noreferrer" class="download-link">
																	📥 View Presentation (PDF)
																</a>
															{/if}
															{#if project.reportUrl}
																<a href={project.reportUrl} target="_blank" rel="noopener noreferrer" class="download-link">
																	📋 View Release Report (PDF)
																</a>
															{/if}
														</div>
													{/if}
												</div>

												{#if project.images && project.images.length > 1}
													<div class="details-right">
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
																		aria-label="Open screenshot {i + 2} in full view"
																	>
																		<div class="carousel-slide">
																			<img src={img} alt="App Screenshot" class="carousel-img" loading="lazy" />
																		</div>
																	</button>
																{/each}
															</div>
														</div>
													</div>
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
										</div>
									{/if}

									<button class="expand-btn">
										{expandedProjectTitle === project.title ? 'Show less' : 'View details'}
									</button>
								</div>
							</div>
						{/each}
					</div>
				</div>
			{:else if activeTab === 'contact'}
				<div class="tab-content contact-section" transition:fade={{ duration: 300 }}>
					<h2>Let's connect</h2>
					<p class="section-intro">
						Whether you want to collaborate on a project, have a question, or just want to say hi,
						feel free to reach out!
					</p>
					<div class="contact-methods">
						<a href="mailto:hola@florgalarza.dev" class="contact-item">
							<div class="contact-icon">
								<svg
									xmlns="http://www.w3.org/2000/svg"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									stroke-linejoin="round"
								>
									<rect x="2" y="4" width="20" height="16" rx="2" />
									<path d="m22 7-8.97 5.7a1.94 1.94 0 0 1-2.06 0L2 7" />
								</svg>
							</div>
							<div class="contact-details">
								<h4>Email</h4>
								<span>hola@florgalarza.dev</span>
							</div>
						</a>
						<a
							href="https://linkedin.com"
							target="_blank"
							rel="noopener noreferrer"
							class="contact-item"
						>
							<div class="contact-icon">
								<svg
									xmlns="http://www.w3.org/2000/svg"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									stroke-linejoin="round"
								>
									<path
										d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"
									/>
									<rect x="2" y="9" width="4" height="12" />
									<circle cx="4" cy="4" r="2" />
								</svg>
							</div>
							<div class="contact-details">
								<h4>LinkedIn</h4>
								<span>linkedin.com/in/florgalarza</span>
							</div>
						</a>
						<a
							href="https://github.com"
							target="_blank"
							rel="noopener noreferrer"
							class="contact-item"
						>
							<div class="contact-icon">
								<svg
									xmlns="http://www.w3.org/2000/svg"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									stroke-linejoin="round"
								>
									<path
										d="M15 22v-4a4.8 4.8 0 0 0-1-3.5c3 0 6-2 6-5.5.08-1.25-.27-2.48-1-3.5.28-1.15.28-2.35 0-3.5 0 0-1 0-3 1.5-2.64-.5-5.36-.5-8 0C6 2 5 2 5 2c-.3 1.15-.3 2.35 0 3.5A5.403 5.403 0 0 0 4 9c0 3.5 3 5.5 6 5.5-.39.49-.68 1.05-.85 1.65-.17.6-.22 1.23-.15 1.85v4"
									/>
									<path d="M9 18c-4.51 2-5-2-7-2" />
								</svg>
							</div>
							<div class="contact-details">
								<h4>GitHub</h4>
								<span>github.com/florgalarza</span>
							</div>
						</a>
					</div>
				</div>
			{/if}
		</div>
	</main>

	<footer class="footer">
		<button class="replay-btn" onclick={replayIntro}>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				class="icon"
			>
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0013.803-3.7M4.031 9.865a8.25 8.25 0 0113.803-3.7l3.181 3.182m0-4.991v4.99"
				/>
			</svg>
			Watch intro again
		</button>
		<p>&copy; 2026 Flor Galarza &middot; Made with SvelteKit</p>
	</footer>
</div>

<svelte:window onkeydown={handleLightboxKeydown} />

{#if lightboxOpen}
	<!-- Lightbox Backdrop -->
	<div 
		class="lightbox-backdrop" 
		onclick={closeLightbox}
		onkeydown={(e) => {
			if (e.key === 'Enter' || e.key === ' ') {
				closeLightbox();
			}
		}}
		role="button"
		tabindex="0"
		aria-label="Close image viewer"
	>
		<!-- Lightbox Content Container -->
		<div class="lightbox-content" onclick={(e) => e.stopPropagation()}>
			<!-- Close Button -->
			<button class="lightbox-close" onclick={closeLightbox} aria-label="Close image viewer">
				&times;
			</button>

			<!-- Navigation Left Arrow -->
			{#if lightboxImages.length > 1}
				<button class="lightbox-nav prev" onclick={prevImage} aria-label="Previous image">
					&#10094;
				</button>
			{/if}

			<!-- Current Image -->
			<img src={lightboxImages[lightboxIndex]} alt="Enlarged project view" class="lightbox-img" />

			<!-- Navigation Right Arrow -->
			{#if lightboxImages.length > 1}
				<button class="lightbox-nav next" onclick={nextImage} aria-label="Next image">
					&#10095;
				</button>
			{/if}

			<!-- Image Counter Indicator -->
			{#if lightboxImages.length > 1}
				<div class="lightbox-counter">
					{lightboxIndex + 1} / {lightboxImages.length}
				</div>
			{/if}
		</div>
	</div>
{/if}

<style>
	/* ===== INTRO OVERLAY ===== */
	.intro-overlay {
		position: fixed;
		inset: 0;
		z-index: 100;
		background: radial-gradient(ellipse at 50% 30%, var(--bg-deep) 0%, var(--bg) 70%);
		display: flex;
		align-items: center;
		justify-content: center;
		transition: opacity 0.8s ease;
	}

	/* ---------- stars ---------- */
	.stars {
		position: absolute;
		inset: 0;
		overflow: hidden;
		opacity: 0;
		animation: starsIn 1.2s ease forwards;
	}
	@keyframes starsIn {
		to {
			opacity: 1;
		}
	}
	:global(.star) {
		position: absolute;
		background: #fff;
		border-radius: 50%;
		opacity: 0.6;
		animation: twinkle 3.5s ease-in-out infinite;
	}
	@keyframes twinkle {
		0%,
		100% {
			opacity: 0.15;
		}
		50% {
			opacity: 0.8;
		}
	}

	/* ---------- skip ---------- */
	.skip {
		position: absolute;
		top: 22px;
		right: 26px;
		z-index: 150;
		font-family: var(--body);
		font-size: 13px;
		letter-spacing: 0.03em;
		color: var(--ink-dim);
		background: transparent;
		border: 1px solid rgba(255, 255, 255, 0.15);
		border-radius: 999px;
		padding: 7px 16px;
		cursor: pointer;
		transition: all 0.25s ease;
	}
	.skip:hover {
		color: var(--ink);
		border-color: var(--gold-soft);
		background: rgba(255, 255, 255, 0.05);
	}

	/* ---------- globe scene ---------- */
	.globe-scene {
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.globe-wrap {
		position: relative;
		width: min(42vh, 280px);
		height: min(42vh, 280px);
	}
	.globe {
		position: absolute;
		inset: 0;
		border-radius: 50%;
		overflow: hidden;
		box-shadow:
			inset -18px -14px 40px rgba(0, 0, 0, 0.55),
			0 0 60px 6px rgba(27, 58, 92, 0.45);
		background: linear-gradient(160deg, var(--ocean-b), var(--ocean-a));
	}
	.map-strip-container {
		display: flex;
		width: 200%;
		height: 100%;
		animation: spin 9s linear infinite;
	}
	.map-svg-wrapper {
		width: 50%;
		height: 100%;
		display: flex;
		align-items: center;
	}
	.map-svg-wrapper :global(svg) {
		width: 100%;
		height: auto;
		aspect-ratio: 784 / 458;
		display: block;
	}
	.map-svg-wrapper :global(path),
	.map-svg-wrapper :global(g) {
		fill: var(--land);
	}
	@keyframes spin {
		from {
			transform: translateX(0);
		}
		to {
			transform: translateX(-50%);
		}
	}
	.globe-shade {
		position: absolute;
		inset: 0;
		border-radius: 50%;
		pointer-events: none;
		background:
			radial-gradient(circle at 30% 30%, rgba(255, 255, 255, 0.1), transparent 45%),
			radial-gradient(circle at 72% 78%, rgba(0, 0, 0, 0.55), transparent 60%);
	}

	/* ---------- greeting text ---------- */
	.greeting {
		position: absolute;
		inset: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		text-align: center;
		font-family: var(--display);
		font-weight: 600;
		font-size: clamp(16px, 3vw, 22px);
		line-height: 1.5;
		color: var(--ink);
		padding: 30px;
		opacity: 0;
		transform: scale(0.92);
		transition:
			opacity 1s ease,
			transform 1s ease;
		pointer-events: none;
		text-shadow:
			0 2px 20px rgba(0, 0, 0, 0.8),
			0 0 40px rgba(0, 0, 0, 0.5);
	}
	.phase-text .greeting {
		opacity: 1;
		transform: scale(1);
	}

	/* ===== PORTFOLIO ===== */
	.portfolio {
		min-height: 100vh;
		background: var(--bg);
		position: relative;
		padding: 40px 20px;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.bg-light-blue {
		position: fixed;
		top: 15%;
		right: 25%;
		width: min(40vw, 350px);
		height: min(40vw, 350px);
		background: radial-gradient(circle, rgba(14, 32, 51, 0.5) 0%, rgba(3, 5, 9, 0) 70%);
		filter: blur(50px);
		z-index: 0;
		pointer-events: none;
	}
	.bg-light-gold {
		position: fixed;
		bottom: 15%;
		right: 10%;
		width: min(35vw, 300px);
		height: min(35vw, 300px);
		background: radial-gradient(circle, rgba(232, 183, 92, 0.06) 0%, rgba(3, 5, 9, 0) 70%);
		filter: blur(50px);
		z-index: 0;
		pointer-events: none;
	}

	/* ===== NAVBAR ===== */
	.navbar {
		max-width: 700px;
		width: 100%;
		margin: 0 auto 24px auto;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 10px 0;
		position: relative;
		z-index: 5;
	}
	.navbar .logo {
		font-family: var(--display);
		font-weight: 700;
		font-size: 22px;
		letter-spacing: -0.02em;
		color: var(--ink);
	}
	.navbar .logo span {
		color: var(--gold);
	}
	.nav-links {
		display: flex;
		gap: 20px;
	}
	.nav-links button {
		background: transparent;
		border: none;
		color: var(--ink-dim);
		font-family: var(--body);
		font-size: 14px;
		cursor: pointer;
		padding: 6px 12px;
		transition: all 0.25s ease;
		border-bottom: 2px solid transparent;
	}
	.nav-links button:hover {
		color: var(--ink);
	}
	.nav-links button.active {
		color: var(--gold);
		border-bottom-color: var(--gold);
	}

	/* ===== CONTENT ===== */
	.content-container {
		max-width: 700px;
		width: 100%;
		margin: 0 auto;
		flex-grow: 1;
		display: flex;
		align-items: center;
		position: relative;
		z-index: 5;
	}

	.glass-card {
		width: 100%;
		background: rgba(255, 255, 255, 0.02);
		backdrop-filter: blur(20px);
		-webkit-backdrop-filter: blur(20px);
		border: 1px solid rgba(255, 255, 255, 0.06);
		border-radius: 18px;
		padding: 36px;
		box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
		min-height: 350px;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}

	.tab-content {
		width: 100%;
	}

	/* ===== ABOUT ===== */
	.profile-header {
		display: flex;
		align-items: center;
		gap: 24px;
		margin-bottom: 24px;
		border-bottom: 1px solid rgba(255, 255, 255, 0.05);
		padding-bottom: 24px;
	}
	.profile-avatar {
		width: 90px;
		height: 90px;
		border-radius: 50%;
		overflow: hidden;
		background: rgba(255, 255, 255, 0.02);
		border: 1px solid rgba(232, 183, 92, 0.25);
		box-shadow: 0 0 25px var(--gold-glow);
		flex-shrink: 0;
	}
	.inline-avatar {
		width: 100%;
		height: 100%;
		border-radius: 50%;
	}
	.profile-image {
		object-fit: cover;
		border: 2px solid rgba(232, 183, 92, 0.4);
		box-shadow: 0 0 20px var(--gold-glow);
	}
	.profile-title h1 {
		font-family: var(--display);
		font-weight: 700;
		font-size: clamp(24px, 3.5vw, 32px);
		margin: 0 0 4px 0;
		color: var(--ink);
	}
	.profile-title .subtitle {
		font-size: 15px;
		color: var(--gold);
		margin: 0 0 8px 0;
		font-family: var(--display);
		font-weight: 500;
		letter-spacing: 0.05em;
		text-transform: uppercase;
	}
	.location-tag {
		display: inline-flex;
		align-items: center;
		gap: 6px;
		font-size: 12px;
		color: var(--ink-dim);
		background: rgba(255, 255, 255, 0.04);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 99px;
		padding: 4px 12px;
	}
	.location-tag .icon {
		width: 13px;
		height: 13px;
		color: var(--gold);
	}
	.bio-text {
		line-height: 1.7;
		font-size: 15px;
		color: rgba(245, 243, 238, 0.85);
	}
	.bio-text strong {
		color: var(--ink);
		font-weight: 500;
	}
	.bio-text p {
		margin-bottom: 16px;
	}
	.bio-text .bio-lead {
		font-size: 17px;
		line-height: 1.6;
		color: var(--ink);
	}
	.bio-text .bio-footer {
		color: var(--gold);
		font-weight: 500;
		margin-top: 20px;
	}
	.tech-pills {
		display: flex;
		flex-wrap: wrap;
		gap: 8px;
		margin-top: 24px;
	}
	.tech-pills span {
		background: rgba(232, 183, 92, 0.06);
		color: var(--gold);
		border: 1px solid rgba(232, 183, 92, 0.15);
		border-radius: 6px;
		padding: 5px 11px;
		font-size: 12px;
		font-weight: 500;
		transition: all 0.25s ease;
	}
	.tech-pills span:hover {
		background: rgba(232, 183, 92, 0.12);
		border-color: var(--gold);
		transform: translateY(-2px);
		box-shadow: 0 4px 10px var(--gold-glow);
	}

	/* ===== PROJECTS ===== */
	.projects-section h2,
	.contact-section h2 {
		font-family: var(--display);
		font-weight: 700;
		font-size: 26px;
		margin: 0 0 8px 0;
	}
	.section-intro {
		color: var(--ink-dim);
		margin-bottom: 24px;
		font-size: 14px;
	}
	.projects-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
		gap: 20px;
	}
	.project-card {
		background: rgba(255, 255, 255, 0.01);
		border: 1px solid rgba(255, 255, 255, 0.05);
		border-radius: 12px;
		transition: all 0.3s ease;
		cursor: pointer;
		overflow: hidden;
		display: flex;
		flex-direction: column;
		padding: 0;
	}
	.card-cover {
		width: 100%;
		height: 160px;
		overflow: hidden;
		border-bottom: 1px solid rgba(255, 255, 255, 0.05);
		position: relative;
		background: rgba(0, 0, 0, 0.2);
	}
	.cover-img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		transition: transform 0.5s ease;
	}
	.project-card:hover .cover-img {
		transform: scale(1.05);
	}
	.fallback-gradient {
		display: flex;
		align-items: center;
		justify-content: center;
		background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%);
		font-size: 40px;
	}
	.fallback-gradient.api-gateways--microservices {
		background: linear-gradient(135deg, #3b0764 0%, #1e1b4b 100%);
	}
	.fallback-gradient.insurance-analytics---data-warehouse {
		background: linear-gradient(135deg, #064e3b 0%, #022c22 100%);
	}
	.fallback-icon {
		opacity: 0.8;
		filter: drop-shadow(0 0 10px rgba(255,255,255,0.1));
	}
	.card-content {
		padding: 20px;
		display: flex;
		flex-direction: column;
		flex: 1;
		gap: 12px;
	}
	.project-card:focus-visible {
		outline: 2px solid var(--gold);
		outline-offset: 4px;
	}
	.project-card:hover {
		background: rgba(255, 255, 255, 0.03);
		border-color: rgba(232, 183, 92, 0.25);
		transform: translateY(-3px);
		box-shadow:
			0 10px 25px rgba(0, 0, 0, 0.2),
			0 0 15px var(--gold-glow);
	}
	.project-card h3 {
		font-family: var(--display);
		font-size: 16px;
		font-weight: 600;
		margin: 0;
		color: var(--ink);
	}
	.project-description {
		font-size: 13px;
		line-height: 1.5;
		color: var(--ink-dim);
		margin: 0;
	}
	.card-tech {
		display: flex;
		gap: 6px;
		flex-wrap: wrap;
		margin: 0;
	}
	.card-tech span {
		background: rgba(255, 255, 255, 0.04);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 4px;
		padding: 3px 8px;
		font-size: 11px;
		color: var(--ink-dim);
	}
	.card-links {
		display: flex;
		gap: 12px;
		margin-top: 4px;
		border-top: 1px solid rgba(255, 255, 255, 0.04);
		padding-top: 12px;
	}
	.link-btn {
		display: inline-flex;
		align-items: center;
		gap: 6px;
		font-size: 12px;
		color: var(--ink-dim);
		text-decoration: none;
		transition: all 0.25s ease;
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 6px;
		padding: 4px 8px;
		background: rgba(255, 255, 255, 0.02);
	}
	.link-btn:hover {
		color: var(--gold);
		border-color: var(--gold-soft);
		background: rgba(232, 183, 92, 0.05);
	}
	.link-btn :global(svg) {
		width: 13px;
		height: 13px;
		color: var(--gold);
	}
	.expand-btn {
		background: transparent;
		border: 1px dashed var(--gold-soft);
		color: var(--gold);
		font-family: var(--body);
		font-size: 12px;
		border-radius: 4px;
		padding: 4px 8px;
		cursor: pointer;
		margin-top: 4px;
		transition: all 0.25s ease;
		display: inline-flex;
		align-items: center;
		width: max-content;
	}
	.expand-btn:hover {
		background: rgba(232, 183, 92, 0.08);
		border-style: solid;
		border-color: var(--gold);
	}
	.project-card.expanded {
		grid-column: 1 / -1;
	}
	.project-details-expanded {
		margin-top: 8px;
		border-top: 1px dashed rgba(255, 255, 255, 0.08);
		padding-top: 16px;
		width: 100%;
		display: flex;
		flex-direction: column;
		gap: 16px;
	}
	.details-left {
		display: flex;
		flex-direction: column;
		gap: 16px;
	}
	.details-right {
		display: flex;
		flex-direction: column;
		gap: 16px;
	}
	.details-top-grid {
		display: flex;
		flex-direction: column;
		gap: 16px;
		width: 100%;
	}
	@media (min-width: 768px) {
		.details-top-grid.has-carousel {
			display: grid;
			grid-template-columns: 1fr 1fr;
			gap: 32px;
			align-items: start;
		}
	}
	.video-container {
		width: 100%;
		border-radius: 8px;
		overflow: hidden;
		border: 1px solid rgba(255, 255, 255, 0.06);
		box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
		background: #000;
	}
	.project-video {
		width: 100%;
		display: block;
		aspect-ratio: 16 / 9;
	}
	.presentation-download {
		display: flex;
		flex-wrap: wrap;
		gap: 8px;
		margin-top: 5px;
	}
	.download-link {
		display: inline-flex;
		align-items: center;
		gap: 8px;
		font-size: 12px;
		color: var(--ink);
		background: rgba(255, 255, 255, 0.04);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 6px;
		padding: 6px 12px;
		text-decoration: none;
		font-weight: 500;
		transition: all 0.25s ease;
	}
	.download-link:hover {
		background: rgba(232, 183, 92, 0.1);
		border-color: var(--gold);
		color: var(--gold);
		box-shadow: 0 4px 12px var(--gold-glow);
	}

	.carousel-wrap {
		width: 100%;
		display: flex;
		flex-direction: column;
		gap: 10px;
		margin-bottom: 5px;
	}
	.carousel-wrap h4 {
		font-family: var(--display);
		font-size: 13px;
		color: var(--gold);
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
		scrollbar-color: rgba(255,255,255,0.1) transparent;
	}
	.carousel-slide {
		flex: 0 0 160px;
		scroll-snap-align: start;
		border-radius: 8px;
		overflow: hidden;
		border: 1px solid rgba(255, 255, 255, 0.05);
		box-shadow: 0 4px 12px rgba(0,0,0,0.3);
		background: rgba(0,0,0,0.2);
		aspect-ratio: 9 / 19.5;
	}
	.carousel-img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		transition: transform 0.3s ease;
	}
	.carousel-slide:hover .carousel-img {
		transform: scale(1.05);
	}

	/* ===== CONTACT ===== */
	.contact-methods {
		display: flex;
		flex-direction: column;
		gap: 12px;
	}
	.contact-item {
		display: flex;
		align-items: center;
		gap: 16px;
		background: rgba(255, 255, 255, 0.01);
		border: 1px solid rgba(255, 255, 255, 0.04);
		border-radius: 12px;
		padding: 14px 20px;
		text-decoration: none;
		transition: all 0.3s ease;
		color: var(--ink);
	}
	.contact-item:hover {
		background: rgba(255, 255, 255, 0.03);
		border-color: rgba(232, 183, 92, 0.25);
		transform: translateX(4px);
		box-shadow:
			0 4px 15px rgba(0, 0, 0, 0.15),
			0 0 10px var(--gold-glow);
	}
	.contact-icon {
		width: 40px;
		height: 40px;
		display: flex;
		align-items: center;
		justify-content: center;
		background: rgba(232, 183, 92, 0.06);
		border: 1px solid rgba(232, 183, 92, 0.15);
		border-radius: 10px;
		flex-shrink: 0;
	}
	.contact-icon :global(svg) {
		width: 20px;
		height: 20px;
		color: var(--gold);
	}
	.contact-details h4 {
		margin: 0 0 2px 0;
		font-family: var(--display);
		font-weight: 500;
		font-size: 13px;
		color: var(--gold);
	}
	.contact-details span {
		font-size: 14px;
		color: var(--ink-dim);
	}

	/* ===== FOOTER ===== */
	.footer {
		max-width: 700px;
		width: 100%;
		margin: 20px auto 0 auto;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 14px 0;
		border-top: 1px solid rgba(255, 255, 255, 0.05);
		color: var(--ink-dim);
		font-size: 12px;
		position: relative;
		z-index: 5;
	}
	.replay-btn {
		background: transparent;
		border: 1px solid rgba(255, 255, 255, 0.15);
		color: var(--ink-dim);
		font-family: var(--body);
		font-size: 11px;
		border-radius: 99px;
		padding: 5px 12px;
		cursor: pointer;
		display: inline-flex;
		align-items: center;
		gap: 6px;
		transition: all 0.25s ease;
	}
	.replay-btn:hover {
		color: var(--ink);
		border-color: var(--gold-soft);
		background: rgba(255, 255, 255, 0.05);
	}
	.replay-btn :global(.icon) {
		width: 11px;
		height: 11px;
	}

	/* ===== REDUCED MOTION ===== */
	@media (prefers-reduced-motion: reduce) {
		.map-strip-container,
		.star {
			animation: none !important;
		}
	}

	/* ===== RESPONSIVE ===== */
	@media (max-width: 768px) {
		.globe-wrap {
			width: min(32vh, 200px);
			height: min(32vh, 200px);
		}
		.greeting {
			font-size: clamp(14px, 2.5vw, 18px);
			padding: 20px;
		}
		.glass-card {
			padding: 24px;
			min-height: 300px;
		}
		.profile-header {
			flex-direction: column;
			text-align: center;
			gap: 16px;
		}
		.navbar {
			margin-bottom: 16px;
		}
		.nav-links {
			gap: 10px;
		}
		.nav-links button {
			padding: 5px 8px;
			font-size: 13px;
		}
		.footer {
			flex-direction: column;
			gap: 12px;
			text-align: center;
		}
	}

	/* ===== LIGHTBOX BUTTON RESETS ===== */
	.card-cover-btn,
	.carousel-slide-btn {
		background: transparent;
		border: none;
		padding: 0;
		margin: 0;
		cursor: pointer;
		display: block;
		width: 100%;
		text-align: left;
	}

	/* ===== LIGHTBOX MODAL ===== */
	.lightbox-backdrop {
		position: fixed;
		inset: 0;
		z-index: 1000;
		background: rgba(3, 5, 9, 0.85);
		backdrop-filter: blur(8px);
		display: flex;
		align-items: center;
		justify-content: center;
		animation: fadeIn 0.3s ease;
	}
	@keyframes fadeIn {
		from { opacity: 0; }
		to { opacity: 1; }
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
		from { transform: scale(0.95); opacity: 0; }
		to { transform: scale(1); opacity: 1; }
	}
	.lightbox-close {
		position: absolute;
		top: -40px;
		right: 0;
		background: transparent;
		border: none;
		color: var(--ink-dim);
		font-size: 32px;
		cursor: pointer;
		transition: color 0.25s ease;
		padding: 4px;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.lightbox-close:hover {
		color: var(--gold);
	}
	.lightbox-nav {
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		background: rgba(255, 255, 255, 0.03);
		border: 1px solid rgba(255, 255, 255, 0.1);
		backdrop-filter: blur(4px);
		color: var(--ink);
		width: 48px;
		height: 48px;
		border-radius: 50%;
		font-size: 18px;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.25s ease;
		z-index: 1010;
	}
	.lightbox-nav:hover {
		background: rgba(232, 183, 92, 0.1);
		border-color: var(--gold);
		color: var(--gold);
		box-shadow: 0 0 15px var(--gold-glow);
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
		color: var(--ink-dim);
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
			z-index: 1020;
			background: rgba(0, 0, 0, 0.5);
			width: 36px;
			height: 36px;
			border-radius: 50%;
			font-size: 24px;
		}
	}
</style>
