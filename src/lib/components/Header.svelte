<script lang="ts">
	import { page } from '$app/stores';

	let searchQuery = $state('');

	const navLinks = [
		{ href: '/', label: 'ภาพรวม' },
		{ href: '/classroom/161', label: 'ห้องเรียน' },
		{ href: '/setup', label: 'ตั้งค่าห้อง' },
		{ href: '/analytics', label: 'สถิติ' }
	];
</script>

<header class="header">
	<div class="header-inner">
		<!-- Logo -->
		<a href="/" class="logo" aria-label="หน้าหลัก BetterRoom">
			<span class="logo-box">BR</span>
			<span class="logo-text">BetterRoom</span>
		</a>

		<!-- Nav -->
		<nav class="nav" aria-label="เมนูหลัก">
			{#each navLinks as link}
				<a
					href={link.href}
					class="nav-link"
					class:active={$page.url.pathname === link.href ||
						($page.url.pathname.startsWith('/classroom') && link.href.startsWith('/classroom'))}
				>
					{link.label}
				</a>
			{/each}
		</nav>

		<!-- Search -->
		<div class="search-wrap">
			<svg
				class="search-icon"
				width="18"
				height="18"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2.5"
				stroke-linecap="round"
				stroke-linejoin="round"
			>
				<circle cx="11" cy="11" r="8" /><line x1="21" y1="21" x2="16.65" y2="16.65" />
			</svg>
			<input
				type="search"
				class="search-input"
				placeholder="ค้นหาห้องเรียนหรือนักเรียน..."
				bind:value={searchQuery}
				aria-label="ค้นหา"
			/>
		</div>

		<!-- Right Icons -->
		<div class="right-icons">
			<button class="icon-btn" aria-label="การแจ้งเตือน">
				<svg
					width="22"
					height="22"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9" /><path
						d="M13.73 21a2 2 0 0 1-3.46 0"
					/>
				</svg>
				<span class="badge">3</span>
			</button>
			<button class="icon-btn" aria-label="รายการโปรด">
				<svg
					width="22"
					height="22"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<path
						d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"
					/>
				</svg>
			</button>
			<button class="icon-btn profile-btn" aria-label="โปรไฟล์ผู้ใช้">
				<svg
					width="22"
					height="22"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2" /><circle cx="12" cy="7" r="4" />
				</svg>
				<span class="profile-name">ครู</span>
			</button>
		</div>
	</div>
</header>

<style>
	.header {
		background: #fff;
		border-bottom: 1px solid var(--ikea-border);
		position: sticky;
		top: 0;
		z-index: 100;
	}

	.header-inner {
		max-width: 1400px;
		margin: 0 auto;
		padding: 0 24px;
		height: 64px;
		display: flex;
		align-items: center;
		gap: 32px;
	}

	.logo {
		display: flex;
		align-items: center;
		gap: 10px;
		flex-shrink: 0;
		text-decoration: none;
	}

	.logo-box {
		background: var(--ikea-blue);
		color: var(--ikea-yellow);
		font-size: 15px;
		font-weight: 900;
		width: 40px;
		height: 30px;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 4px;
		letter-spacing: 1px;
	}

	.logo-text {
		font-size: 18px;
		font-weight: 800;
		color: var(--ikea-dark);
		letter-spacing: -0.3px;
	}

	.nav {
		display: flex;
		align-items: center;
		gap: 4px;
		flex-shrink: 0;
	}

	.nav-link {
		padding: 6px 14px;
		font-size: 14px;
		font-weight: 600;
		color: var(--ikea-dark);
		border-radius: 4px;
		transition: background 0.15s;
		white-space: nowrap;
	}

	.nav-link:hover {
		background: var(--ikea-gray);
	}

	.nav-link.active {
		color: var(--ikea-blue);
		background: #e8f0fa;
	}

	.search-wrap {
		flex: 1;
		position: relative;
		max-width: 480px;
	}

	.search-icon {
		position: absolute;
		left: 12px;
		top: 50%;
		transform: translateY(-50%);
		color: #767676;
		pointer-events: none;
	}

	.search-input {
		width: 100%;
		padding: 9px 12px 9px 40px;
		border: 1.5px solid var(--ikea-border);
		border-radius: 4px;
		font-size: 14px;
		font-family: var(--font);
		background: var(--ikea-gray);
		color: var(--ikea-dark);
		outline: none;
		transition:
			border-color 0.15s,
			background 0.15s;
	}

	.search-input:focus {
		border-color: var(--ikea-blue);
		background: #fff;
	}

	.search-input::placeholder {
		color: #767676;
	}

	.right-icons {
		display: flex;
		align-items: center;
		gap: 4px;
		margin-left: auto;
		flex-shrink: 0;
	}

	.icon-btn {
		background: none;
		border: none;
		padding: 8px;
		border-radius: 4px;
		display: flex;
		align-items: center;
		gap: 6px;
		position: relative;
		color: var(--ikea-dark);
		transition: background 0.15s;
	}

	.icon-btn:hover {
		background: var(--ikea-gray);
	}

	.badge {
		position: absolute;
		top: 4px;
		right: 4px;
		background: var(--ikea-blue);
		color: #fff;
		font-size: 10px;
		font-weight: 700;
		width: 16px;
		height: 16px;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.profile-name {
		font-size: 13px;
		font-weight: 600;
	}
</style>
