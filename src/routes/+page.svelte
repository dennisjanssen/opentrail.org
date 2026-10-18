<script>
	import { goto } from '$app/navigation';
	import {
		settings,
		TRAILS,
		isInstalled,
		deferredPrompt,
		platform,
		promptInstall
	} from '$lib/store.js';
	import { onMount } from 'svelte';
	import Icon from '$lib/Icon.svelte';
	import WaypointBadge from '$lib/WaypointBadge.svelte';
	let showTrails = $isInstalled || !!$settings.trail;
	let installHelp = false;
	let feature = 0;
	const features = [
		'Easily find points of interest along your route',
		'Your trail, even when you are offline',
		'Fresh trail knowledge from fellow hikers'
	];
	/** @type {Record<string,string>} */
	const names = {
		PCT: 'Pacific Crest Trail',
		AT: 'Appalachian Trail',
		CDT: 'Continental Divide Trail'
	};
	onMount(() => {
		if (window.location.hostname.split('.')[0] === 'test') goto('/app');
	});
	$: if ($isInstalled) showTrails = true;
</script>

<svelte:head
	><title>{showTrails ? 'Choose a trail' : 'Welcome'} &middot; OpenTrail</title></svelte:head
>
<main class:choose-trail={showTrails} class="onboarding">
	<header class="welcome-brand">
		<img src="/brand/logomark.svg" width="88" height="88" alt="" />
		<div>
			<h1>OpenTrail</h1>
			<p>A free trail info community</p>
		</div>
	</header>
	{#if showTrails}
		<section class="trail-choices">
			<h2>Choose your trail</h2>
			<p class="muted">You can change this any time in Settings.</p>
			{#each Object.keys(TRAILS).filter((t) => t !== 'test') as trail}
				{#if !$settings.offline || $settings.trail === trail}
					<a class="trail-choice" href="/app?trail={trail}"
						><span class="trail-emblem">{trail}</span><strong>{names[trail]}</strong><Icon
							name="chevron"
						/></a
					>
				{/if}
			{/each}
			{#if $settings.offline}<p class="muted">
					Turn off offline mode in Settings to switch trails.
				</p>{/if}
		</section>
		<footer>
			Data contributions are open for public use under the <a
				href="https://opendatacommons.org/licenses/odbl/summary/">Open Database License</a
			>.
			<div>
				<a href="https://cdn.opentrail.org/terms.html">Terms</a> ?
				<a href="https://cdn.opentrail.org/privacy.html">Privacy</a>
			</div>
		</footer>
	{:else}
		<section
			class="welcome-feature"
			aria-roledescription="carousel"
			aria-label="OpenTrail features"
		>
			<div class="welcome-map" aria-hidden="true">
				<WaypointBadge kind={feature === 2 ? 'a' : 'c'} size={36} /><WaypointBadge
					kind="w"
					size={36}
				/><WaypointBadge kind="t" size={36} />
			</div>
			<div class="feature-copy">
				<h2 aria-live="polite">{features[feature]}</h2>
				<div class="carousel-controls">
					<button
						class="icon-button"
						aria-label="Previous feature"
						onclick={() => (feature = (feature + 2) % 3)}><Icon name="back" /></button
					>
					<div class="carousel-dots">
						{#each features as _, i}<button
								class:active={feature === i}
								aria-label="Feature {i + 1}"
								aria-pressed={feature === i}
								onclick={() => (feature = i)}
							></button>{/each}
					</div>
					<button
						class="icon-button"
						aria-label="Next feature"
						onclick={() => (feature = (feature + 1) % 3)}><Icon name="chevron" /></button
					>
				</div>
			</div>
		</section>
		<button
			class="btn install-button"
			onclick={() => ($deferredPrompt ? promptInstall() : (installHelp = !installHelp))}
			><Icon name="download" /> Install app</button
		>
		{#if installHelp}<div class="install-help" role="status">
				{#if $platform === 'ios-safari'}Tap <strong>Share</strong>, then
					<strong>Add to Home Screen</strong>, then <strong>Add</strong>.{:else}Open your browser
					menu and choose <strong>Install app</strong> or <strong>Add to Home Screen</strong>. If
					unavailable, try Safari on iOS or Chrome on Android.{/if}
			</div>{/if}
		<button class="skip-install" onclick={() => (showTrails = true)}>Not now</button>
		<p class="install-note">
			You can use OpenTrail without installing, but offline maps and some features need the app. You
			can install it later from Settings.
		</p>
	{/if}
</main>
