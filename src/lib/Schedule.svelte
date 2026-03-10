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
      description: 'The original weekly tournament, double elimination, SS rank and below.',
    },
    {
      id: 'taws-asia',
      name: 'TAWS Asia',
      shortName: 'Asia',
      color: '#f59e0b',
      description: 'Asia-friendly timeslot. Alternates fortnightly with TAWS America.',
    },
    {
      id: 'taws-america',
      name: 'TAWS America',
      shortName: 'America',
      color: '#34d399',
      description: 'Americas-friendly timeslot. Alternates fortnightly with TAWS Asia.',
    },
    {
      id: 'taws-u',
      name: 'TAWS U',
      shortName: 'TAWS U',
      color: '#f472b6',
      description: 'U Rank ceiling division. Schedule varies.',
    },
    {
      id: 'taws-open',
      name: 'TAWS Open',
      shortName: 'Open',
      color: '#60a5fa',
      description: 'No rank ceiling, open to all players. Schedule varies.',
    },
  ];

  const events: Record<string, string[]> = {
    'taws': [],
    'taws-asia': [],
    'taws-america': [
      '2026-03-15T23:00:00Z',
    ],
    'taws-u': [
      '2026-03-14T23:00:00Z',
    ],
    'taws-open': [],
  };

  function getUpcoming(id: string, from: Date, count: number): Date[] {
    return (events[id] ?? [])
      .map(s => new Date(s))
      .filter(d => d > from)
      .sort((a, b) => a.getTime() - b.getTime())
      .slice(0, count);
  }

  function formatLocal(date: Date, tz: string): string {
    return date.toLocaleString('en-US', {
      timeZone: tz,
      weekday: 'short',
      month: 'short',
      day: 'numeric',
      hour: 'numeric',
      minute: '2-digit',
      hour12: true,
    });
  }

  function formatTime(date: Date, tz: string): string {
    return date.toLocaleString('en-US', {
      timeZone: tz,
      hour: 'numeric',
      minute: '2-digit',
      hour12: true,
    });
  }

  function formatDayShort(date: Date, tz: string): string {
    return date.toLocaleString('en-US', { timeZone: tz, weekday: 'short' });
  }

  function relativeLabel(date: Date): string {
    const diff = date.getTime() - now.getTime();
    const days = Math.floor(diff / 86400000);
    const hours = Math.floor(diff / 3600000);
    const mins = Math.floor(diff / 60000);
    if (mins < 0) return 'just ended';
    if (mins < 60) return `in ${mins}m`;
    if (hours < 24) return `in ${hours}h`;
    if (days === 1) return 'tomorrow';
    if (days < 7) return `in ${days} days`;
    return formatLocal(date, userTz);
  }

  type GridDay = {
    label: string;
    dateStr: string;
    isToday: boolean;
    events: { tournament: Tournament; time: string }[];
  };

  function buildWeekGrid(offset = 0): GridDay[] {
    const days: GridDay[] = [];

    const anchor = new Date(now);
    anchor.setHours(0, 0, 0, 0);
    const dayOfWeek = anchor.getDay();
    const daysToLastMon = dayOfWeek === 0 ? 6 : dayOfWeek - 1;
    anchor.setDate(anchor.getDate() - daysToLastMon + offset * 7);

    const todayStr = now.toLocaleDateString('en-US', { timeZone: userTz, year: 'numeric', month: '2-digit', day: '2-digit' });

    for (let i = 0; i < 7; i++) {
      const d = new Date(anchor);
      d.setDate(d.getDate() + i);
      const dayEnd = new Date(d);
      dayEnd.setDate(dayEnd.getDate() + 1);

      const dStr = d.toLocaleDateString('en-US', { timeZone: userTz, year: 'numeric', month: '2-digit', day: '2-digit' });
      const isToday = dStr === todayStr;

      const dayEvents: { tournament: Tournament; time: string }[] = [];
      for (const t of tournaments) {
        for (const s of events[t.id] ?? []) {
          const occ = new Date(s);
          if (occ >= d && occ < dayEnd) {
            dayEvents.push({ tournament: t, time: formatTime(occ, userTz) });
          }
        }
      }
      dayEvents.sort((a, b) => a.time.localeCompare(b.time));

      days.push({
        label: formatDayShort(d, userTz),
        dateStr: d.toLocaleDateString('en-US', { timeZone: userTz, month: 'short', day: 'numeric' }),
        isToday,
        events: dayEvents,
      });
    }
    return days;
  }

  type CardData = {
    tournament: Tournament;
    next: Date | undefined;
    upcoming: Date[];
  };

  function buildCards(): CardData[] {
    const withDates = tournaments.map(t => {
      const upcoming = getUpcoming(t.id, now, 3);
      return { tournament: t, next: upcoming[0], upcoming };
    });
    const hasDates = withDates.filter(c => c.next !== undefined).sort((a, b) => a.next!.getTime() - b.next!.getTime());
    const noDates = withDates.filter(c => c.next === undefined);
    return [...hasDates, ...noDates];
  }

  let weekGrid: GridDay[] = [];
  let cards: CardData[] = [];
  let tzDisplay = '';
  let hideEmptyDays = false;
  let weekOffset = 0;

  function shiftWeek(delta: number) {
    weekOffset = Math.max(0, weekOffset + delta);
    weekGrid = buildWeekGrid(weekOffset);
  }

  onMount(() => {
    now = new Date();
    userTz = Intl.DateTimeFormat().resolvedOptions().timeZone;
    tzDisplay = userTz;
    weekGrid = buildWeekGrid(weekOffset);
    cards = buildCards();
  });
</script>

<section class="schedule">
  <div class="section-header">
    <h2 class="section-title">Schedule</h2>
    {#if tzDisplay}
      <span class="tz-badge">
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="12" cy="12" r="10"/><path d="M12 6v6l4 2"/></svg>
        {tzDisplay}
      </span>
    {/if}
    <button class="toggle-btn" class:active={hideEmptyDays} on:click={() => hideEmptyDays = !hideEmptyDays}>
      Hide empty days
    </button>
  </div>

  <div class="week-nav">
    <button class="week-nav-btn" disabled={weekOffset === 0} on:click={() => shiftWeek(-1)}>
      <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="15 18 9 12 15 6"/></svg>
      Prev week
    </button>
    <span class="week-nav-label">{weekOffset === 0 ? 'This week' : `Week +${weekOffset}`}</span>
    <button class="week-nav-btn" on:click={() => shiftWeek(1)}>
      Next week
      <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="9 18 15 12 9 6"/></svg>
    </button>
  </div>

  <div class="week-strip">
    {#each weekGrid.filter(d => !hideEmptyDays || d.events.length > 0) as day}
      <div class="week-day" class:today={day.isToday}>
        <div class="week-day-label">{day.label}</div>
        <div class="week-day-date">{day.dateStr}</div>
        <div class="week-day-events">
          {#if day.events.length === 0}
            <span class="no-events">—</span>
          {:else}
            {#each day.events as ev}
              <div class="week-pill" style="--pill-color: {ev.tournament.color}">
                <span class="pill-name">{ev.tournament.shortName}</span>
                <span class="pill-time">{ev.time}</span>
              </div>
            {/each}
          {/if}
        </div>
      </div>
    {/each}
  </div>

  <div class="cards-grid">
    {#each cards as card}
      <div class="t-card" style="--card-color: {card.tournament.color}">
        <div class="card-accent"></div>
        <div class="card-body">
          <div class="card-header">
            <span class="card-name">{card.tournament.name}</span>
            {#if card.next}
              <span class="card-badge">{relativeLabel(card.next)}</span>
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
</section>

<style>
  .schedule {
    width: 100%;
    padding: 24px;
    box-sizing: border-box;
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
    color: rgba(255,255,255,0.5);
    margin: 0;
  }

  .toggle-btn {
    margin-left: auto;
    font-family: var(--font-heading);
    font-size: 0.68rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.07em;
    color: rgba(255,255,255,0.85);
    background: rgba(255,255,255,0.15);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    border: 1px solid rgba(255,255,255,0.3);
    border-radius: 4px;
    padding: 4px 10px;
    cursor: pointer;
    box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    text-shadow: 0 1px 3px rgba(0,0,0,0.3);
    transition: background 0.15s, color 0.15s, border-color 0.15s;
  }

  .toggle-btn:hover {
    background: rgba(255,255,255,0.25);
    color: #fff;
  }

  .toggle-btn.active {
    color: #a78bfa;
    background: rgba(167,139,250,0.3);
    border-color: rgba(167,139,250,0.6);
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
    color: rgba(255,255,255,0.85);
    background: rgba(255,255,255,0.15);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    border: 1px solid rgba(255,255,255,0.3);
    border-radius: 4px;
    padding: 4px 8px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  }

  .week-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 8px;
  }

  .week-nav-label {
    font-family: var(--font-heading);
    font-size: 0.68rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: rgba(255,255,255,0.35);
  }

  .week-nav-btn {
    display: flex;
    align-items: center;
    gap: 5px;
    font-family: var(--font-heading);
    font-size: 0.68rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.07em;
    color: rgba(255,255,255,0.85);
    background: rgba(255,255,255,0.15);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    border: 1px solid rgba(255,255,255,0.3);
    border-radius: 4px;
    padding: 4px 10px;
    cursor: pointer;
    box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    text-shadow: 0 1px 3px rgba(0,0,0,0.3);
    transition: background 0.15s, color 0.15s;
  }

  .week-nav-btn:hover:not(:disabled) {
    background: rgba(255,255,255,0.25);
    color: #fff;
  }

  .week-nav-btn:disabled {
    opacity: 0.3;
    cursor: default;
  }

  .week-strip {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 6px;
    margin-bottom: 24px;
  }

  .week-day {
    background: rgba(255,255,255,0.12);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    border: 1px solid rgba(255,255,255,0.25);
    border-radius: 8px;
    padding: 10px 8px;
    display: flex;
    flex-direction: column;
    gap: 4px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    transition: background 0.15s;
    min-width: 0;
  }

  .week-day.today {
    background: rgba(167,139,250,0.28);
    border-color: rgba(167,139,250,0.6);
  }

  .week-day-label {
    font-family: var(--font-heading);
    font-size: 0.68rem;
    font-weight: 800;
    text-transform: uppercase;
    letter-spacing: 0.07em;
    color: rgba(255,255,255,0.9);
  }

  .today .week-day-label {
    color: #a78bfa;
  }

  .week-day-date {
    font-family: var(--font-heading);
    font-size: 0.6rem;
    color: rgba(255,255,255,0.35);
    margin-bottom: 4px;
  }

  .week-day-events {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .no-events {
    font-size: 0.7rem;
    color: rgba(255,255,255,0.2);
    text-align: center;
    padding: 4px 0;
  }

  .week-pill {
    background: color-mix(in srgb, var(--pill-color) 25%, rgba(0,0,0,0.4));
    backdrop-filter: blur(6px);
    -webkit-backdrop-filter: blur(6px);
    border: 1px solid color-mix(in srgb, var(--pill-color) 55%, rgba(255,255,255,0.3));
    border-radius: 4px;
    padding: 3px 5px;
    display: flex;
    flex-direction: column;
    gap: 1px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.15);
  }

  .pill-name {
    font-family: var(--font-heading);
    font-size: 0.6rem;
    font-weight: 800;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: var(--pill-color);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .pill-time {
    font-size: 0.58rem;
    color: rgba(255,255,255,0.55);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .cards-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
    gap: 10px;
  }

  .t-card {
    background: rgba(255,255,255,0.12);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    border: 1px solid rgba(255,255,255,0.25);
    border-radius: 10px;
    overflow: hidden;
    display: flex;
    box-shadow: 0 2px 12px rgba(0,0,0,0.15);
    transition: border-color 0.15s, background 0.15s;
  }

  .t-card:hover {
    background: rgba(255,255,255,0.2);
    border-color: color-mix(in srgb, var(--card-color) 60%, rgba(255,255,255,0.4));
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
    background: color-mix(in srgb, var(--card-color) 25%, rgba(0,0,0,0.4));
    backdrop-filter: blur(6px);
    -webkit-backdrop-filter: blur(6px);
    border: 1px solid color-mix(in srgb, var(--card-color) 55%, rgba(255,255,255,0.2));
    border-radius: 4px;
    padding: 2px 7px;
    white-space: nowrap;
    box-shadow: 0 1px 4px rgba(0,0,0,0.12);
  }

  .card-desc {
    font-size: 0.75rem;
    color: rgba(255,255,255,0.45);
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
    background: rgba(255,255,255,0.25);
  }

  .upcoming-date {
    font-size: 0.72rem;
    color: rgba(255,255,255,0.75);
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
    color: rgba(255,255,255,0.25);
    font-style: italic;
  }

  @media (max-width: 600px) {
    .week-strip {
      grid-template-columns: repeat(7, 1fr);
      gap: 4px;
    }

    .week-day {
      padding: 7px 4px;
    }

    .week-day-date {
      display: none;
    }

    .cards-grid {
      grid-template-columns: 1fr 1fr;
    }
  }
</style>