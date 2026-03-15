<script lang="ts">
	// ---- types ----------------------------------------------------------------
	interface Message {
		role: 'user' | 'ai';
		text: string;
		ts: Date;
	}

	interface QuickTag {
		label: string;
		text: string;
	}

	// ---- demo data ------------------------------------------------------------
	const student = { name: 'นายกิตติ สมใจ', id: 'S-042', class: 'ห้อง 161', row: 2, col: 3 };

	// 5×6 grid representation (row, col) – student seat highlighted
	const ROWS = 4;
	const COLS = 6;

	const QUICK_TAGS: QuickTag[] = [
		{ label: '🌬️ ลมเย็นเกินไป', text: 'ที่นั่งของฉันอยู่ใต้แอร์พอดี ลมเย็นมากเกินไปในช่วงเช้า' },
		{ label: '☀️ แสงแดดจ้า', text: 'แสงแดดส่องเข้ามาตรงตาตอนบ่าย ทำให้มองกระดานไม่ชัด' },
		{ label: '🗣️ เพื่อนพูดมาก', text: 'เพื่อนที่นั่งข้างๆ คุยเยอะทำให้สมาธิหาย' },
		{ label: '👀 มองกระดานไม่ชัด', text: 'มองกระดานได้ยาก อยากย้ายมานั่งแถวหน้ากว่านี้' },
		{ label: '🤝 ขาดการมีส่วนร่วม', text: 'รู้สึกโดดเดี่ยว อยากนั่งใกล้กลุ่มเพื่อนที่ทำงานกันได้' },
		{ label: '✅ พอใจที่นั่งปัจจุบัน', text: 'ชอบที่นั่งปัจจุบันมาก ไม่ต้องการเปลี่ยนแปลงอะไร' }
	];

	// ---- AI demo responses ---------------------------------------------------
	function aiReply(userText: string): string {
		const lower = userText.toLowerCase();
		if (lower.includes('แอร์') || lower.includes('ลม') || lower.includes('เย็น'))
			return 'ขอบคุณที่แจ้ง! ระบบจะลองย้ายคุณออกจากใต้แอร์ในการจัดที่นั่งรอบถัดไป 🎯';
		if (lower.includes('แสง') || lower.includes('แดด') || lower.includes('windows'))
			return 'รับทราบแล้วค่ะ ระบบจะหลีกเลี่ยงให้ที่นั่งของคุณไม่อยู่ตรงแนวหน้าต่างในรอบถัดไป ☀️';
		if (lower.includes('เพื่อน') || lower.includes('คุย') || lower.includes('สมาธิ'))
			return 'เข้าใจแล้วค่ะ ระบบจะพยายามจัดที่นั่งให้ห่างจากนักเรียนที่มีรูปแบบการเรียนต่างกัน 📚';
		if (lower.includes('กระดาน') || lower.includes('มองไม่') || lower.includes('แถวหน้า'))
			return 'ขอบคุณ! คำขอนี้จะถูกนำไปพิจารณาในรอบถัดไป ระบบจะพยายามจัดให้อยู่ใกล้กระดานมากขึ้น 👁️';
		if (lower.includes('โดดเดี่ยว') || lower.includes('กลุ่ม') || lower.includes('เพื่อน'))
			return 'รับทราบแล้วค่ะ ระบบจะพิจารณาจัดกลุ่มการเรียนให้เหมาะสมกับคุณมากขึ้น 🤝';
		if (lower.includes('ชอบ') || lower.includes('พอใจ') || lower.includes('ไม่ต้องการ'))
			return 'ขอบคุณสำหรับ feedback ดีๆ ค่ะ! ระบบจะพยายามรักษาตำแหน่งที่ใกล้เคียงในรอบถัดไป ✨';
		return 'ขอบคุณสำหรับ feedback ค่ะ! ข้อมูลของคุณจะถูกนำไปพิจารณาในการจัดที่นั่งรอบถัดไป 🤖';
	}

	// ---- state ---------------------------------------------------------------
	let inputText = $state('');
	let messages = $state<Message[]>([
		{
			role: 'ai',
			text: `สวัสดีค่ะ ${student.name} 👋 วันนี้รู้สึกอย่างไรกับที่นั่งของคุณบ้าง? บอก AI ได้เลยค่ะ`,
			ts: new Date()
		}
	]);
	let isTyping = $state(false);
	let submitted = $state(false);
	let chatEndEl: HTMLDivElement | null = null;

	function scrollToBottom() {
		setTimeout(() => chatEndEl?.scrollIntoView({ behavior: 'smooth' }), 50);
	}

	function insertTag(tag: QuickTag) {
		inputText = tag.text;
	}

	async function sendMessage() {
		const text = inputText.trim();
		if (!text) return;

		messages = [...messages, { role: 'user', text, ts: new Date() }];
		inputText = '';
		isTyping = true;
		scrollToBottom();

		await new Promise((r) => setTimeout(r, 1200));
		isTyping = false;
		messages = [...messages, { role: 'ai', text: aiReply(text), ts: new Date() }];
		submitted = true;
		scrollToBottom();
	}

	function handleKeydown(e: KeyboardEvent) {
		if (e.key === 'Enter' && !e.shiftKey) {
			e.preventDefault();
			sendMessage();
		}
	}

	function formatTime(d: Date) {
		return d.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });
	}
</script>

<section class="sf-page">
	<div class="sf-inner">
		<!-- ── Header ── -->
		<div class="sf-header">
			<div class="sf-header-left">
				<a href="/" class="back-link">← กลับหน้าหลัก</a>
				<h1 class="sf-title">ห้องนักเรียน</h1>
				<p class="sf-subtitle">แจ้ง AI เพื่อปรับที่นั่งรอบถัดไป</p>
			</div>
			<div class="sf-badge">
				<div class="badge-name">{student.name}</div>
				<div class="badge-meta">{student.id} · {student.class}</div>
			</div>
		</div>

		<div class="sf-grid">
			<!-- ── Left: Seat Visual ── -->
			<div class="sf-panel panel-seat">
				<h2 class="panel-title">ที่นั่งของคุณ</h2>
				<p class="panel-hint">แถว {student.row} · คอลัมน์ {student.col}</p>

				<div class="seat-grid">
					{#each Array(ROWS) as _, ri}
						<div class="seat-row">
							{#each Array(COLS) as _, ci}
								{@const isMe = ri + 1 === student.row && ci + 1 === student.col}
								{@const isEmpty = (ri === 0 && ci === 5) || (ri === 3 && ci === 0)}
								<div
									class="seat-cell"
									class:seat-me={isMe}
									class:seat-empty={isEmpty}
									class:seat-taken={!isMe && !isEmpty}
									title={isMe ? student.name : ''}
								>
									{#if isMe}
										<span class="seat-you-label">คุณ</span>
									{/if}
								</div>
							{/each}
						</div>
					{/each}
				</div>

				<div class="legend">
					<span class="legend-dot dot-me"></span><span>ที่นั่งของคุณ</span>
					<span class="legend-dot dot-taken"></span><span>เพื่อนร่วมชั้น</span>
				</div>

				{#if submitted}
					<div class="submit-banner">
						<span class="submit-icon">✅</span>
						<span>AI รับ feedback แล้ว — จะอัปเดตที่นั่งรอบถัดไป</span>
					</div>
				{/if}
			</div>

			<!-- ── Right: Chat ── -->
			<div class="sf-panel panel-chat">
				<h2 class="panel-title">บอก AI ของคุณ</h2>

				<!-- Quick tags -->
				<div class="quick-tags">
					{#each QUICK_TAGS as tag}
						<button class="tag-chip" onclick={() => insertTag(tag)}>{tag.label}</button>
					{/each}
				</div>

				<!-- Messages -->
				<div class="chat-messages">
					{#each messages as msg}
						<div class="msg-row" class:msg-user={msg.role === 'user'} class:msg-ai={msg.role === 'ai'}>
							{#if msg.role === 'ai'}
								<div class="msg-avatar">🤖</div>
							{/if}
							<div class="msg-bubble">
								<p class="msg-text">{msg.text}</p>
								<span class="msg-time">{formatTime(msg.ts)}</span>
							</div>
						</div>
					{/each}

					{#if isTyping}
						<div class="msg-row msg-ai">
							<div class="msg-avatar">🤖</div>
							<div class="msg-bubble typing-bubble">
								<span class="dot"></span><span class="dot"></span><span class="dot"></span>
							</div>
						</div>
					{/if}

					<div bind:this={chatEndEl}></div>
				</div>

				<!-- Input -->
				<div class="chat-input-row">
					<textarea
						class="chat-input"
						placeholder="พิมพ์ความรู้สึกของคุณเกี่ยวกับที่นั่ง... (Enter เพื่อส่ง)"
						bind:value={inputText}
						onkeydown={handleKeydown}
						rows="2"
					></textarea>
					<button class="send-btn" onclick={sendMessage} disabled={!inputText.trim()}>ส่ง ↑</button>
				</div>
			</div>
		</div>
	</div>
</section>

<style>
	/* ── Layout ── */
	.sf-page {
		padding: 40px 0 80px;
		min-height: 100vh;
		background: var(--ikea-gray);
	}

	.sf-inner {
		max-width: 1200px;
		margin: 0 auto;
		padding: 0 32px;
	}

	/* ── Header ── */
	.sf-header {
		display: flex;
		align-items: flex-end;
		justify-content: space-between;
		margin-bottom: 32px;
		flex-wrap: wrap;
		gap: 16px;
	}

	.back-link {
		font-size: 13px;
		color: var(--ikea-blue);
		display: block;
		margin-bottom: 8px;
	}
	.back-link:hover {
		text-decoration: underline;
	}

	.sf-title {
		font-size: 32px;
		font-weight: 900;
		letter-spacing: -0.5px;
	}

	.sf-subtitle {
		font-size: 15px;
		color: #555;
		margin-top: 4px;
	}

	.sf-badge {
		background: var(--ikea-blue);
		color: #fff;
		border-radius: 12px;
		padding: 12px 20px;
		text-align: right;
	}

	.badge-name {
		font-weight: 800;
		font-size: 16px;
	}

	.badge-meta {
		font-size: 13px;
		opacity: 0.8;
		margin-top: 2px;
	}

	/* ── Two-column grid ── */
	.sf-grid {
		display: grid;
		grid-template-columns: 340px 1fr;
		gap: 24px;
		align-items: start;
	}

	@media (max-width: 860px) {
		.sf-grid {
			grid-template-columns: 1fr;
		}
	}

	/* ── Panels ── */
	.sf-panel {
		background: #fff;
		border: 1.5px solid var(--ikea-border);
		border-radius: 16px;
		padding: 28px 24px;
	}

	.panel-title {
		font-size: 18px;
		font-weight: 800;
		margin-bottom: 4px;
	}

	.panel-hint {
		font-size: 13px;
		color: #888;
		margin-bottom: 20px;
	}

	/* ── Seat grid ── */
	.seat-grid {
		display: flex;
		flex-direction: column;
		gap: 8px;
		margin-bottom: 18px;
	}

	.seat-row {
		display: flex;
		gap: 8px;
		justify-content: center;
	}

	.seat-cell {
		width: 40px;
		height: 40px;
		border-radius: 8px;
		border: 2px solid transparent;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 10px;
		font-weight: 700;
		transition: transform 0.15s;
	}

	.seat-taken {
		background: #d8f5da;
		border-color: var(--green-seat);
	}

	.seat-empty {
		background: #f0f0f0;
		border-color: #ddd;
	}

	.seat-me {
		background: var(--ikea-yellow);
		border-color: #e8c400;
		transform: scale(1.1);
		box-shadow: 0 0 0 3px rgba(255, 219, 0, 0.35);
	}

	.seat-you-label {
		font-size: 9px;
		font-weight: 900;
		color: #333;
	}

	.legend {
		display: flex;
		align-items: center;
		gap: 8px;
		font-size: 12px;
		color: #666;
		margin-bottom: 20px;
		flex-wrap: wrap;
	}

	.legend-dot {
		width: 12px;
		height: 12px;
		border-radius: 3px;
		display: inline-block;
	}

	.dot-me {
		background: var(--ikea-yellow);
		border: 1.5px solid #e8c400;
	}

	.dot-taken {
		background: #d8f5da;
		border: 1.5px solid var(--green-seat);
	}

	/* ── Submit banner ── */
	.submit-banner {
		background: #f0fdf4;
		border: 1.5px solid #86efac;
		border-radius: 10px;
		padding: 12px 16px;
		font-size: 13px;
		font-weight: 600;
		color: #166534;
		display: flex;
		align-items: center;
		gap: 8px;
	}

	.submit-icon {
		font-size: 16px;
	}

	/* ── Chat panel ── */
	.panel-chat {
		display: flex;
		flex-direction: column;
		gap: 16px;
	}

	/* Quick tags */
	.quick-tags {
		display: flex;
		flex-wrap: wrap;
		gap: 8px;
	}

	.tag-chip {
		background: var(--ikea-gray);
		border: 1.5px solid var(--ikea-border);
		border-radius: 999px;
		padding: 6px 14px;
		font-size: 13px;
		font-family: var(--font);
		cursor: pointer;
		transition:
			background 0.15s,
			border-color 0.15s;
	}

	.tag-chip:hover {
		background: var(--ikea-yellow);
		border-color: #e8c400;
	}

	/* Messages */
	.chat-messages {
		flex: 1;
		min-height: 280px;
		max-height: 380px;
		overflow-y: auto;
		display: flex;
		flex-direction: column;
		gap: 12px;
		padding: 4px 2px;
	}

	.msg-row {
		display: flex;
		align-items: flex-end;
		gap: 8px;
	}

	.msg-user {
		flex-direction: row-reverse;
	}

	.msg-avatar {
		font-size: 20px;
		flex-shrink: 0;
		margin-bottom: 2px;
	}

	.msg-bubble {
		max-width: 75%;
		padding: 10px 14px;
		border-radius: 14px;
		font-size: 14px;
		line-height: 1.55;
	}

	.msg-ai .msg-bubble {
		background: var(--ikea-gray);
		border: 1.5px solid var(--ikea-border);
		border-bottom-left-radius: 4px;
	}

	.msg-user .msg-bubble {
		background: var(--ikea-blue);
		color: #fff;
		border-bottom-right-radius: 4px;
	}

	.msg-text {
		margin-bottom: 4px;
	}

	.msg-time {
		font-size: 11px;
		opacity: 0.55;
		display: block;
		text-align: right;
	}

	/* Typing indicator */
	.typing-bubble {
		display: flex;
		align-items: center;
		gap: 5px;
		padding: 12px 16px;
	}

	.dot {
		width: 7px;
		height: 7px;
		background: #aaa;
		border-radius: 50%;
		animation: bounce 1.2s infinite;
	}

	.dot:nth-child(2) {
		animation-delay: 0.2s;
	}

	.dot:nth-child(3) {
		animation-delay: 0.4s;
	}

	@keyframes bounce {
		0%,
		60%,
		100% {
			transform: translateY(0);
		}
		30% {
			transform: translateY(-6px);
		}
	}

	/* Input row */
	.chat-input-row {
		display: flex;
		gap: 10px;
		align-items: flex-end;
	}

	.chat-input {
		flex: 1;
		resize: none;
		border: 1.5px solid var(--ikea-border);
		border-radius: 12px;
		padding: 10px 14px;
		font-family: var(--font);
		font-size: 14px;
		line-height: 1.5;
		outline: none;
		transition: border-color 0.15s;
	}

	.chat-input:focus {
		border-color: var(--ikea-blue);
	}

	.send-btn {
		background: var(--ikea-blue);
		color: #fff;
		border: none;
		border-radius: 12px;
		padding: 10px 18px;
		font-family: var(--font);
		font-size: 15px;
		font-weight: 700;
		cursor: pointer;
		transition: background 0.15s;
		white-space: nowrap;
	}

	.send-btn:hover:not(:disabled) {
		background: #004a8c;
	}

	.send-btn:disabled {
		opacity: 0.4;
		cursor: not-allowed;
	}
</style>
