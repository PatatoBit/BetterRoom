<script lang="ts">
	type SeatStatus = 'available' | 'occupied' | 'warning' | 'conflict' | 'empty';

	interface Seat {
		id: string;
		status: SeatStatus;
		studentName?: string;
	}

	interface Props {
		seats: Seat[][];
		mini?: boolean;
		interactive?: boolean;
		shape?: 'trapezoid' | 'rect-wide' | 'rect-tall' | 'rect-offset';
	}

	let { seats, mini = false, interactive = false, shape = 'trapezoid' }: Props = $props();

	const STATUS_COLORS: Record<SeatStatus, string> = {
		available: '#5dbb63',
		occupied: '#5dbb63',
		warning: '#ffdb00',
		conflict: '#cc0008',
		empty: '#e0e0e0'
	};

	const SHAPE_POLYGONS = {
		trapezoid: '10,4 90,4 98,24 98,76 84,96 16,96 2,76 2,24',
		'rect-wide': '4,10 96,4 96,90 4,96',
		'rect-tall': '8,4 92,8 88,96 6,92',
		'rect-offset': '4,4 94,6 98,96 8,94'
	} as const;

	const shapePolygon = $derived(SHAPE_POLYGONS[shape]);

	let hoveredSeat = $state<string | null>(null);
</script>

<div class="seating-chart" class:mini>
	<div class="board-label" style={mini ? 'font-size:8px;padding:2px 10px;margin-bottom:6px;' : ''}>
		กระดาน
	</div>
	<div class={`room-outline shape-${shape}`} class:mini-outline={mini}>
		{#if !mini}
			<span class="room-dim left">8m</span>
			<span class="room-dim right">8m</span>
		{/if}
		<svg class="room-walls" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
			<polygon points={shapePolygon} />
		</svg>
		<div class="seats-grid" style={mini ? 'gap:3px;' : ''}>
			{#each seats as row}
				<div class="seat-row" style={mini ? 'gap:3px;' : ''}>
					{#each row as seat}
						{#if interactive}
							<div
								class="seat"
								class:mini
								class:interactive
								style="background:{STATUS_COLORS[seat.status]};"
								role="button"
								aria-label={seat.studentName ?? seat.id}
								tabindex="0"
								onmouseenter={() => {
									hoveredSeat = seat.id;
								}}
								onmouseleave={() => {
									hoveredSeat = null;
								}}
							>
								{#if hoveredSeat === seat.id && seat.studentName}
									<div class="tooltip">{seat.studentName}</div>
								{/if}
							</div>
						{:else}
							<div
								class="seat"
								class:mini
								style="background:{STATUS_COLORS[seat.status]};"
								aria-label={seat.studentName ?? seat.id}
							></div>
						{/if}
					{/each}
				</div>
			{/each}
		</div>
		{#if !mini}
			<div class="legend">
				<span class="legend-dot" style="background:#5dbb63;"></span> Engaged
				<span class="legend-dot" style="background:#ffdb00;"></span> Warning
				<span class="legend-dot" style="background:#cc0008;"></span> Conflict
			</div>
		{/if}
	</div>
</div>

<style>
	.seating-chart {
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 100%;
	}

	.board-label {
		background: var(--ikea-blue);
		color: #fff;
		font-size: 13px;
		font-weight: 700;
		padding: 5px 32px;
		border-radius: 4px;
		margin-bottom: 16px;
		letter-spacing: 0.5px;
	}

	.room-outline {
		padding: 32px 40px;
		position: relative;
		min-width: 280px;
		width: min(680px, 100%);
		min-height: 390px;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 12px;
	}

	.room-outline.mini-outline {
		padding: 8px;
		min-width: 0;
		min-height: 0;
		width: 190px;
		height: 120px;
	}

	.room-outline.shape-trapezoid {
		width: min(680px, 100%);
	}

	.room-outline.shape-rect-wide {
		width: min(720px, 100%);
	}

	.room-outline.shape-rect-tall {
		width: min(620px, 100%);
	}

	.room-outline.shape-rect-offset {
		width: min(660px, 100%);
	}

	.room-outline.mini-outline.shape-trapezoid {
		width: 184px;
		height: 122px;
	}

	.room-outline.mini-outline.shape-rect-wide {
		width: 204px;
		height: 110px;
	}

	.room-outline.mini-outline.shape-rect-tall {
		width: 168px;
		height: 128px;
	}

	.room-outline.mini-outline.shape-rect-offset {
		width: 194px;
		height: 116px;
	}

	.room-walls {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		pointer-events: none;
	}

	.room-walls polygon {
		fill: none;
		stroke: var(--ikea-dark);
		stroke-width: 0.9;
		vector-effect: non-scaling-stroke;
	}

	:global(.mini) .room-walls polygon {
		stroke-width: 1.3;
	}

	.room-dim {
		position: absolute;
		font-size: 13px;
		font-weight: 600;
		color: #444;
	}

	.room-dim.left {
		left: -30px;
		top: 50%;
		transform: translateY(-50%);
	}

	.room-dim.right {
		right: -30px;
		top: 50%;
		transform: translateY(-50%);
	}

	.seats-grid {
		display: flex;
		flex-direction: column;
		gap: 8px;
		align-items: center;
		position: relative;
		z-index: 1;
	}

	.seat-row {
		display: flex;
		gap: 8px;
		align-items: center;
	}

	.seat {
		width: 40px;
		height: 32px;
		border-radius: 5px;
		position: relative;
		flex-shrink: 0;
		transition:
			transform 0.1s,
			filter 0.1s;
	}

	.seat.mini {
		width: 14px;
		height: 11px;
		border-radius: 2px;
	}

	.seat.interactive:hover {
		transform: scale(1.12);
		filter: brightness(1.1);
		cursor: pointer;
	}

	.tooltip {
		position: absolute;
		bottom: calc(100% + 6px);
		left: 50%;
		transform: translateX(-50%);
		background: var(--ikea-dark);
		color: #fff;
		font-size: 11px;
		padding: 3px 8px;
		border-radius: 3px;
		white-space: nowrap;
		pointer-events: none;
		z-index: 10;
	}

	.legend {
		display: flex;
		align-items: center;
		gap: 16px;
		font-size: 12px;
		color: #555;
		margin-top: 8px;
		flex-wrap: wrap;
		justify-content: center;
	}

	.legend-dot {
		display: inline-block;
		width: 10px;
		height: 10px;
		border-radius: 2px;
		margin-right: 4px;
		vertical-align: middle;
	}
</style>
