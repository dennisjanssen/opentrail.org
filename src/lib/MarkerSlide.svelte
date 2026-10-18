<script>
	import Icon from '$lib/Icon.svelte';
	import WaypointBadge from '$lib/WaypointBadge.svelte';
	import { data, settings, trailRoute, userMiles, detailId } from '$lib/store.js';
	import {
		parseDescURL,
		isSafeURL,
		mToFt,
		ftToM,
		formatDist,
		formatElev,
		timeAgo
	} from '$lib/helpers.js';
	export let index;
	export let offset;
	export let onPrev;
	export let onNext;
	export let onClose;

	$: prop = $data.features[index].properties;
	$: imp = $settings.units !== 'metric';
	$: totalMiles = $trailRoute.features?.[0]?.geometry.coordinates.length / 10;
	$: displayMile =
		$settings.reverseMiles && totalMiles != null ? totalMiles - prop.mile : Number(prop.mile);
	$: userCoords = $trailRoute.features?.[0]?.geometry.coordinates;
	$: userIdx = Math.round($userMiles.miles * 10);
	$: userElevFt = userCoords?.[userIdx]?.[2] != null ? mToFt(userCoords[userIdx][2]) : null;
	$: mileDiff = Math.round(Math.abs($userMiles.miles - Number(prop.mile)) * 10) / 10;
	$: mileSign = $settings.reverseMiles
		? $userMiles.miles >= Number(prop.mile)
			? '+'
			: '-'
		: Number(prop.mile) >= $userMiles.miles
			? '+'
			: '-';
	$: elevDiffFt =
		userElevFt != null && prop.elev != null
			? Math.round(Math.abs(userElevFt - Number(prop.elev)))
			: null;
	$: elevSign =
		userElevFt != null && prop.elev != null ? (Number(prop.elev) >= userElevFt ? '+' : '-') : '';
	$: userRecent = new Date() - $userMiles.date < 1000000;
	$: commentCount = prop.comments?.length ?? 0;
	$: latestComment = commentCount > 0 ? prop.comments[0] : null;
</script>

<swiper-slide virtualIndex={index} style={`left: ${offset}px`}>
	<div class="block h-full cursor-pointer" onclick={() => ($detailId = $data.features[index]?.id)}>
		<div
			class="selected-waypoint-card bg-base-100 rounded-lg pt-2 pb-2 p-4 w-full h-full select-text relative overflow-visible flex flex-col"
		>
			{#if onPrev}
				<button
					class="selected-prev btn btn-circle btn-sm bg-base-100 border border-base-300 shadow-md text-base-content absolute z-10"
					aria-label="Previous waypoint"
					style="top: calc(50% - 16px); left: -16px;"
					onclick={(e) => {
						e.stopPropagation();
						onPrev();
					}}
				>
					<Icon name="chevron-left" size={20} />
				</button>
			{/if}
			{#if onNext}
				<button
					class="selected-next btn btn-circle btn-sm bg-base-100 border border-base-300 shadow-md text-base-content absolute z-10"
					aria-label="Next waypoint"
					style="top: calc(50% - 16px); right: -16px;"
					onclick={(e) => {
						e.stopPropagation();
						onNext();
					}}
				>
					<Icon name="chevron-right" size={20} />
				</button>
			{/if}
			<button
				class="selected-close icon-button"
				aria-label="Close selected waypoint"
				onclick={(e) => {
					e.stopPropagation();
					onClose();
				}}><Icon name="close" size={18} /></button
			>
			<div class="flex-1 min-h-0 overflow-hidden">
				{#if prop.images.length > 0}
					<div class="indicator float-right rounded-lg m-1">
						{#if prop.images.length > 1}
							<span class="indicator-item badge badge-neutral p-1.5">
								{prop.images.length}
							</span>
						{/if}
						<img
							src={`https://cdn.opentrail.org/img/${prop.images[0]}.jpg`}
							height="80"
							width="80"
							class="rounded-lg"
							loading="lazy"
						/>
					</div>
				{/if}
				<p class="text-sm font-bold truncate">
					<WaypointBadge kind={prop.icon} size={30} />
					{prop.title}
				</p>
				<p class="text-sm muted">
					{#if imp}Mile {displayMile.toFixed(
							1
						)}{#if userRecent}&nbsp;({mileSign}{mileDiff}){/if}{:else}{formatDist(
							displayMile,
							false,
							1
						)}{#if userRecent}&nbsp;({mileSign}{formatDist(
								mileDiff,
								false,
								1
							)}){/if}{/if}&nbsp;&nbsp;&nbsp;{#if imp}Elev {prop.elev?.toLocaleString(
							'en-US'
						)}'{#if userRecent && elevDiffFt != null}&nbsp;({elevSign}{elevDiffFt}'){/if}{:else}Elev {formatElev(
							prop.elev != null ? ftToM(prop.elev) : null,
							false
						)}{#if userRecent && elevDiffFt != null}&nbsp;({elevSign}{Math.round(
								ftToM(elevDiffFt)
							)}m){/if}{/if}
				</p>
				<p class="text-sm whitespace-pre-wrap break-words">
					{#each parseDescURL(prop.desc) as token}
						{#if token.startsWith('http://') || token.startsWith('https://')}
							{#if isSafeURL(token)}
								<a href={token} class="link" target="_blank" rel="noopener noreferrer">{token}</a>
							{:else}{token}{/if}
						{:else if token.startsWith('www.')}
							<a href={'https://' + token} class="link" target="_blank" rel="noopener noreferrer"
								>{token}</a
							>
						{:else}{token}{/if}
					{/each}
				</p>
			</div>
			<button
				class="btn btn-primary mt-2 w-full"
				onclick={(e) => {
					e.stopPropagation();
					$detailId = $data.features[index]?.id;
				}}>View details</button
			>
			{#if commentCount > 0}
				<div class="flex items-center gap-1.5 text-xs mt-1 shrink-0">
					<Icon name="comment" size={20} /><span>{commentCount}</span>
					<span>{timeAgo(latestComment.date)}</span>
				</div>
			{/if}
		</div>
	</div>
</swiper-slide>
