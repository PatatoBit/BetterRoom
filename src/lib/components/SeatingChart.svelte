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
	}

	let { seats, mini = false, interactive = false }: Props = $props();

	const STATUS_COLORS: Record<SeatStatus, string> = {
		available: '#5dbb63',
		occupied: '#5dbb63',
		warning: '#ffdb00',
		conflict: '#cc0008',
		empty: '#e0e0e0'
	};

	let hoveredSeat = $state<string | null>(null);
</script>

<div class="seating-chart" class:mini>
	<div class="board-label" style={mini ? 'font-size:8px;padding:2px 10px;margin-bottom:6px;' : ''}>
		กระดาน
	</div>
	<div class="room-outline" style={mini ? 'padding:8px;' : ''}>
		{#if !mini}
			<span class="room-dim left">8m</span>
			<span class="room-dim right">8m</span>
		{/if}
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
		border: 2.5px solid var(--ikea-dark);
		border-radius: 48px 48px 40px 40px / 28px 28px 24px 24px;
		clip-path: polygon(6% 0%, 94% 0%, 100% 12%, 100% 88%, 94% 100%, 6% 100%, 0% 88%, 0% 12%);
		padding: 32px 40px;
		position: relative;
		min-width: 280px;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 12px;
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
