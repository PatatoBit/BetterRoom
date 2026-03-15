<script lang="ts">
	import SeatingChart from '$lib/components/SeatingChart.svelte';

	type Step = 1 | 2 | 3;
	let currentStep = $state<Step>(1);

	// Step 1: Room Input
	const inputMethods = [
		{ id: 'camera', label: 'Select Camera Input', icon: 'camera' },
		{ id: 'lidar', label: 'Use LIDAR', icon: 'scan' },
		{ id: 'floor', label: 'Upload Floor Plan', icon: 'upload' },
		{ id: 'images', label: 'Overall Images', icon: 'image' }
	];
	let selectedInput = $state<string | null>(null);

	// Step 2: Invite students
	let studentEmails = $state([
		'napat.k@school.ac.th',
		'pranee.s@school.ac.th',
		'araya.m@school.ac.th'
	]);
	let newEmail = $state('');

	function addEmail() {
		const trimmed = newEmail.trim();
		if (trimmed && trimmed.includes('@') && !studentEmails.includes(trimmed)) {
			studentEmails = [...studentEmails, trimmed];
			newEmail = '';
		}
	}

	function removeEmail(email: string) {
		studentEmails = studentEmails.filter((e) => e !== email);
	}

	// Step 3: Proposed layout
	type SeatStatus = 'available' | 'occupied' | 'warning' | 'conflict' | 'empty';
	interface Seat {
		id: string;
		status: SeatStatus;
		studentName?: string;
	}

	const proposedSeats: Seat[][] = [
		[
			{ id: 'A1', status: 'occupied' },
			{ id: 'A2', status: 'occupied' },
			{ id: 'A3', status: 'occupied' },
			{ id: 'A4', status: 'occupied' },
			{ id: 'A5', status: 'occupied' }
		],
		[
			{ id: 'B1', status: 'occupied' },
			{ id: 'B2', status: 'occupied' },
			{ id: 'B3', status: 'occupied' },
			{ id: 'B4', status: 'occupied' },
			{ id: 'B5', status: 'occupied' }
		],
		[
			{ id: 'C1', status: 'occupied' },
			{ id: 'C2', status: 'occupied' },
			{ id: 'C3', status: 'occupied' },
			{ id: 'C4', status: 'occupied' },
			{ id: 'C5', status: 'occupied' }
		]
	];

	const scores = [
		{ label: 'Cleanliness', value: 92, color: '#5dbb63' },
		{ label: 'Sightlines', value: 85, color: 'var(--ikea-blue)' },
		{ label: 'Communication Ease', value: 78, color: '#ffdb00' }
	];

	const steps = ['Room Input', 'Invite Students', 'Proposed Layout'];
</script>

<div class="setup-page">
	<div class="setup-inner">
		<div class="setup-header">
			<div>
				<div class="breadcrumb"><a href="/">Overview</a> / <strong>Room Setup</strong></div>
				<h1 class="setup-title">Connect Room</h1>
			</div>
		</div>

		<!-- Step Indicator -->
		<div class="step-indicator" aria-label="Setup progress">
			{#each steps as step, i}
				<div
					class="step-item"
					class:done={i + 1 < currentStep}
					class:active={i + 1 === currentStep}
				>
					<div class="step-circle">
						{#if i + 1 < currentStep}
							<svg
								width="14"
								height="14"
								viewBox="0 0 24 24"
								fill="none"
								stroke="currentColor"
								stroke-width="3"
								stroke-linecap="round"
								stroke-linejoin="round"
							>
								<polyline points="20 6 9 17 4 12" />
							</svg>
						{:else}
							{i + 1}
						{/if}
					</div>
					<span class="step-label">{step}</span>
				</div>
				{#if i < steps.length - 1}
					<div class="step-connector" class:filled={i + 1 < currentStep}></div>
				{/if}
			{/each}
		</div>

		<!-- ===== STEP 1: Room Input ===== -->
		{#if currentStep === 1}
			<div class="step-content">
				<h2 class="step-title">How would you like to set up the room?</h2>
				<p class="step-desc">Choose a method to capture or describe your classroom layout.</p>
				<div class="method-grid">
					{#each inputMethods as method}
						<button
							class="method-btn"
							class:selected={selectedInput === method.id}
							onclick={() => (selectedInput = method.id)}
							aria-pressed={selectedInput === method.id}
						>
							<div class="method-icon">
								{#if method.icon === 'camera'}
									<svg
										width="28"
										height="28"
										viewBox="0 0 24 24"
										fill="none"
										stroke="currentColor"
										stroke-width="1.8"
										stroke-linecap="round"
										stroke-linejoin="round"
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
								{:else if method.icon === 'scan'}
									<svg
										width="28"
										height="28"
										viewBox="0 0 24 24"
										fill="none"
										stroke="currentColor"
										stroke-width="1.8"
										stroke-linecap="round"
										stroke-linejoin="round"
									>
										<polyline points="3 7 3 3 7 3" /><polyline points="17 3 21 3 21 7" />
										<polyline points="21 17 21 21 17 21" /><polyline points="7 21 3 21 3 17" />
										<line x1="3" y1="12" x2="21" y2="12" />
									</svg>
								{:else if method.icon === 'upload'}
									<svg
										width="28"
										height="28"
										viewBox="0 0 24 24"
										fill="none"
										stroke="currentColor"
										stroke-width="1.8"
										stroke-linecap="round"
										stroke-linejoin="round"
									>
										<polyline points="16 16 12 12 8 16" /><line x1="12" y1="12" x2="12" y2="21" />
										<path d="M20.39 18.39A5 5 0 0 0 18 9h-1.26A8 8 0 1 0 3 16.3" />
									</svg>
								{:else}
									<svg
										width="28"
										height="28"
										viewBox="0 0 24 24"
										fill="none"
										stroke="currentColor"
										stroke-width="1.8"
										stroke-linecap="round"
										stroke-linejoin="round"
									>
										<rect x="3" y="3" width="18" height="18" rx="2" ry="2" />
										<circle cx="8.5" cy="8.5" r="1.5" />
										<polyline points="21 15 16 10 5 21" />
									</svg>
								{/if}
							</div>
							<span class="method-label">{method.label}</span>
						</button>
					{/each}
				</div>
				<div class="step-actions">
					<button class="btn-primary" disabled={!selectedInput} onclick={() => (currentStep = 2)}>
						Continue →
					</button>
				</div>
			</div>
		{/if}

		<!-- ===== STEP 2: Invite Students ===== -->
		{#if currentStep === 2}
			<div class="step-content">
				<h2 class="step-title">เชิญนักเรียน</h2>
				<p class="step-desc">Add student email addresses to invite them to this classroom.</p>

				<div class="invite-list">
					{#each studentEmails as email}
						<div class="invite-item">
							<div class="invite-avatar">{email[0].toUpperCase()}</div>
							<span class="invite-email">{email}</span>
							<button
								class="invite-remove"
								onclick={() => removeEmail(email)}
								aria-label="Remove {email}"
							>
								<svg
									width="14"
									height="14"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2.5"
									stroke-linecap="round"
								>
									<line x1="18" y1="6" x2="6" y2="18" /><line x1="6" y1="6" x2="18" y2="18" />
								</svg>
							</button>
						</div>
					{/each}
				</div>

				<!-- Add email -->
				<div class="add-email-row">
					<input
						type="email"
						class="email-input"
						placeholder="example@mail.com"
						bind:value={newEmail}
						onkeydown={(e) => e.key === 'Enter' && addEmail()}
						aria-label="New student email"
					/>
					<button class="add-btn" onclick={addEmail} aria-label="Add email">
						<svg
							width="18"
							height="18"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2.5"
							stroke-linecap="round"
						>
							<line x1="12" y1="5" x2="12" y2="19" /><line x1="5" y1="12" x2="19" y2="12" />
						</svg>
					</button>
				</div>

				<div class="step-actions">
					<button class="btn-outline" onclick={() => (currentStep = 1)}>← Back</button>
					<button class="btn-primary" onclick={() => (currentStep = 3)}>Continue →</button>
				</div>
			</div>
		{/if}

		<!-- ===== STEP 3: Proposed Layout ===== -->
		{#if currentStep === 3}
			<div class="step-content">
				<h2 class="step-title">ข้อเสนอแผนผังห้อง</h2>
				<p class="step-desc">AI-optimized seating arrangement based on your preferences.</p>

				<!-- Score badges -->
				<div class="score-badges">
					{#each scores as score}
						<div class="score-badge">
							<div class="score-circle" style="--c:{score.color}">
								<svg width="48" height="48" viewBox="0 0 48 48">
									<circle cx="24" cy="24" r="20" fill="none" stroke="#e0e0e0" stroke-width="4" />
									<circle
										cx="24"
										cy="24"
										r="20"
										fill="none"
										stroke={score.color}
										stroke-width="4"
										stroke-dasharray="{(score.value / 100) * 125.66} 125.66"
										stroke-dashoffset="31.4"
										stroke-linecap="round"
										transform="rotate(-90 24 24)"
									/>
								</svg>
								<span class="score-num">{score.value}</span>
							</div>
							<span class="score-label">{score.label}</span>
						</div>
					{/each}
				</div>

				<!-- Proposed layout chart -->
				<div class="proposed-layout">
					<SeatingChart seats={proposedSeats} />
				</div>

				<div class="step-actions">
					<button class="btn-outline" onclick={() => (currentStep = 2)}>← Back</button>
					<button class="btn-outline">แก้ไขเอง</button>
					<button
						class="btn-yellow"
						onclick={() => {
							/* give students a vote */
						}}>ให้นักเรียนโหวต</button
					>
					<a href="/" class="btn-primary">สร้างใหม่ ✓</a>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	.setup-page {
		padding: 40px 0 100px;
		min-height: calc(100vh - 64px);
	}

	.setup-inner {
		max-width: 760px;
		margin: 0 auto;
		padding: 0 32px;
	}

	.setup-header {
		margin-bottom: 32px;
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

	.setup-title {
		font-size: 28px;
		font-weight: 900;
		letter-spacing: -0.4px;
	}

	/* Step indicator */
	.step-indicator {
		display: flex;
		align-items: center;
		margin-bottom: 48px;
	}

	.step-item {
		display: flex;
		align-items: center;
		gap: 8px;
		flex-shrink: 0;
	}

	.step-circle {
		width: 32px;
		height: 32px;
		border-radius: 50%;
		border: 2px solid var(--ikea-border);
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 13px;
		font-weight: 700;
		background: #fff;
		color: #888;
		transition:
			background 0.2s,
			border-color 0.2s,
			color 0.2s;
	}

	.step-item.active .step-circle {
		border-color: var(--ikea-blue);
		background: var(--ikea-blue);
		color: #fff;
	}

	.step-item.done .step-circle {
		border-color: #5dbb63;
		background: #5dbb63;
		color: #fff;
	}

	.step-label {
		font-size: 13px;
		font-weight: 600;
		color: #888;
	}

	.step-item.active .step-label {
		color: var(--ikea-dark);
	}
	.step-item.done .step-label {
		color: #2e7d32;
	}

	.step-connector {
		flex: 1;
		height: 2px;
		background: var(--ikea-border);
		margin: 0 12px;
		transition: background 0.2s;
	}

	.step-connector.filled {
		background: #5dbb63;
	}

	/* Step content */
	.step-content {
		display: flex;
		flex-direction: column;
		gap: 24px;
	}

	.step-title {
		font-size: 22px;
		font-weight: 800;
	}

	.step-desc {
		font-size: 15px;
		color: #555;
		margin-top: -16px;
	}

	/* Method grid */
	.method-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
		gap: 16px;
	}

	.method-btn {
		border: 2px solid var(--ikea-border);
		border-radius: 4px;
		padding: 24px 16px;
		background: #fff;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 12px;
		cursor: pointer;
		transition:
			border-color 0.15s,
			background 0.15s,
			box-shadow 0.15s;
		font-family: var(--font);
	}

	.method-btn:hover {
		border-color: var(--ikea-blue);
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
	}

	.method-btn.selected {
		border-color: var(--ikea-blue);
		background: #e8f0fa;
	}

	.method-icon {
		color: var(--ikea-dark);
	}
	.method-label {
		font-size: 13px;
		font-weight: 700;
		text-align: center;
	}

	/* Step actions */
	.step-actions {
		display: flex;
		gap: 12px;
		justify-content: flex-end;
		flex-wrap: wrap;
		padding-top: 8px;
		border-top: 1px solid var(--ikea-border);
	}

	.btn-primary {
		background: var(--ikea-yellow);
		color: var(--ikea-dark);
		border: none;
		padding: 12px 28px;
		font-weight: 700;
		font-size: 14px;
		border-radius: 4px;
		cursor: pointer;
		text-decoration: none;
		transition: background 0.15s;
	}

	.btn-primary:hover {
		background: #e8c800;
	}
	.btn-primary:disabled {
		opacity: 0.4;
		cursor: not-allowed;
	}

	.btn-outline {
		background: #fff;
		border: 2px solid var(--ikea-dark);
		color: var(--ikea-dark);
		padding: 12px 24px;
		font-size: 14px;
		font-weight: 700;
		border-radius: 4px;
		cursor: pointer;
		transition: background 0.15s;
		font-family: var(--font);
	}

	.btn-outline:hover {
		background: var(--ikea-gray);
	}

	.btn-yellow {
		background: var(--ikea-blue);
		color: #fff;
		border: none;
		padding: 12px 24px;
		font-size: 14px;
		font-weight: 700;
		border-radius: 4px;
		cursor: pointer;
		font-family: var(--font);
		transition: background 0.15s;
	}

	.btn-yellow:hover {
		background: #0047a0;
	}

	/* Invite list */
	.invite-list {
		display: flex;
		flex-direction: column;
		gap: 10px;
	}

	.invite-item {
		display: flex;
		align-items: center;
		gap: 12px;
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		padding: 12px 16px;
		background: #fafafa;
	}

	.invite-avatar {
		width: 32px;
		height: 32px;
		border-radius: 50%;
		background: var(--ikea-blue);
		color: #fff;
		font-size: 13px;
		font-weight: 700;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
	}

	.invite-email {
		flex: 1;
		font-size: 14px;
		font-weight: 600;
	}

	.invite-remove {
		background: none;
		border: none;
		color: #999;
		cursor: pointer;
		padding: 4px;
		border-radius: 4px;
		display: flex;
		align-items: center;
		transition: color 0.15s;
	}

	.invite-remove:hover {
		color: var(--ikea-red);
		background: #fff5f5;
	}

	.add-email-row {
		display: flex;
		gap: 10px;
	}

	.email-input {
		flex: 1;
		padding: 12px 16px;
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		font-size: 14px;
		font-family: var(--font);
		outline: none;
		transition: border-color 0.15s;
	}

	.email-input:focus {
		border-color: var(--ikea-blue);
	}

	.add-btn {
		background: var(--ikea-gray);
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		width: 48px;
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		transition: background 0.15s;
	}

	.add-btn:hover {
		background: #e0e0e0;
	}

	/* Score badges */
	.score-badges {
		display: flex;
		gap: 24px;
		flex-wrap: wrap;
	}

	.score-badge {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 8px;
	}

	.score-circle {
		position: relative;
		width: 48px;
		height: 48px;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.score-circle svg {
		position: absolute;
		top: 0;
		left: 0;
	}

	.score-num {
		font-size: 13px;
		font-weight: 800;
		position: relative;
		z-index: 1;
	}

	.score-label {
		font-size: 12px;
		font-weight: 700;
		text-align: center;
		max-width: 80px;
		color: #444;
	}

	/* Proposed layout */
	.proposed-layout {
		display: flex;
		justify-content: center;
		padding: 16px;
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		background: #fafafa;
	}
</style>
