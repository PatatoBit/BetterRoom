<script lang="ts">
	import { onMount, tick } from 'svelte';
	import { browser } from '$app/environment';
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';

	interface TourStep {
		id: string;
		title: string;
		description: string;
		route: string;
		selector: string;
		advanceByClick?: boolean;
	}

	interface SpotlightRect {
		top: number;
		left: number;
		width: number;
		height: number;
	}

	const COMPLETE_KEY = 'betterroom-onboarding-complete-v1';

	const steps: TourStep[] = [
		{
			id: 'create-class',
			title: 'สร้างห้องเรียนแรกของคุณ',
			description:
				'ใช้ปุ่ม เพิ่มห้อง เพื่อเริ่มการแนะนำการใช้งาน และเริ่มขั้นตอนตั้งค่าห้องเรียนใหม่',
			route: '/',
			selector: '[data-tour="overview-add-room"]',
			advanceByClick: true
		},
		{
			id: 'pick-input',
			title: 'เลือกวิธีนำเข้าข้อมูลห้อง',
			description:
				'เลือกวิธีตั้งค่าหนึ่งแบบ กล้อง, LIDAR, แปลนห้อง หรือรูปภาพ ใช้ได้ทั้งหมดสำหรับเดโมนี้',
			route: '/setup',
			selector: '[data-tour="setup-method-camera"]',
			advanceByClick: true
		},
		{
			id: 'step-one-continue',
			title: 'ไปยังขั้นตอนเชิญนักเรียน',
			description: 'ไปต่อขั้นตอนที่ 2 หลังจากเลือกวิธีนำเข้าข้อมูลห้องแล้ว',
			route: '/setup',
			selector: '[data-tour="setup-step1-continue"]',
			advanceByClick: true
		},
		{
			id: 'invite-students',
			title: 'เชิญนักเรียน',
			description: 'เพิ่มอีเมลในช่องนี้เพื่อเชิญนักเรียนเข้าสู่ห้องเรียนที่เพิ่งสร้าง',
			route: '/setup',
			selector: '[data-tour="setup-email-input"]'
		},
		{
			id: 'step-two-continue',
			title: 'เปิดดูแผนผังที่แนะนำ',
			description: 'ไปยังขั้นตอนที่ 3 เพื่อดูผังที่นั่งที่ AI สร้างให้',
			route: '/setup',
			selector: '[data-tour="setup-step2-continue"]',
			advanceByClick: true
		},
		{
			id: 'randomize-layout',
			title: 'ลองแผนผังแบบอื่น',
			description: 'ใช้ปุ่ม สุ่มแผนผัง เพื่อดูรูปแบบที่นั่งและทรงห้องที่ต่างกัน',
			route: '/setup',
			selector: '[data-tour="setup-randomize"]'
		},
		{
			id: 'finish-create',
			title: 'เสร็จสิ้นและสร้างห้อง',
			description: 'จบเดโมการแนะนำนี้ด้วยการสร้างห้องเรียน และกลับไปยังหน้าแดชบอร์ด',
			route: '/setup',
			selector: '[data-tour="setup-create-finish"]',
			advanceByClick: true
		}
	];

	let active = $state(false);
	let stepIndex = $state(0);
	let spotlightRect = $state<SpotlightRect | null>(null);
	let targetMissing = $state(false);
	let tooltipTop = $state(24);
	let tooltipLeft = $state(24);

	const currentStep = $derived(steps[stepIndex]);
	const lastStepIndex = steps.length - 1;

	function clamp(value: number, min: number, max: number): number {
		return Math.min(Math.max(value, min), max);
	}

	function routeMatches(stepRoute: string, pathname: string): boolean {
		if (stepRoute === '/') return pathname === '/';
		return pathname === stepRoute || pathname.startsWith(`${stepRoute}/`);
	}

	function computeTooltipPosition(rect: SpotlightRect) {
		const panelWidth = Math.min(360, window.innerWidth - 24);
		const panelHeight = 210;
		const spaceBelow = window.innerHeight - (rect.top + rect.height);
		const showAbove = spaceBelow < panelHeight + 20;

		tooltipTop = showAbove
			? clamp(rect.top - panelHeight - 16, 12, window.innerHeight - panelHeight - 12)
			: clamp(rect.top + rect.height + 16, 12, window.innerHeight - panelHeight - 12);
		tooltipLeft = clamp(rect.left, 12, window.innerWidth - panelWidth - 12);
	}

	function updateSpotlight() {
		if (!browser || !active || !routeMatches(currentStep.route, $page.url.pathname)) {
			spotlightRect = null;
			targetMissing = true;
			return;
		}

		const target = document.querySelector<HTMLElement>(currentStep.selector);
		if (!target) {
			spotlightRect = null;
			targetMissing = true;
			return;
		}

		targetMissing = false;
		const rect = target.getBoundingClientRect();
		spotlightRect = {
			top: Math.max(6, rect.top - 8),
			left: Math.max(6, rect.left - 8),
			width: Math.max(24, rect.width + 16),
			height: Math.max(24, rect.height + 16)
		};
		computeTooltipPosition(spotlightRect);
	}

	async function ensureStepRoute(index: number) {
		const route = steps[index].route;
		if (!routeMatches(route, $page.url.pathname)) {
			await goto(route);
			await tick();
		}
	}

	async function startTour() {
		if (!browser) return;
		active = true;
		stepIndex = 0;
		await ensureStepRoute(0);
		updateSpotlight();
	}

	function skipTour() {
		active = false;
		localStorage.setItem(COMPLETE_KEY, '1');
	}

	function finishTour() {
		active = false;
		localStorage.setItem(COMPLETE_KEY, '1');
	}

	async function nextStep() {
		if (currentStep.advanceByClick) {
			const target = document.querySelector<HTMLElement>(currentStep.selector);
			target?.click();
		}

		if (stepIndex >= lastStepIndex) {
			finishTour();
			return;
		}

		stepIndex += 1;
		await ensureStepRoute(stepIndex);
		await tick();
		updateSpotlight();
	}

	async function prevStep() {
		if (stepIndex <= 0) return;
		stepIndex -= 1;
		await ensureStepRoute(stepIndex);
		await tick();
		updateSpotlight();
	}

	onMount(() => {
		if (!browser) return;

		const onResize = () => updateSpotlight();
		const onScroll = () => updateSpotlight();
		const onKeydown = (event: KeyboardEvent) => {
			if (event.key === 'Escape' && active) {
				skipTour();
			}
		};

		window.addEventListener('resize', onResize);
		window.addEventListener('scroll', onScroll, true);
		window.addEventListener('keydown', onKeydown);

		if (!localStorage.getItem(COMPLETE_KEY)) {
			setTimeout(() => {
				startTour();
			}, 250);
		}

		const interval = setInterval(() => {
			if (active) updateSpotlight();
		}, 180);

		return () => {
			window.removeEventListener('resize', onResize);
			window.removeEventListener('scroll', onScroll, true);
			window.removeEventListener('keydown', onKeydown);
			clearInterval(interval);
		};
	});

	$effect(() => {
		if (!active) return;
		$page.url.pathname;
		stepIndex;
		tick().then(updateSpotlight);
	});
</script>

{#if !active}
	<button class="tour-replay" onclick={startTour}>เริ่มเดโมแนะนำอีกครั้ง</button>
{/if}

{#if active}
	<div class="tour-layer" aria-hidden="true">
		{#if spotlightRect}
			<div
				class="tour-spotlight"
				style="top: {spotlightRect.top}px; left: {spotlightRect.left}px; width: {spotlightRect.width}px; height: {spotlightRect.height}px;"
			></div>
		{/if}

		<div class="tour-dialog" style="top: {tooltipTop}px; left: {tooltipLeft}px;" aria-live="polite">
			<div class="tour-meta">ขั้นตอน {stepIndex + 1} จาก {steps.length}</div>
			<h3>{currentStep.title}</h3>
			<p>{currentStep.description}</p>

			{#if targetMissing}
				<p class="tour-waiting">กำลังรอให้องค์ประกอบนี้ปรากฏบนหน้าจอ...</p>
			{/if}

			<div class="tour-actions">
				<button class="btn-flat" onclick={skipTour}>ข้าม</button>
				<button class="btn-flat" onclick={prevStep} disabled={stepIndex === 0}>ย้อนกลับ</button>
				<button class="btn-strong" onclick={nextStep}>
					{stepIndex === lastStepIndex ? 'เสร็จสิ้น' : 'ถัดไป'}
				</button>
			</div>
		</div>
	</div>
{/if}

<style>
	.tour-layer {
		position: fixed;
		inset: 0;
		z-index: 1200;
		pointer-events: none;
	}

	.tour-layer::before {
		content: '';
		position: absolute;
		inset: 0;
		background: rgba(8, 12, 19, 0.48);
	}

	.tour-spotlight {
		position: fixed;
		border-radius: 10px;
		border: 2px solid rgba(255, 219, 0, 0.95);
		box-shadow:
			0 0 0 9999px rgba(8, 12, 19, 0.48),
			0 0 0 3px rgba(255, 255, 255, 0.7),
			0 16px 36px rgba(0, 0, 0, 0.35);
		transition:
			top 0.18s ease,
			left 0.18s ease,
			width 0.18s ease,
			height 0.18s ease;
		pointer-events: none;
	}

	.tour-dialog {
		position: fixed;
		width: min(360px, calc(100vw - 24px));
		background: #ffffff;
		border: 1px solid #d4d4d4;
		border-radius: 12px;
		padding: 16px;
		box-shadow: 0 18px 48px rgba(0, 0, 0, 0.24);
		pointer-events: auto;
	}

	.tour-meta {
		font-size: 11px;
		font-weight: 700;
		letter-spacing: 0.6px;
		text-transform: uppercase;
		color: #0058a3;
		margin-bottom: 8px;
	}

	.tour-dialog h3 {
		font-size: 18px;
		font-weight: 800;
		line-height: 1.25;
		margin-bottom: 8px;
	}

	.tour-dialog p {
		font-size: 14px;
		color: #333;
		line-height: 1.45;
	}

	.tour-waiting {
		margin-top: 10px;
		color: #8a5a00;
		font-weight: 600;
		font-size: 12px;
	}

	.tour-actions {
		display: flex;
		justify-content: flex-end;
		gap: 8px;
		margin-top: 16px;
	}

	.btn-flat,
	.btn-strong {
		border-radius: 8px;
		padding: 9px 12px;
		font-size: 13px;
		font-weight: 700;
		border: 1px solid transparent;
		font-family: var(--font);
	}

	.btn-flat {
		background: #f4f4f4;
		color: #111;
		border-color: #e2e2e2;
	}

	.btn-flat:disabled {
		opacity: 0.4;
		cursor: not-allowed;
	}

	.btn-strong {
		background: var(--ikea-yellow);
		color: #111;
		border-color: #e3c500;
	}

	.tour-replay {
		position: fixed;
		right: 18px;
		bottom: 18px;
		z-index: 1100;
		border: none;
		border-radius: 999px;
		background: var(--ikea-blue);
		color: #fff;
		font-size: 13px;
		font-weight: 700;
		padding: 11px 16px;
		box-shadow: 0 10px 22px rgba(0, 88, 163, 0.35);
	}

	@media (max-width: 700px) {
		.tour-replay {
			right: 12px;
			bottom: 12px;
			font-size: 12px;
			padding: 10px 14px;
		}
	}
</style>
