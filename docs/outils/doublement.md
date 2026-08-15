# [Temps de doublement volumique](https://pubs.rsna.org/doi/epdf/10.1148/radiol.2017151022){:target="_blank"}

<div class="box md-typeset" id="volume-doubling-time">
	<form onsubmit="return false;" oninput="computeDoublingTime()">
		<div class="mode-toggle" role="tablist" aria-label="Mode de saisie">
			<label class="mode-option">
				<input id="mode-volume" name="doubling-mode" type="radio" value="volume" checked>
				<span>Volume</span>
			</label>
			<label class="mode-option">
				<input id="mode-diameter" name="doubling-mode" type="radio" value="diameter">
				<span>Diamètre</span>
			</label>
		</div>
		<div class="doubling-grid">
			<div class="doubling-exam">
				<div class="group-title">Examen actuel</div>
				<input id="doubling-date-2" type="date" />
				<!-- dimension fields removed; mode selector controls interpretation of the result boxes -->
			</div>

			<div class="doubling-exam">
				<div class="group-title">Examen initial</div>
				<input id="doubling-date-1" type="date" />
				<!-- dimension fields removed; mode selector controls interpretation of the result boxes -->
			</div>
		</div>

		<div class="results doubling-results">
			<div class="result">
				<div class="title">Volume actuel (cc)</div>
				<div class="value">
					<input id="doubling-vol-2" class="vol-input" type="text" inputmode="decimal" placeholder="" title="Saisir un volume en cc (sera rempli par le calcul si 3 dimensions renseignées)" />
				</div>
			</div>
			<div class="result">
				<div class="title">Volume initial (cc)</div>
				<div class="value">
					<input id="doubling-vol-1" class="vol-input" type="text" inputmode="decimal" placeholder="" title="Saisir un volume en cc (sera rempli par le calcul si 3 dimensions renseignées)" />
				</div>
			</div>
			<div class="result doubling-main-result">
				<div class="value"><span id="doubling-result">—</span></div>
				<div class="subvalue"><span id="doubling-subtext"></span></div>
			</div>
		</div>

		<div class="doubling-sentence">
			<div class="result wide">
				<div class="value" id="doubling-sentence-text">Volume lésionnel estimé à — cc contre — cc le —/—/—, soit un temps de doublement de — jours.</div>
			</div>
		</div>

		<div class="actions">
			<button id="copy-doubling-btn" type="button" class="copy" onclick="copyDoublingSentence()" disabled>Copier</button>
			<button type="button" class="clear" onclick="clearDoublingTime()">Effacer</button>
			<span class="copied" id="doubling-copied" aria-live="polite"></span>
		</div>
	</form>
</div>

<style>
	.actions { 
		margin: 0.45rem 0 0.35rem; 
	}

    .group-title {
        font-weight: 600;
    }

	.doubling-grid {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: 1rem;
	}

	.doubling-exam {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.doubling-exam label {
		font-size: 0.8rem;
	}

	.doubling-dimensions {
		display: grid;
		grid-template-columns: repeat(3, minmax(0, 1fr));
		gap: 0.6rem;
		margin-top: 0.25rem;
	}

	.doubling-results {
		margin-top: 1rem;
	}

	/* mode toggle: centered pill, halves look like clickable buttons */
	.mode-toggle { display:flex; gap:0; margin:0 auto 0.8rem; width:14rem; border-radius:999px; overflow:hidden; border:1px solid rgba(0,0,0,0.06); background:var(--md-sys-color-surface-container, transparent); }
	.mode-option { flex:1; }
	.mode-option input { display:none; }
	.mode-option span { display:block; text-align:center; padding:0.5rem 0; font-size:0.8rem; cursor:pointer; background:transparent; color:inherit; transition: background .14s ease; user-select:none; }
	.mode-option:first-child span { border-right:1px solid rgba(0,0,0,0.06); }
	.mode-option input:checked + span { background: linear-gradient(to bottom, rgba(0,0,0,0.03), rgba(0,0,0,0.01)); font-weight:700; box-shadow: inset 0 -4px 10px rgba(0,0,0,0.04); }

	/* center exam titles, dates, sentence and buttons */
	.doubling-exam { align-items: center; }
	.doubling-exam .group-title { text-align:center; }
	.doubling-exam input[type="date"] { text-align:center; }

	/* center Copy and Effacer with identical width and margins */
	.copy-row, .actions { display:flex; justify-content:center; max-width:14rem; margin:0.5rem auto 0; gap:0.6rem; }

	.doubling-results .result .value { display:flex; justify-content:center; align-items:center; }

	.doubling-results .result .title { text-align: center; }

	.vol-input {
		width: 6.4rem;
		padding: 0.25rem 0.4rem;
		border: 1px solid var(--md-default-fg-color--lighter);
		border-radius: 0.4rem;
		background: var(--md-code-bg-color);
		font-size: 0.8rem;
		text-align: center;
	}

	.doubling-main-result .value {
		font-weight: 600;
		text-align: center;
	}

	/* center the main result vertically and horizontally */
	.doubling-main-result {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		min-height: 3.2rem;
	}

	.doubling-main-result .subvalue {
		text-align: center;
		font-size: 0.85rem;
		margin-top: 0.25rem;
		color: inherit;
	}



	.doubling-sentence {
		margin-top: 0.6rem;
	}

	.doubling-sentence .value { text-align: center; }

	.doubling-sentence .wide {
		grid-column: 1 / -1;
	}

	.copy-row {
		display: flex;
		align-items: center;
		gap: 0.6rem;
	}

	.copy {
		border: 1px solid var(--md-default-fg-color--lighter);
		background: transparent;
		border-radius: 0.5rem;
		padding: 0.35rem 0.7rem;
		cursor: pointer;
		font-size: 0.8rem;
	}

	.copy:disabled {
		cursor: not-allowed;
		opacity: 0.6;
	}

	.copied {
		font-size: 0.8rem;
		opacity: 0.8;
	}

	@media (max-width: 700px) {
		.doubling-grid {
			grid-template-columns: 1fr;
		}

		.doubling-dimensions {
			grid-template-columns: 1fr;
		}
	}
</style>

<script>
function doublingNum(value){
	if (!value) return NaN;
	return Number.parseFloat(String(value).replace(/\s/g, '').replace(',', '.'));
}

function doublingDate(value){
	if (!value) return NaN;
	const parts = value.split('-').map(Number);
	return parts.length === 3 ? Date.UTC(parts[0], parts[1] - 1, parts[2]) : NaN;
}

// removed: previous function for 3-axis volume calculation

function formatDoublingVolume(value){
	return Number.isFinite(value) ? String(Math.round(value)) : '—';
}



function formatDoublingDate(value){
	const [year, month, day] = value.split('-');
	return `${day}/${month}/${year}`;
}

function resetDoublingSentence(){
	document.getElementById('doubling-sentence-text').textContent = 'Volume lésionnel estimé à — cc contre — cc le —/—/—, soit un temps de doublement de — jours.';
	document.getElementById('copy-doubling-btn').disabled = true;
}

function computeDoublingTime(){
	const date1 = doublingDate(document.getElementById('doubling-date-1').value);
	const date2 = doublingDate(document.getElementById('doubling-date-2').value);
	// read mode: 'volume' or 'diameter'
	const mode = document.getElementById('mode-diameter').checked ? 'diameter' : 'volume';
	const volInput1 = document.getElementById('doubling-vol-1');
	const volInput2 = document.getElementById('doubling-vol-2');
	const raw1 = doublingNum(volInput1.value);
	const raw2 = doublingNum(volInput2.value);

	// interpret inputs according to mode
	let volume1 = NaN, volume2 = NaN;
	if (mode === 'diameter') {
		// inputs are diameters (same units as entered)
		if (Number.isFinite(raw1) && raw1 > 0) volume1 = (Math.PI / 6) * Math.pow(raw1, 3) / 1000; // mm^3->cc if diameter in mm
		if (Number.isFinite(raw2) && raw2 > 0) volume2 = (Math.PI / 6) * Math.pow(raw2, 3) / 1000;
		// update titles
		document.querySelectorAll('.doubling-results .result')[0].querySelector('.title').textContent = 'Diamètre actuel (mm)';
		document.querySelectorAll('.doubling-results .result')[1].querySelector('.title').textContent = 'Diamètre initial (mm)';
	} else {
		// volume mode: inputs are volumes in cc
		if (Number.isFinite(raw1) && raw1 > 0) volume1 = raw1;
		if (Number.isFinite(raw2) && raw2 > 0) volume2 = raw2;
		// restore titles
		document.querySelectorAll('.doubling-results .result')[0].querySelector('.title').textContent = 'Volume actuel (cc)';
		document.querySelectorAll('.doubling-results .result')[1].querySelector('.title').textContent = 'Volume initial (cc)';
	}
	// update input title according to mode
	volInput1.title = mode === 'diameter' ? 'Saisir diamètre en mm' : 'Saisir volume en cc';
	volInput2.title = mode === 'diameter' ? 'Saisir diamètre en mm' : 'Saisir volume en cc';
	const result = document.getElementById('doubling-result');
	const subTextEl = document.getElementById('doubling-subtext');

	// display input placeholders/values: keep inputs as entered (diameter or volume)
	result.textContent = '—';
	subTextEl.textContent = '';

	// no immediate update on diameter input — sentence updates only after full compute

	// require all numeric values to compute doubling time
	if (![date1, date2, volume1, volume2].every(Number.isFinite)) return;

	// compute intervalDays early to enforce the 10000 days limit
	const intervalDays = (date2 - date1) / 86400000;
	if (intervalDays >= 10000) {
		// do not compute doubling time for excessively long intervals
		result.textContent = '—';
		subTextEl.textContent = '';
		document.getElementById('copy-doubling-btn').disabled = true;
		return;
	}
	if (date2 <= date1) {
		// report error in the main sentence area; keep subtext empty
		document.getElementById('doubling-sentence-text').textContent = 'La date de contrôle doit être postérieure à la date initiale.';
		document.getElementById('copy-doubling-btn').disabled = true;
		subTextEl.textContent = '';
		return;
	}
	if (Math.abs(volume2 - volume1) < 1e-9) {
		document.getElementById('doubling-sentence-text').textContent = 'Volume stable : pas de doublement mesurable sur cet intervalle.';
		document.getElementById('copy-doubling-btn').disabled = true;
		subTextEl.textContent = '';
		return;
	}

	const doublingDays = intervalDays * Math.log(2) / Math.log(volume2 / volume1);
	if (doublingDays < 0) {
		const pctNeg = Math.round(((volume2 - volume1) / volume1) * 100);
		const signNeg = pctNeg > 0 ? '+' : '';
		// subtext contains only the percent change and interval
		subTextEl.textContent = `(${signNeg}${pctNeg}% en ${Math.round(intervalDays)} jours)`;
		// update main sentence
		document.getElementById('doubling-sentence-text').textContent = `Volume lésionnel estimé à ${formatDoublingVolume(volume2)} cc contre ${formatDoublingVolume(volume1)} cc le ${formatDoublingDate(document.getElementById('doubling-date-1').value)}.`;
		document.getElementById('copy-doubling-btn').disabled = false;
		return;
	}

	const roundedDoublingDays = Math.round(doublingDays);
	const months = Math.round(doublingDays / 30.44);
	result.textContent = `${roundedDoublingDays} jours = ${months} mois`;


	// afficher uniquement "+/- ...% en ... jours" dans la subtext
	const pct = Math.round(((volume2 - volume1) / volume1) * 100);
	const sign = pct > 0 ? '+' : '';
	subTextEl.textContent = `(${sign}${pct}% en ${Math.round(intervalDays)} jours)`;

	document.getElementById('doubling-sentence-text').textContent = `Volume lésionnel estimé à ${formatDoublingVolume(volume2)} cc contre ${formatDoublingVolume(volume1)} cc le ${formatDoublingDate(document.getElementById('doubling-date-1').value)}, soit un temps de doublement de ${roundedDoublingDays} jours.`;
	document.getElementById('copy-doubling-btn').disabled = false;
}

function copyDoublingSentence(){
	const sentence = document.getElementById('doubling-sentence-text').textContent;
	const button = document.getElementById('copy-doubling-btn');
	const message = document.getElementById('doubling-copied');

	if (button.disabled) return;

	const showCopied = () => {
		message.textContent = 'Copié ✓';
		window.setTimeout(() => {
			message.textContent = '';
		}, 1500);
	};

	if (navigator.clipboard?.writeText) {
		navigator.clipboard.writeText(sentence).then(showCopied).catch(() => {});
		return;
	}

	const textarea = document.createElement('textarea');
	textarea.value = sentence;
	document.body.appendChild(textarea);
	textarea.select();
	try { document.execCommand('copy'); showCopied(); } catch (error) {}
	document.body.removeChild(textarea);
}

function clearDoublingTime(){
	// clear all inputs except set the control date (Examen actuel) to today
	document.querySelectorAll('#volume-doubling-time input').forEach(input => {
		if (input.id === 'doubling-date-2') return;
		if (input.type === 'date') { input.value = ''; return; }
		input.value = '';
	});
	const today = new Date();
	const todayStr = [today.getFullYear(), String(today.getMonth() + 1).padStart(2, '0'), String(today.getDate()).padStart(2, '0')].join('-');
	document.getElementById('doubling-date-2').value = todayStr;
	computeDoublingTime();
}

const today = new Date();
document.getElementById('doubling-date-2').value = [
	today.getFullYear(),
	String(today.getMonth() + 1).padStart(2, '0'),
	String(today.getDate()).padStart(2, '0')
].join('-');

// Reset volume/diameter input fields and the copy sentence when mode changes
['mode-volume', 'mode-diameter'].forEach(id => {
	const el = document.getElementById(id);
	if (!el) return;
	el.addEventListener('change', () => {
		const v1 = document.getElementById('doubling-vol-1');
		const v2 = document.getElementById('doubling-vol-2');
		if (v1) v1.value = '';
		if (v2) v2.value = '';
		resetDoublingSentence();
		computeDoublingTime();
	});
});
</script>
