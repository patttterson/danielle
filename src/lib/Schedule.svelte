<script lang="ts">
	import { onMount } from 'svelte';

	type Tournament = {
		id: string;
		name: string;
		shortName: string;
		color: string;
		description: string;
	};

	let userTz = Intl.DateTimeFormat().resolvedOptions().timeZone;
	let now = new Date();

	const tournaments: Tournament[] = [
		{
			id: 'taws',
			name: 'TAWS',
			shortName: 'TAWS',
			color: '#a78bfa',
			description: 'The original weekly tournament, double elimination, SS rank and below.'
		},
		{
			id: 'taws-america',
			name: 'TAWS America',
			shortName: 'America',
			color: '#34d399',
			description: 'Americas-friendly timeslot.'
		},
		{
			id: 'taws-u',
			name: 'TAWS U',
			shortName: 'TAWS U',
			color: '#f472b6',
			description: 'U Rank and below.'
		},
		{
			id: 'taws-asia',
			name: 'TAWS Asia',
			shortName: 'Asia',
			color: '#f59e0b',
			description: 'Asia-friendly timeslot.'
		},
		{
			id: 'taws-open',
			name: 'TAWS Open',
			shortName: 'Open',
			color: '#60a5fa',
			description: 'No rank ceiling, open to all players. Schedule varies.'
		}
	];

	const events: Record<string, string[]> = {
		taws: [],
		'taws-asia': [],
		'taws-america': ['2026-03-15T23:00:00Z'],
		'taws-u': [
      '2026-03-14T23:00:00Z'
    ],
		'taws-open': []
	};

	function formatLocal(date: Date, tz: string, includeYear = false): string {
		return date.toLocaleString('en-US', {
			timeZone: tz,
			weekday: 'short',
			month: 'short',
			day: 'numeric',
			...(includeYear ? { year: 'numeric' } : {}),
			hour: 'numeric',
			minute: '2-digit',
			hour12: true
		});
	}

	function isToday(date: Date): boolean {
		const d = new Date(date.toLocaleString('en-US', { timeZone: userTz }));
		const n = new Date(now.toLocaleString('en-US', { timeZone: userTz }));
		return d.getFullYear() === n.getFullYear() && d.getMonth() === n.getMonth() && d.getDate() === n.getDate();
	}

	function relativeLabel(date: Date): string {
		const diff = date.getTime() - now.getTime();
		const days = Math.floor(diff / 86400000);
		const hours = Math.floor(diff / 3600000);
		const mins = Math.floor(diff / 60000);
		if (mins < 60) return `in ${mins}m`;
		if (hours < 24) return `in ${hours}h`;
		if (days === 1) return 'tomorrow';
		if (days < 7) return `in ${days} days`;
		return '';
	}

	type CardData = {
		tournament: Tournament;
		next: Date | undefined;
		upcoming: Date[];
	};

	type EventRow = {
		tournament: Tournament;
		date: Date;
	};

	function buildCards(): CardData[] {
		return tournaments.map((t) => {
			const upcoming = (events[t.id] ?? [])
				.map((s) => new Date(s))
				.filter((d) => d > now)
				.sort((a, b) => a.getTime() - b.getTime())
				.slice(0, 3);
			return { tournament: t, next: upcoming[0], upcoming };
		});
	}

	function buildEventRows(): { upcoming: EventRow[]; completed: EventRow[] } {
		const all: EventRow[] = [];
		for (const t of tournaments) {
			for (const s of events[t.id] ?? []) {
				all.push({ tournament: t, date: new Date(s) });
			}
		}
		const upcoming = all
			.filter((e) => e.date > now)
			.sort((a, b) => a.date.getTime() - b.date.getTime());
		const completed = all
			.filter((e) => e.date <= now)
			.sort((a, b) => b.date.getTime() - a.date.getTime());
		return { upcoming, completed };
	}

	let cards: CardData[] = [];
	let upcomingRows: EventRow[] = [];
	let completedRows: EventRow[] = [];
	let tzDisplay = '';

	onMount(() => {
		now = new Date();
		userTz = Intl.DateTimeFormat().resolvedOptions().timeZone;
		tzDisplay = userTz;
		cards = buildCards();
		const rows = buildEventRows();
		upcomingRows = rows.upcoming;
		completedRows = rows.completed;
	});
</script>

<section class="schedule">
	<div class="section-header">
		<h2 class="section-title">Schedule</h2>
		{#if tzDisplay}
			<span class="tz-badge">
				<svg
					width="12"
					height="12"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2.5"><circle cx="12" cy="12" r="10" /><path d="M12 6v6l4 2" /></svg
				>
				{tzDisplay}
			</span>
		{/if}
	</div>

	<div class="layout">
		<div class="cards-col">
			<div class="cards-grid">
				{#each cards as card}
					<div class="t-card" class:today={card.upcoming.some(isToday)} style="--card-color: {card.tournament.color}">
						<div class="card-accent"></div>
						<div class="card-body">
							<div class="card-header">
								<span class="card-name">{card.tournament.name}</span>
								{#if card.next}
									<span class="card-badge"
										>{relativeLabel(card.next) || formatLocal(card.next, userTz)}</span
									>
								{/if}
							</div>
							<p class="card-desc">{card.tournament.description}</p>
							<div class="card-upcoming">
								{#if card.upcoming.length === 0}
									<span class="no-upcoming">No upcoming events scheduled.</span>
								{:else}
									{#each card.upcoming as occ, i}
										<div class="upcoming-row" class:next={i === 0}>
											<span class="upcoming-dot"></span>
											<span class="upcoming-date">{formatLocal(occ, userTz)}</span>
										</div>
									{/each}
								{/if}
							</div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<div class="event-list-section">
			<div class="list-block">
				<div class="list-heading">Upcoming</div>
				{#if upcomingRows.length === 0}
					<div class="list-empty">No upcoming tournaments scheduled.</div>
				{:else}
					{#each upcomingRows as row}
						<div class="event-row">
							<span class="event-dot" style="background: {row.tournament.color}"></span>
							<span class="event-name" style="color: {row.tournament.color}"
								>{row.tournament.name}</span
							>
							<span class="event-date">{formatLocal(row.date, userTz)}</span>
							{#if relativeLabel(row.date)}
								<span class="event-rel">{relativeLabel(row.date)}</span>
							{/if}
						</div>
					{/each}
				{/if}
			</div>

			<div class="list-block">
				<div class="list-heading">Completed</div>
				{#if completedRows.length === 0}
					<div class="list-empty">No completed tournaments yet.</div>
				{:else}
					{#each completedRows as row}
						<div class="event-row completed">
							<span class="event-dot" style="background: {row.tournament.color}"></span>
							<span class="event-name" style="color: {row.tournament.color}"
								>{row.tournament.name}</span
							>
							<span class="event-date">{formatLocal(row.date, userTz, true)}</span>
						</div>
					{/each}
				{/if}
			</div>
		</div>
	</div>
</section>

<style>
	.schedule {
		width: 100%;
		padding: 24px;
		box-sizing: border-box;
	}

	.cards-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
		gap: 10px;
	}

	.t-card {
		background: rgba(255, 255, 255, 0.12);
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
		border: 1px solid rgba(255, 255, 255, 0.25);
		border-radius: 10px;
		overflow: hidden;
		display: flex;
		box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
		transition:
			border-color 0.15s,
			background 0.15s;
	}

	.t-card:hover {
		background: rgba(255, 255, 255, 0.2);
		border-color: color-mix(in srgb, var(--card-color) 60%, rgba(255, 255, 255, 0.4));
	}

	.t-card.today {
		border-color: color-mix(in srgb, var(--card-color) 70%, transparent);
		box-shadow:
			0 0 10px color-mix(in srgb, var(--card-color) 40%, transparent),
			0 0 24px color-mix(in srgb, var(--card-color) 20%, transparent),
			0 2px 12px rgba(0, 0, 0, 0.15);
	}

	.card-accent {
		width: 4px;
		flex-shrink: 0;
		background: var(--card-color);
		opacity: 0.85;
	}

	.card-body {
		padding: 12px 14px;
		display: flex;
		flex-direction: column;
		gap: 6px;
		flex: 1;
		min-width: 0;
	}

	.card-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 8px;
	}

	.card-name {
		font-family: var(--font-heading);
		font-size: 0.9rem;
		font-weight: 800;
		color: #fff;
		text-transform: uppercase;
		letter-spacing: 0.04em;
	}

	.card-badge {
		font-family: var(--font-heading);
		font-size: 0.62rem;
		font-weight: 700;
		text-transform: uppercase;
		letter-spacing: 0.06em;
		color: var(--card-color);
		background: color-mix(in srgb, var(--card-color) 25%, rgba(0, 0, 0, 0.4));
		backdrop-filter: blur(6px);
		-webkit-backdrop-filter: blur(6px);
		border: 1px solid color-mix(in srgb, var(--card-color) 55%, rgba(255, 255, 255, 0.2));
		border-radius: 4px;
		padding: 2px 7px;
		white-space: nowrap;
		box-shadow: 0 1px 4px rgba(0, 0, 0, 0.12);
	}

	.card-desc {
		font-size: 0.75rem;
		color: rgba(255, 255, 255, 0.7);
		margin: 0;
		line-height: 1.4;
	}

	.card-upcoming {
		display: flex;
		flex-direction: column;
		gap: 3px;
		margin-top: 2px;
	}

	.upcoming-row {
		display: flex;
		align-items: center;
		gap: 7px;
		opacity: 0.5;
	}

	.upcoming-row.next {
		opacity: 1;
	}

	.upcoming-dot {
		width: 5px;
		height: 5px;
		border-radius: 50%;
		background: var(--card-color);
		flex-shrink: 0;
	}

	.upcoming-row:not(.next) .upcoming-dot {
		background: rgba(255, 255, 255, 0.25);
	}

	.upcoming-date {
		font-size: 0.72rem;
		color: rgba(255, 255, 255, 0.9);
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.upcoming-row.next .upcoming-date {
		color: #fff;
		font-weight: 600;
	}

	.no-upcoming {
		font-size: 0.72rem;
		color: rgba(255, 255, 255, 0.25);
		font-style: italic;
	}

	.section-header {
		display: flex;
		align-items: center;
		gap: 12px;
		margin-bottom: 16px;
	}

	.section-title {
		font-family: var(--font-heading);
		font-size: 1.1rem;
		font-weight: 800;
		text-transform: uppercase;
		letter-spacing: 0.1em;
		color: rgba(255, 255, 255, 0.8);
		margin: 0;
	}

	.tz-badge {
		display: flex;
		align-items: center;
		gap: 5px;
		font-family: var(--font-heading);
		font-size: 0.72rem;
		font-weight: 700;
		text-transform: uppercase;
		letter-spacing: 0.08em;
		color: rgba(255, 255, 255, 0.85);
		background: rgba(255, 255, 255, 0.15);
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
		border: 1px solid rgba(255, 255, 255, 0.3);
		border-radius: 4px;
		padding: 4px 8px;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
	}

	.layout {
		display: flex;
		flex-direction: column;
		gap: 28px;
	}

	.cards-col {
		width: 100%;
	}


	/* Event list */

	.event-list-section {
		display: flex;
		flex-direction: column;
		gap: 20px;
	}

	.list-block {
		display: flex;
		flex-direction: column;
	}

	.list-heading {
		font-family: var(--font-heading);
		font-size: 0.7rem;
		font-weight: 800;
		text-transform: uppercase;
		letter-spacing: 0.12em;
		color: rgba(255, 255, 255, 0.55);
		padding-bottom: 8px;
		border-bottom: 1px solid rgba(255, 255, 255, 0.1);
	}

	.list-empty {
		font-size: 0.75rem;
		color: rgba(255, 255, 255, 0.25);
		font-style: italic;
		padding: 10px 0;
	}

	.event-row {
		display: flex;
		align-items: center;
		gap: 10px;
		padding: 9px 4px;
		border-bottom: 1px solid rgba(255, 255, 255, 0.06);
		transition: background 0.1s;
	}

	.event-row:hover {
		background: rgba(255, 255, 255, 0.05);
		border-radius: 6px;
	}

	.event-row.completed {
		opacity: 0.5;
	}

	.event-dot {
		width: 7px;
		height: 7px;
		border-radius: 50%;
		flex-shrink: 0;
	}

	.event-name {
		font-family: var(--font-heading);
		font-size: 0.78rem;
		font-weight: 800;
		text-transform: uppercase;
		letter-spacing: 0.05em;
		white-space: nowrap;
		min-width: 100px;
	}

	.event-date {
		font-size: 0.78rem;
		color: rgba(255, 255, 255, 0.85);
		white-space: nowrap;
		margin-left: auto;
	}

	.event-rel {
		font-family: var(--font-heading);
		font-size: 0.62rem;
		font-weight: 700;
		text-transform: uppercase;
		letter-spacing: 0.06em;
		color: rgba(255, 255, 255, 0.6);
		background: rgba(255, 255, 255, 0.1);
		border: 1px solid rgba(255, 255, 255, 0.18);
		border-radius: 4px;
		padding: 2px 7px;
		white-space: nowrap;
	}

	@media (min-width: 900px) {
		.cards-grid {
			grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
		}

		.event-list-section {
			min-width: 0;
		}
	}

	@media (max-width: 600px) {
		.cards-grid {
			grid-template-columns: 1fr 1fr;
		}

		.event-name {
			min-width: 70px;
		}
	}
</style>
