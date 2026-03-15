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
		layoutKey?: string;
		layoutPattern?: LayoutPattern;
	}

	type Point = [number, number];

	interface PositionedSeat {
		seat: Seat;
		x: number;
		y: number;
	}

	type LayoutPattern =
		| 'parliament'
		| 'group-clusters'
		| 'straight-row-lines'
		| 'multiple-straight-row-lines';

	let {
		seats,
		mini = false,
		interactive = false,
		shape = 'trapezoid',
		layoutKey = '',
		layoutPattern
	}: Props = $props();

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

	const shapePoints = $derived.by<Point[]>(() =>
		shapePolygon.split(' ').map((pair) => {
			const [x, y] = pair.split(',').map(Number);
			return [x, y];
		})
	);

	function clamp(value: number, min: number, max: number): number {
		return Math.min(max, Math.max(min, value));
	}

	function hashString(input: string): number {
		let h = 2166136261;
		for (let i = 0; i < input.length; i += 1) {
			h ^= input.charCodeAt(i);
			h += (h << 1) + (h << 4) + (h << 7) + (h << 8) + (h << 24);
		}
		return h >>> 0;
	}

	function isPointInPolygon(point: Point, polygon: Point[]): boolean {
		const [px, py] = point;
		let inside = false;
		for (let i = 0, j = polygon.length - 1; i < polygon.length; j = i++) {
			const [xi, yi] = polygon[i];
			const [xj, yj] = polygon[j];
			const intersect = yi > py !== yj > py && px < ((xj - xi) * (py - yi)) / (yj - yi) + xi;
			if (intersect) {
				inside = !inside;
			}
		}
		return inside;
	}

	function pullInsidePolygon(point: Point, polygon: Point[], target: Point): Point {
		let [x, y] = point;
		if (isPointInPolygon([x, y], polygon)) {
			return [x, y];
		}

		for (let i = 0; i < 18; i += 1) {
			x += (target[0] - x) * 0.18;
			y += (target[1] - y) * 0.18;
			x = clamp(x, 4, 96);
			y = clamp(y, 7, 93);
			if (isPointInPolygon([x, y], polygon)) {
				return [x, y];
			}
		}

		return [target[0], target[1]];
	}

	function relaxSeatPoints(points: Point[], polygon: Point[], isMini: boolean): Point[] {
		const relaxed = points.map(([x, y]) => [x, y] as Point);
		const minHorizontalGap = isMini ? 8.5 : 7.2;
		const minVerticalGap = isMini ? 7.2 : 9.4;
		const center: Point = [50, 54];

		for (let iteration = 0; iteration < 20; iteration += 1) {
			let moved = false;

			for (let i = 0; i < relaxed.length; i += 1) {
				for (let j = i + 1; j < relaxed.length; j += 1) {
					const first = relaxed[i];
					const second = relaxed[j];
					const dx = second[0] - first[0];
					const dy = second[1] - first[1];
					const overlapX = minHorizontalGap - Math.abs(dx);
					const overlapY = minVerticalGap - Math.abs(dy);

					if (overlapX <= 0 || overlapY <= 0) {
						continue;
					}

					moved = true;
					const pushX = overlapX / 2;
					const pushY = overlapY / 2;
					const dirX = dx === 0 ? (i % 2 === 0 ? -1 : 1) : Math.sign(dx);
					const dirY = dy === 0 ? (j % 2 === 0 ? -1 : 1) : Math.sign(dy);

					first[0] -= pushX * dirX;
					second[0] += pushX * dirX;
					first[1] -= pushY * dirY;
					second[1] += pushY * dirY;

					const adjustedFirst = pullInsidePolygon(
						[clamp(first[0], 8, 92), clamp(first[1], 12, 88)],
						polygon,
						center
					);
					const adjustedSecond = pullInsidePolygon(
						[clamp(second[0], 8, 92), clamp(second[1], 12, 88)],
						polygon,
						center
					);

					relaxed[i] = adjustedFirst;
					relaxed[j] = adjustedSecond;
				}
			}

			if (!moved) {
				break;
			}
		}

		return relaxed;
	}

	function flattenSeats(rows: Seat[][]): Seat[] {
		return rows.flatMap((row) => row);
	}

	function choosePattern(rows: Seat[][], allSeats: Seat[]): LayoutPattern {
		if (layoutPattern) {
			return layoutPattern;
		}

		const rowSignature = rows.map((row) => row.length).join('-');
		const signature = `${shape}:${layoutKey || rowSignature}:${allSeats.length}`;
		const hash = hashString(signature);
		const patterns: LayoutPattern[] = [
			'parliament',
			'group-clusters',
			'straight-row-lines',
			'multiple-straight-row-lines'
		];
		return patterns[hash % patterns.length];
	}

	function buildParliamentLayout(count: number): Point[] {
		const points: Point[] = [];
		const ringCount = Math.max(3, Math.min(6, Math.round(Math.sqrt(count) / 1.35)));
		let seatsPlaced = 0;

		for (let ring = 0; ring < ringCount && seatsPlaced < count; ring += 1) {
			const seatsRemaining = count - seatsPlaced;
			const seatsInRing = Math.min(
				seatsRemaining,
				Math.max(4, Math.round((count / ringCount) * (0.82 + ring * 0.2)))
			);
			const yBase = 20 + (ring / Math.max(1, ringCount - 1)) * 58;
			const spread = 26 + ring * 5.5;

			for (let i = 0; i < seatsInRing && seatsPlaced < count; i += 1) {
				const t = seatsInRing <= 1 ? 0.5 : i / (seatsInRing - 1);
				const x = 50 + (t - 0.5) * spread * 2;
				const y = yBase + Math.pow(Math.abs(t - 0.5), 1.45) * (mini ? 6.5 : 10.5);
				points.push([x, y]);
				seatsPlaced += 1;
			}
		}

		return points;
	}

	function buildGroupClustersLayout(count: number): Point[] {
		if (count <= 0) {
			return [];
		}

		const clusterSize = 4;
		const clusterCount = Math.ceil(count / clusterSize);
		const clusterCols = Math.max(2, Math.min(4, Math.ceil(Math.sqrt(clusterCount))));
		const clusterRows = Math.max(1, Math.ceil(clusterCount / clusterCols));
		const points: Point[] = [];
		const localOffsets: Point[] = [
			[-3.8, -3.4],
			[3.8, -3.4],
			[-3.8, 3.4],
			[3.8, 3.4]
		];

		for (let i = 0; i < count; i += 1) {
			const cluster = Math.floor(i / clusterSize);
			const slot = i % clusterSize;
			const cr = Math.floor(cluster / clusterCols);
			const cc = cluster % clusterCols;
			const colNorm = clusterCols <= 1 ? 0.5 : cc / (clusterCols - 1);
			const rowNorm = clusterRows <= 1 ? 0.5 : cr / (clusterRows - 1);
			const centerX = 24 + colNorm * 52;
			const centerY = 26 + rowNorm * 50;
			const [ox, oy] = localOffsets[slot];
			points.push([centerX + ox, centerY + oy]);
		}

		return points;
	}

	function buildStraightRowLinesLayout(rows: Seat[][]): Point[] {
		const points: Point[] = [];
		if (rows.length === 0) {
			return [];
		}

		for (let rowIndex = 0; rowIndex < rows.length; rowIndex += 1) {
			const row = rows[rowIndex];
			const rowNorm = rows.length <= 1 ? 0.5 : rowIndex / (rows.length - 1);
			const y = 22 + rowNorm * 56;
			for (let seatIndex = 0; seatIndex < row.length; seatIndex += 1) {
				const colNorm = row.length <= 1 ? 0.5 : seatIndex / (row.length - 1);
				const x = 18 + colNorm * 64;
				points.push([x, y]);
			}
		}

		return points;
	}

	function buildMultipleStraightRowLinesLayout(rows: Seat[][]): Point[] {
		const points: Point[] = [];
		if (rows.length === 0) {
			return [];
		}

		const rawY: number[] = [];
		for (let rowIndex = 0; rowIndex < rows.length; rowIndex += 1) {
			const aisleBreaksBefore = Math.floor(rowIndex / 2);
			rawY.push(1 + rowIndex * 1.05 + aisleBreaksBefore * 0.48);
		}
		const minRaw = Math.min(...rawY);
		const maxRaw = Math.max(...rawY);

		for (let rowIndex = 0; rowIndex < rows.length; rowIndex += 1) {
			const row = rows[rowIndex];
			const yNorm = maxRaw === minRaw ? 0.5 : (rawY[rowIndex] - minRaw) / (maxRaw - minRaw);
			const y = 22 + yNorm * 56;

			const leftCount = Math.ceil(row.length / 2);
			const rightCount = row.length - leftCount;

			for (let seatIndex = 0; seatIndex < row.length; seatIndex += 1) {
				if (seatIndex < leftCount) {
					const t = leftCount <= 1 ? 0.5 : seatIndex / (leftCount - 1);
					points.push([18 + t * 25, y]);
				} else {
					const rightIndex = seatIndex - leftCount;
					const t = rightCount <= 1 ? 0.5 : rightIndex / (rightCount - 1);
					points.push([57 + t * 25, y]);
				}
			}
		}

		return points;
	}

	function buildPatternPoints(pattern: LayoutPattern, rows: Seat[][], count: number): Point[] {
		switch (pattern) {
			case 'group-clusters':
				return buildGroupClustersLayout(count);
			case 'straight-row-lines':
				return buildStraightRowLinesLayout(rows);
			case 'multiple-straight-row-lines':
				return buildMultipleStraightRowLinesLayout(rows);
			default:
				return buildParliamentLayout(count);
		}
	}

	const positionedSeats = $derived.by<PositionedSeat[]>(() => {
		const allSeats = flattenSeats(seats);
		const pattern = choosePattern(seats, allSeats);
		const basePoints = buildPatternPoints(pattern, seats, allSeats.length);
		const polygon = shapePoints;
		const normalizedPoints = basePoints.map(([x, y]) =>
			pullInsidePolygon([x, y], polygon, [50, 54])
		);
		const relaxedPoints = relaxSeatPoints(normalizedPoints, polygon, mini);
		const items: PositionedSeat[] = [];

		for (let index = 0; index < allSeats.length; index += 1) {
			const seat = allSeats[index];
			const [x, y] = relaxedPoints[index] ?? [50, 54];

			items.push({ seat, x, y });
		}

		return items;
	});

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
		<div class="seat-area">
			{#each positionedSeats as seatPoint}
				{#if interactive}
					<div
						class="seat"
						class:mini
						class:interactive
						style="left:{seatPoint.x}%;top:{seatPoint.y}%;background:{STATUS_COLORS[
							seatPoint.seat.status
						]};"
						role="button"
						aria-label={seatPoint.seat.studentName ?? seatPoint.seat.id}
						tabindex="0"
						onmouseenter={() => {
							hoveredSeat = seatPoint.seat.id;
						}}
						onmouseleave={() => {
							hoveredSeat = null;
						}}
					>
						{#if hoveredSeat === seatPoint.seat.id && seatPoint.seat.studentName}
							<div class="tooltip">{seatPoint.seat.studentName}</div>
						{/if}
					</div>
				{:else}
					<div
						class="seat"
						class:mini
						style="left:{seatPoint.x}%;top:{seatPoint.y}%;background:{STATUS_COLORS[
							seatPoint.seat.status
						]};"
						aria-label={seatPoint.seat.studentName ?? seatPoint.seat.id}
					></div>
				{/if}
			{/each}
		</div>
		{#if !mini}
			<div class="legend">
				<span class="legend-dot" style="background:#5dbb63;"></span> มีส่วนร่วม
				<span class="legend-dot" style="background:#ffdb00;"></span> ควรระวัง
				<span class="legend-dot" style="background:#cc0008;"></span> ขัดแย้ง
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

	.seat-area {
		position: relative;
		width: 100%;
		flex: 1;
		min-height: 250px;
		position: relative;
		z-index: 1;
	}

	.room-outline.mini-outline .seat-area {
		min-height: 0;
	}

	.seat {
		position: absolute;
		width: 40px;
		height: 32px;
		box-sizing: border-box;
		border: 2px solid rgba(17, 17, 17, 0.3);
		border-radius: 5px;
		transform: translate(-50%, -50%);
		flex-shrink: 0;
		box-shadow: 0 2px 6px rgba(17, 17, 17, 0.12);
		transition:
			transform 0.1s,
			filter 0.1s,
			border-color 0.1s;
	}

	.seat.mini {
		width: 14px;
		height: 11px;
		border-width: 1px;
		border-radius: 2px;
	}

	.seat.interactive:hover {
		transform: translate(-50%, -50%) scale(1.08);
		filter: brightness(1.1);
		border-color: rgba(17, 17, 17, 0.55);
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
