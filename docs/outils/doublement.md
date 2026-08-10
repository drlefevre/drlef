# Temps de doublement

<div class="box md-typeset" id="volume-doubling-time">
	<form onsubmit="return false;" oninput="computeDoublingTime()">
		<div class="doubling-grid">
			<div class="doubling-exam">
				<div class="group-title">Examen actuel</div>
				<input id="doubling-date-2" type="date" />
				<div class="doubling-dimensions">
					<input id="doubling-d1-2" type="text" inputmode="decimal" placeholder="mm" />
					<input id="doubling-d2-2" type="text" inputmode="decimal" placeholder="mm" />
					<input id="doubling-d3-2" type="text" inputmode="decimal" placeholder="mm" />
				</div>
			</div>

			<div class="doubling-exam">
				<div class="group-title">Examen initial</div>
				<input id="doubling-date-1" type="date" />
				<div class="doubling-dimensions">
					<input id="doubling-d1-1" type="text" inputmode="decimal" placeholder="mm" />
					<input id="doubling-d2-1" type="text" inputmode="decimal" placeholder="mm" />
					<input id="doubling-d3-1" type="text" inputmode="decimal" placeholder="mm" />
				</div>
			</div>
		</div>

		<div class="results doubling-results">
			<div class="result">
				<div class="title">Volume actuel</div>
				<div class="value"><span id="doubling-volume-2">—</span> cc</div>
			</div>
			<div class="result">
				<div class="title">Volume initial</div>
				<div class="value"><span id="doubling-volume-1">—</span> cc</div>
			</div>
			<div class="result doubling-main-result">
				<div class="title">Temps de doublement</div>
				<div class="value"><span id="doubling-result">—</span></div>
			</div>
		</div>

		<div class="doubling-sentence">
			<div class="result wide">
				<div class="value" id="doubling-sentence-text">Volume lésionnel estimé à — cc contre — cc le —/—/—, soit un temps de doublement de — jours.</div>
				<div class="copy-row">
					<button id="copy-doubling-btn" type="button" class="copy" onclick="copyDoublingSentence()" disabled>Copier</button>
					<span class="copied" id="doubling-copied" aria-live="polite"></span>
				</div>
			</div>
		</div>

		<div class="actions">
			<div class="doubling-note" id="doubling-note" aria-live="polite">
			</div>
			<button type="button" class="clear" onclick="clearDoublingTime()">Effacer</button>
		</div>
	</form>
</div>

<style>
    .actions { 
        margin: 0.45rem 0 0.35rem; 
		flex-direction: column;
		align-items: flex-start;
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

	.doubling-main-result .value {
		font-weight: 600;
	}

	.doubling-note {
		font-size: 0.8rem;
		line-height: 1.35;
		margin: 0 0 0.35rem;
	}

	.doubling-sentence {
		margin-top: 0.6rem;
	}

	.doubling-sentence .wide {
		grid-column: 1 / -1;
	}

	.copy-row {
		display: flex;
		align-items: center;
		gap: 0.6rem;
		margin-top: 0.35rem;
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

function doublingVolume(prefix){
	const dimensions = [1, 2, 3].map(index => doublingNum(document.getElementById(`doubling-d${index}-${prefix}`).value));
	return dimensions.every(Number.isFinite) && dimensions.every(value => value > 0)
		? (Math.PI / 6) * dimensions[0] * dimensions[1] * dimensions[2] / 1000
		: NaN;
}

function formatDoublingVolume(value){
	return Number.isFinite(value) ? value.toFixed(value < 10 ? 2 : 1).replace('.', ',') : '—';
}

function formatVolumeRatio(value){
	return value.toFixed(2).replace('.', ',');
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
	const volume1 = doublingVolume('1');
	const volume2 = doublingVolume('2');
	const result = document.getElementById('doubling-result');
	const note = document.getElementById('doubling-note');

	document.getElementById('doubling-volume-1').textContent = formatDoublingVolume(volume1);
	document.getElementById('doubling-volume-2').textContent = formatDoublingVolume(volume2);
	result.textContent = '—';
	note.textContent = '';
	resetDoublingSentence();

	if (![date1, date2, volume1, volume2].every(Number.isFinite)) return;
	if (date2 <= date1) {
		note.textContent = 'La date de contrôle doit être postérieure à la date initiale.';
		return;
	}
	if (volume2 === volume1) {
		note.textContent = 'Volume stable : pas de doublement mesurable sur cet intervalle.';
		return;
	}

	const intervalDays = (date2 - date1) / 86400000;
	const doublingDays = intervalDays * Math.log(2) / Math.log(volume2 / volume1);
	if (doublingDays < 0) {
		note.textContent = `Volume en diminution ; rapport volumique = ${formatVolumeRatio(volume2 / volume1)}.`;
		return;
	}

	const roundedDoublingDays = Math.round(doublingDays);
	result.textContent = `${roundedDoublingDays} jours (${(doublingDays / 30.44).toFixed(1).replace('.', ',')} mois)`;
	note.textContent = `Intervalle : ${Math.round(intervalDays)} jours ; rapport volumique = ${formatVolumeRatio(volume2 / volume1)}.`;
	document.getElementById('doubling-sentence-text').textContent = `Volume lésionnel estimé à ${formatDoublingVolume(volume2)} cc contre ${formatDoublingVolume(volume1)} cc le ${formatDoublingDate(document.getElementById('doubling-date-1').value)} soit un temps de doublement de ${roundedDoublingDays} jours.`;
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
	document.querySelectorAll('#volume-doubling-time input').forEach(input => {
		input.value = '';
	});
	computeDoublingTime();
}
</script>
