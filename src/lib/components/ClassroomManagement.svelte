<script lang="ts">
	import SeatingChart from '$lib/components/SeatingChart.svelte';

	interface Props {
		classId?: string;
	}

	let { classId = '161' }: Props = $props();

	type SeatStatus = 'available' | 'occupied' | 'warning' | 'conflict' | 'empty';
	interface Seat {
		id: string;
		status: SeatStatus;
		studentName?: string;
	}

	const classData: Record<string, { name: string; teacher: string }> = {
		'161': { name: 'ห้อง 161', teacher: 'อาจารย์สมศักดิ์ ชัยประสิทธิ์' },
		'152': { name: 'ห้อง 152', teacher: 'อาจารย์มาลี วงศ์ชัย' },
		'154': { name: 'ห้อง 154', teacher: 'อาจารย์ประเสริฐ บุญศรี' }
	};

	const info = $derived(classData[classId] ?? classData['161']);

	const layoutPatternByClass = {
		'161': 'parliament',
		'152': 'group-clusters',
		'154': 'multiple-straight-row-lines'
	} as const;

	const activeLayoutPattern = $derived(
		layoutPatternByClass[classId as keyof typeof layoutPatternByClass] ?? 'straight-row-lines'
	);

	const seats: Seat[][] = [
		[
			{ id: 'A1', status: 'occupied', studentName: 'Napat K.' },
			{ id: 'A2', status: 'occupied', studentName: 'Jirawat P.' },
			{ id: 'A3', status: 'warning', studentName: 'Somchai L.' },
			{ id: 'A4', status: 'occupied', studentName: 'Pranee S.' },
			{ id: 'A5', status: 'occupied', studentName: 'Wiboon T.' }
		],
		[
			{ id: 'B1', status: 'occupied', studentName: 'Araya M.' },
			{ id: 'B2', status: 'occupied', studentName: 'Korn V.' },
			{ id: 'B3', status: 'occupied', studentName: 'Darika N.' },
			{ id: 'B4', status: 'occupied', studentName: 'Thanat P.' },
			{ id: 'B5', status: 'occupied', studentName: 'Suda R.' },
			{ id: 'B6', status: 'conflict', studentName: 'Chatree W.' }
		],
		[
			{ id: 'C1', status: 'occupied', studentName: 'Pim A.' },
			{ id: 'C2', status: 'occupied', studentName: 'Nat B.' },
			{ id: 'C3', status: 'occupied', studentName: 'Fai C.' },
			{ id: 'C4', status: 'occupied', studentName: 'Joy D.' },
			{ id: 'C5', status: 'occupied', studentName: 'Mint E.' }
		]
	];

	const studentComments = [
		{
			name: 'Napat K.',
			text: 'บทเรียนวันนี้น่าสนใจ แต่จังหวะค่อนข้างเร็วไปนิดค่ะ',
			time: '10:42 AM'
		},
		{ name: 'Pranee S.', text: 'นั่งแถวหลังแล้วได้ยินไม่ค่อยชัดค่ะ', time: '10:51 AM' },
		{
			name: 'Araya M.',
			text: 'อธิบายชัดเจนมาก และสนุกกับกิจกรรมกลุ่มค่ะ',
			time: '11:03 AM'
		}
	];

	const incidents = [
		{
			type: 'คุยกันมากเกินไป',
			students: ['Somchai L.', 'Chatree W.'],
			severity: 'medium',
			time: '10:38 AM'
		},
		{ type: 'ตรวจพบการใช้โทรศัพท์', students: ['Thanat P.'], severity: 'low', time: '11:10 AM' },
		{ type: 'เสียสมาธิซ้ำหลายครั้ง', students: ['Chatree W.'], severity: 'high', time: '11:18 AM' }
	];

	const cleaningScore = 78;
	const moodRating = 4;
	const moodLabel = 'ดี';

	const dailyActivity = [40, 65, 55, 80, 72, 85, 60, 73, 90, 68, 75, 88];
	const hours = ['8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19'];

	const maxActivity = Math.max(...dailyActivity);
	const chartH = 80;
</script>

<div class="management-view">
	<div class="mv-inner">
		<!-- Page header -->
		<div class="mv-top">
			<div>
				<div class="breadcrumb"><a href="/">ภาพรวม</a> / <strong>{info.name}</strong></div>
				<h1 class="mv-title">{info.name}</h1>
				<p class="mv-sub">{info.teacher}</p>
			</div>
			<div class="mv-actions">
				<a href="/setup" class="btn-outline">แก้ไขแผนผัง</a>
				<button class="btn-primary">เริ่มคาบเรียน</button>
			</div>
		</div>

		<div class="mv-columns">
			<!-- ============ LEFT: AI INSIGHT ============ -->
			<aside class="sidebar sidebar-left">
				<div class="sidebar-section">
					<h3 class="sidebar-heading">ดูทั้งหมด</h3>
					<p class="sidebar-sub">เมื่อวาน &nbsp;<strong>วันนี้</strong></p>

					<!-- Mood -->
					<div class="mood-block">
						<div class="mood-face" aria-label="อารมณ์รวม: ดี">
							<svg width="64" height="64" viewBox="0 0 64 64">
								<circle cx="32" cy="32" r="30" fill="none" stroke="#111" stroke-width="2.5" />
								<circle cx="22" cy="26" r="3" fill="#111" />
								<circle cx="42" cy="26" r="3" fill="#111" />
								<path
									d="M20 40 Q32 52 44 40"
									fill="none"
									stroke="#111"
									stroke-width="2.5"
									stroke-linecap="round"
								/>
							</svg>
						</div>
						<div class="mood-label">{moodLabel}</div>
						<div class="star-rating" aria-label="คะแนน {moodRating} จาก 5">
							{#each Array(5) as _, i}
								<svg width="20" height="20" viewBox="0 0 24 24">
									<polygon
										points="12,2 15.09,8.26 22,9.27 17,14.14 18.18,21.02 12,17.77 5.82,21.02 7,14.14 2,9.27 8.91,8.26"
										fill={i < moodRating ? '#ffdb00' : 'none'}
										stroke={i < moodRating ? '#e8a800' : '#ccc'}
										stroke-width="1.5"
									/>
								</svg>
							{/each}
						</div>
					</div>
				</div>

				<!-- AI Summary -->
				<div class="sidebar-section">
					<h3 class="sidebar-heading">สรุป (AI)</h3>
					<div class="ai-summary-box">
						นักเรียนในแถว B-C มีส่วนร่วมสูง พบจุดขัดแย้งบริเวณที่นั่ง B6 แนะนำให้แยกที่นั่งของ
						Chatree W. นักเรียนที่ A3 ดูเสียสมาธิ ควรเข้าไปพูดคุยเพิ่มเติม
					</div>
				</div>

				<!-- Student Comments -->
				<div class="sidebar-section scrollable-comments">
					<h3 class="sidebar-heading">ความเห็นนักเรียน</h3>
					{#each studentComments as comment}
						<div class="comment-card">
							<div class="comment-header">
								<div class="avatar">{comment.name[0]}</div>
								<div>
									<div class="comment-name">{comment.name}</div>
									<div class="comment-time">{comment.time}</div>
								</div>
							</div>
							<p class="comment-text">{comment.text}</p>
						</div>
					{/each}
				</div>
			</aside>

			<!-- ============ CENTER: LIVE FEED + SEATING ============ -->
			<main class="center-col">
				<!-- Live Feed -->
				<div class="live-feed-card">
					<div class="live-badge">
						<span class="live-dot"></span> ถ่ายทอดสด
					</div>
					<div class="live-video-placeholder">
						<svg
							width="48"
							height="48"
							viewBox="0 0 24 24"
							fill="none"
							stroke="#999"
							stroke-width="1.5"
						>
							<path d="M23 7l-7 5 7 5V7z" /><rect
								x="1"
								y="5"
								width="15"
								height="14"
								rx="2"
								ry="2"
							/>
						</svg>
						<span>ภาพจากกล้อง</span>
					</div>
				</div>

				<!-- Seating Chart -->
				<div class="seating-card">
					<h3 class="card-title">ผังที่นั่งแบบโต้ตอบ</h3>
					<div class="seating-wrap">
						<SeatingChart
							{seats}
							interactive={true}
							layoutKey={classId}
							layoutPattern={activeLayoutPattern}
						/>
					</div>
				</div>
			</main>

			<!-- ============ RIGHT: ANALYTICS ============ -->
			<aside class="sidebar sidebar-right">
				<!-- Incidents -->
				<div class="sidebar-section">
					<h3 class="sidebar-heading">เหตุเป็นไปได้</h3>
					<p class="sidebar-sub">เล่นหยอกล้อ? – ตอนนี้</p>
					<div class="incidents-list">
						{#each incidents as inc}
							<div
								class="incident-card"
								class:sev-high={inc.severity === 'high'}
								class:sev-medium={inc.severity === 'medium'}
								class:sev-low={inc.severity === 'low'}
							>
								<div class="incident-header">
									<span class="incident-type">{inc.type}</span>
									<span class="incident-time">{inc.time}</span>
								</div>
								<div class="incident-students">{inc.students.join(', ')}</div>
							</div>
						{/each}
					</div>
				</div>

				<!-- Daily Activity Graph -->
				<div class="sidebar-section">
					<h3 class="sidebar-heading">กราฟแต่ละวัน</h3>
					<div class="mini-chart" aria-label="กราฟกิจกรรมรายวัน">
						<svg
							width="100%"
							height={chartH + 20}
							viewBox="0 0 {hours.length * 18} {chartH + 20}"
							preserveAspectRatio="none"
						>
							{#each dailyActivity as val, i}
								{@const barH = (val / maxActivity) * chartH}
								<rect
									x={i * 18 + 2}
									y={chartH - barH}
									width="14"
									height={barH}
									rx="2"
									fill="var(--ikea-blue)"
									opacity="0.85"
								/>
							{/each}
						</svg>
					</div>
				</div>

				<!-- Cleaning Score -->
				<div class="sidebar-section">
					<h3 class="sidebar-heading">เวรวันนี้</h3>
					<p class="sidebar-sub">คะแนนความสะอาด</p>
					<div class="clean-score-row">
						<div class="clean-score-bar-wrap">
							<div class="clean-score-bar" style="width:{cleaningScore}%;"></div>
						</div>
						<span class="clean-score-num">{cleaningScore}/100</span>
					</div>
				</div>

				<!-- Teacher Report -->
				<div class="sidebar-section">
					<h3 class="sidebar-heading">รายงานจากครูที่มาสอนวันนี้</h3>
					<div class="teacher-report-box">
						ห้องค่อนข้างมีเสียงรบกวนช่วงท้ายคาบ โดยรวมจัดการได้ดี
					</div>
					<div class="teacher-report-meta">ส่งรายงาน 11:30 AM</div>
				</div>
			</aside>
		</div>
	</div>
</div>

<style>
	.management-view {
		padding: 32px 0 80px;
	}

	.mv-inner {
		max-width: 1400px;
		margin: 0 auto;
		padding: 0 32px;
	}

	.mv-top {
		display: flex;
		align-items: flex-end;
		justify-content: space-between;
		margin-bottom: 28px;
	}

	.breadcrumb {
		font-size: 13px;
		color: #666;
		margin-bottom: 6px;
	}

	.breadcrumb a {
		color: var(--ikea-blue);
		font-weight: 600;
	}

	.mv-title {
		font-size: 28px;
		font-weight: 900;
		letter-spacing: -0.4px;
	}

	.mv-sub {
		font-size: 14px;
		color: #555;
		margin-top: 2px;
	}

	.mv-actions {
		display: flex;
		gap: 10px;
		align-items: center;
	}

	.btn-outline {
		background: #fff;
		border: 2px solid var(--ikea-dark);
		color: var(--ikea-dark);
		padding: 10px 20px;
		font-size: 13px;
		font-weight: 700;
		border-radius: 4px;
		cursor: pointer;
		text-decoration: none;
		transition: background 0.15s;
	}

	.btn-outline:hover {
		background: var(--ikea-gray);
	}

	.btn-primary {
		background: var(--ikea-yellow);
		color: var(--ikea-dark);
		border: none;
		padding: 10px 20px;
		font-size: 13px;
		font-weight: 700;
		border-radius: 4px;
		cursor: pointer;
		transition: background 0.15s;
	}

	.btn-primary:hover {
		background: #e8c800;
	}

	.mv-columns {
		display: grid;
		grid-template-columns: 220px 1fr 240px;
		gap: 20px;
		align-items: start;
	}

	/* ---- SIDEBARS ---- */
	.sidebar {
		display: flex;
		flex-direction: column;
		gap: 16px;
	}

	.sidebar-section {
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		padding: 16px;
		background: #fff;
	}

	.sidebar-heading {
		font-size: 13px;
		font-weight: 800;
		text-transform: uppercase;
		letter-spacing: 0.6px;
		margin-bottom: 8px;
		color: var(--ikea-dark);
	}

	.sidebar-sub {
		font-size: 12px;
		color: #777;
		margin-bottom: 10px;
	}

	/* Mood block */
	.mood-block {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 8px;
		padding: 8px 0;
	}

	.mood-face {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.mood-label {
		font-size: 20px;
		font-weight: 900;
	}

	.star-rating {
		display: flex;
		gap: 2px;
	}

	/* AI summary */
	.ai-summary-box {
		font-size: 13px;
		line-height: 1.6;
		color: #333;
		background: var(--ikea-gray);
		padding: 10px;
		border-radius: 3px;
		border-left: 3px solid var(--ikea-blue);
	}

	/* Comments */
	.scrollable-comments {
		max-height: 280px;
		overflow-y: auto;
	}

	.comment-card {
		border: 1px solid var(--ikea-border);
		border-radius: 4px;
		padding: 10px 12px;
		margin-bottom: 8px;
		background: #fafafa;
	}

	.comment-header {
		display: flex;
		align-items: center;
		gap: 8px;
		margin-bottom: 6px;
	}

	.avatar {
		width: 28px;
		height: 28px;
		border-radius: 50%;
		background: var(--ikea-blue);
		color: #fff;
		font-size: 12px;
		font-weight: 700;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
	}

	.comment-name {
		font-size: 12px;
		font-weight: 700;
	}
	.comment-time {
		font-size: 11px;
		color: #888;
	}
	.comment-text {
		font-size: 12px;
		color: #444;
		line-height: 1.5;
	}

	/* ---- CENTER ---- */
	.center-col {
		display: flex;
		flex-direction: column;
		gap: 20px;
	}

	.live-feed-card {
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		overflow: hidden;
	}

	.live-badge {
		display: flex;
		align-items: center;
		gap: 6px;
		padding: 10px 16px;
		font-size: 13px;
		font-weight: 700;
		border-bottom: 1px solid var(--ikea-border);
		background: #fff;
	}

	.live-dot {
		width: 8px;
		height: 8px;
		border-radius: 50%;
		background: var(--ikea-red);
		animation: pulse 1.5s infinite;
	}

	@keyframes pulse {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0.3;
		}
	}

	.live-video-placeholder {
		background: #f0f0f0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 8px;
		height: 180px;
		color: #999;
		font-size: 13px;
		font-weight: 600;
	}

	.seating-card {
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		padding: 24px;
		background: #fff;
	}

	.card-title {
		font-size: 15px;
		font-weight: 800;
		margin-bottom: 20px;
	}

	.seating-wrap {
		display: flex;
		justify-content: center;
		padding: 0 16px;
	}

	/* ---- INCIDENTS ---- */
	.incidents-list {
		display: flex;
		flex-direction: column;
		gap: 8px;
	}

	.incident-card {
		padding: 10px 12px;
		border-radius: 4px;
		border-left: 4px solid transparent;
		background: #fafafa;
		border: 1px solid var(--ikea-border);
	}

	.incident-card.sev-high {
		border-left: 4px solid #cc0008;
		background: #fff5f5;
	}
	.incident-card.sev-medium {
		border-left: 4px solid #ffdb00;
		background: #fffde7;
	}
	.incident-card.sev-low {
		border-left: 4px solid #5dbb63;
		background: #f1f8f2;
	}

	.incident-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 4px;
	}

	.incident-type {
		font-size: 12px;
		font-weight: 700;
	}
	.incident-time {
		font-size: 11px;
		color: #888;
	}
	.incident-students {
		font-size: 11px;
		color: #555;
	}

	/* Mini chart */
	.mini-chart {
		width: 100%;
		overflow: hidden;
	}

	/* Cleaning score */
	.clean-score-row {
		display: flex;
		align-items: center;
		gap: 10px;
		margin-top: 4px;
	}

	.clean-score-bar-wrap {
		flex: 1;
		height: 10px;
		background: var(--ikea-gray);
		border-radius: 5px;
		overflow: hidden;
	}

	.clean-score-bar {
		height: 100%;
		background: var(--ikea-blue);
		border-radius: 5px;
		transition: width 0.5s ease;
	}

	.clean-score-num {
		font-size: 12px;
		font-weight: 700;
		flex-shrink: 0;
	}

	/* Teacher report */
	.teacher-report-box {
		font-size: 13px;
		color: #333;
		background: var(--ikea-gray);
		border-radius: 3px;
		padding: 10px;
		border-left: 3px solid var(--ikea-yellow);
		margin-bottom: 4px;
	}

	.teacher-report-meta {
		font-size: 11px;
		color: #888;
	}

	@media (max-width: 1100px) {
		.mv-columns {
			grid-template-columns: 200px 1fr;
		}
		.sidebar-right {
			display: none;
		}
	}

	@media (max-width: 750px) {
		.mv-columns {
			grid-template-columns: 1fr;
		}
		.sidebar-left {
			display: none;
		}
	}
</style>
