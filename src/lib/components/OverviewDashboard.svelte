<script lang="ts">
	import SeatingChart from '$lib/components/SeatingChart.svelte';

	const kpis = [
		{ label: 'จำนวนนักเรียนทั้งหมด', value: '2,440', delta: '+12 สัปดาห์นี้', up: true },
		{ label: 'การแจ้งเตือนที่ยังเปิดอยู่', value: '156', delta: '-8 จากเมื่อวาน', up: false },
		{ label: 'ประสิทธิภาพห้องเรียน', value: '298', delta: '+4%', up: true },
		{ label: 'การแทรกแซงโดย AI', value: '721', delta: '+23 เดือนนี้', up: true }
	];

	type SeatStatus = 'available' | 'occupied' | 'warning' | 'conflict' | 'empty';
	interface Seat {
		id: string;
		status: SeatStatus;
		studentName?: string;
	}

	function makeMiniSeats(rows: SeatStatus[][]): Seat[][] {
		return rows.map((row, ri) => row.map((s, ci) => ({ id: `${ri}-${ci}`, status: s })));
	}

	const classes = [
		{
			id: '161',
			name: 'ห้อง 161',
			teacher: 'Ajarn Somsak',
			students: 28,
			mood: 'ดี',
			layoutPattern: 'parliament' as const,
			shape: 'trapezoid' as const,
			seats: makeMiniSeats([
				['occupied', 'occupied', 'warning', 'occupied', 'occupied'],
				['occupied', 'occupied', 'occupied', 'occupied', 'conflict', 'occupied'],
				['occupied', 'occupied', 'occupied', 'occupied', 'occupied']
			])
		},
		{
			id: '152',
			name: 'ห้อง 152',
			teacher: 'Ajarn Malee',
			students: 31,
			mood: 'ปกติ',
			layoutPattern: 'group-clusters' as const,
			shape: 'rect-wide' as const,
			seats: makeMiniSeats([
				['occupied', 'occupied', 'warning', 'occupied', 'occupied'],
				['occupied', 'occupied', 'occupied', 'conflict', 'occupied', 'occupied'],
				['occupied', 'occupied', 'occupied', 'occupied', 'occupied']
			])
		},
		{
			id: '154',
			name: 'ห้อง 154',
			teacher: 'Ajarn Prasert',
			students: 26,
			mood: 'ดี',
			layoutPattern: 'multiple-straight-row-lines' as const,
			shape: 'rect-offset' as const,
			seats: makeMiniSeats([
				['occupied', 'occupied', 'warning', 'occupied', 'occupied'],
				['occupied', 'occupied', 'occupied', 'occupied', 'conflict', 'occupied'],
				['occupied', 'occupied', 'occupied', 'occupied', 'occupied']
			])
		}
	];
</script>

<section class="overview">
	<div class="page-inner">
		<div class="page-title-row">
			<div>
				<h1 class="page-title">ภาพรวม</h1>
				<p class="page-subtitle">ภาพรวมแบบเรียลไทม์ของทุกห้องเรียนที่ดูแล</p>
			</div>
			<a href="/setup" class="btn-primary" data-tour="overview-add-room">+ เพิ่มห้อง</a>
		</div>

		<!-- KPI Cards -->
		<div class="kpi-row">
			{#each kpis as kpi}
				<div class="kpi-card">
					<span class="kpi-value">{kpi.value}</span>
					<span class="kpi-label">{kpi.label}</span>
					<span class="kpi-delta" class:up={kpi.up} class:down={!kpi.up}>{kpi.delta}</span>
				</div>
			{/each}
		</div>

		<hr class="divider" />

		<!-- Class Cards Grid -->
		<div class="section-header">
			<h2 class="section-title">ห้องเรียนที่ใช้งานอยู่</h2>
			<a href="/classroom/161" class="see-all">ดูทั้งหมด →</a>
		</div>

		<div class="class-grid">
			{#each classes as cls}
				<a href="/classroom/{cls.id}" class="class-card">
					<div class="class-card-chart">
						<SeatingChart
							seats={cls.seats}
							mini={true}
							shape={cls.shape}
							layoutKey={cls.id}
							layoutPattern={cls.layoutPattern}
						/>
					</div>
					<div class="class-card-footer">
						<span class="class-name">{cls.name}</span>
						<div class="class-meta">
							<span class="meta-tag">{cls.students} คน</span>
							<span
								class="meta-mood"
								class:mood-good={cls.mood === 'ดี'}
								class:mood-neutral={cls.mood === 'ปกติ'}>{cls.mood}</span
							>
						</div>
					</div>
				</a>
			{/each}
		</div>
	</div>
</section>

<style>
	.overview {
		padding: 40px 0 80px;
	}

	.page-inner {
		max-width: 1400px;
		margin: 0 auto;
		padding: 0 32px;
	}

	.page-title-row {
		display: flex;
		align-items: flex-end;
		justify-content: space-between;
		margin-bottom: 32px;
	}

	.page-title {
		font-size: 32px;
		font-weight: 900;
		letter-spacing: -0.5px;
		margin-bottom: 4px;
	}

	.page-subtitle {
		font-size: 15px;
		color: #555;
	}

	.btn-primary {
		background: var(--ikea-yellow);
		color: var(--ikea-dark);
		border: none;
		padding: 12px 24px;
		font-weight: 700;
		font-size: 14px;
		border-radius: 4px;
		cursor: pointer;
		text-decoration: none;
		transition: background 0.15s;
		white-space: nowrap;
	}

	.btn-primary:hover {
		background: #e8c800;
	}

	.kpi-row {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		gap: 16px;
		margin-bottom: 40px;
	}

	.kpi-card {
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		padding: 28px 24px;
		display: flex;
		flex-direction: column;
		gap: 6px;
		background: #fff;
		transition: box-shadow 0.15s;
	}

	.kpi-card:hover {
		box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
	}

	.kpi-value {
		font-size: 40px;
		font-weight: 900;
		letter-spacing: -1px;
		line-height: 1;
	}

	.kpi-label {
		font-size: 14px;
		font-weight: 600;
		color: #444;
	}

	.kpi-delta {
		font-size: 13px;
		font-weight: 600;
	}

	.kpi-delta.up {
		color: #2e7d32;
	}
	.kpi-delta.down {
		color: #c62828;
	}

	.divider {
		border: none;
		border-top: 1.5px solid var(--ikea-border);
		margin-bottom: 32px;
	}

	.section-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 20px;
	}

	.section-title {
		font-size: 22px;
		font-weight: 800;
	}

	.see-all {
		font-size: 14px;
		font-weight: 600;
		color: var(--ikea-blue);
		text-decoration: underline;
	}

	.class-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
		gap: 20px;
	}

	.class-card {
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		overflow: hidden;
		text-decoration: none;
		color: inherit;
		background: #fff;
		display: flex;
		flex-direction: column;
		cursor: pointer;
		transition:
			box-shadow 0.15s,
			border-color 0.15s;
	}

	.class-card:hover {
		box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
		border-color: var(--ikea-blue);
	}

	.class-card-chart {
		padding: 24px 24px 16px;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #fafafa;
		min-height: 160px;
	}

	.class-card-footer {
		padding: 14px 20px;
		border-top: 1px solid var(--ikea-border);
		display: flex;
		flex-direction: column;
		gap: 6px;
	}

	.class-name {
		font-size: 16px;
		font-weight: 700;
	}

	.class-meta {
		display: flex;
		gap: 8px;
		align-items: center;
	}

	.meta-tag {
		font-size: 12px;
		background: var(--ikea-gray);
		padding: 2px 8px;
		border-radius: 2px;
		color: #555;
		font-weight: 600;
	}

	.meta-mood {
		font-size: 12px;
		font-weight: 700;
		padding: 2px 8px;
		border-radius: 2px;
	}

	.meta-mood.mood-good {
		background: #e8f5e9;
		color: #2e7d32;
	}
	.meta-mood.mood-neutral {
		background: #fff8e1;
		color: #f57f17;
	}

	@media (max-width: 900px) {
		.kpi-row {
			grid-template-columns: repeat(2, 1fr);
		}
	}
	@media (max-width: 600px) {
		.kpi-row {
			grid-template-columns: 1fr;
		}
		.page-title-row {
			flex-direction: column;
			align-items: flex-start;
			gap: 16px;
		}
	}
</style>
