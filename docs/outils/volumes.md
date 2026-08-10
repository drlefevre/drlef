# Volumes

<div class="box md-typeset" id="lambert-volumes">
  <form onsubmit="return false;" oninput="computeLambertVolumes()">
    <div class="row6">
      <div class="group">
        <div class="group-title">Droit</div>
        <div class="row3">
          <input id="lambert-d1-r" type="text" inputmode="decimal" placeholder="mm" />
          <input id="lambert-d2-r" type="text" inputmode="decimal" placeholder="mm" />
          <input id="lambert-d3-r" type="text" inputmode="decimal" placeholder="mm" />
        </div>
      </div>
      <div class="group">
        <div class="group-title">Gauche</div>
        <div class="row3">
          <input id="lambert-d1-l" type="text" inputmode="decimal" placeholder="mm" />
          <input id="lambert-d2-l" type="text" inputmode="decimal" placeholder="mm" />
          <input id="lambert-d3-l" type="text" inputmode="decimal" placeholder="mm" />
        </div>
      </div>
    </div>

    <div class="results">
      <div class="result">
        <div class="value"><span id="v-lambert-right">—</span> cc</div>
      </div>
      <div class="result">
        <div class="value"><span id="v-lambert-left">—</span> cc</div>
      </div>
    </div>

    <div class="sentence-block">
      <div class="result wide">
        <div class="value" id="lambert-sentence">Volumes testiculaires estimés selon la formule de Lambert à — cc à droite et à — cc à gauche (normal entre 12 et 20 cc).</div>
        <div class="copy-row">
          <button id="copy-lambert-btn" type="button" class="copy" onclick="copyLambertSentence()" disabled>Copier</button>
          <span class="copied" id="lambert-copied" aria-live="polite"></span>
        </div>
      </div>
    </div>

    <div class="actions">
      <button type="button" class="clear" onclick="clearLambertVolumes()">Effacer</button>
    </div>
  </form>
</div>

<div class="box md-typeset" id="volumes">
  <form onsubmit="return false;" oninput="computeVolumes()">
    <div class="row3">
      <input id="vol-d1" type="text" inputmode="decimal" placeholder="mm" />
      <input id="vol-d2" type="text" inputmode="decimal" placeholder="mm" />
      <input id="vol-d3" type="text" inputmode="decimal" placeholder="mm" />
    </div>

    <div class="results">
      <div class="result">
        <div class="title">Ellipsoïde (× 0,52)</div>
        <div class="value"><span id="v-ellip">—</span> cc</div>
      </div>
      <div class="result">
        <div class="title">Lambert (× 0,71)</div>
        <div class="value"><span id="v-lambert">—</span> cc</div>
      </div>
    </div>

    <div class="actions">
      <button type="button" class="clear" onclick="clearVolumes()">Effacer</button>
    </div>
  </form>
</div>

<style>
  .row3 {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 0.75rem;
    margin-bottom: 0.75rem;
  }

  .row6 {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
    margin-bottom: 0.5rem;
  }

  .group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .group-title {
    font-weight: 600;
  }

  .results {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 0.75rem;
    margin: 0.4rem 0 0.4rem;
  }

  .result {
    border: 1px dashed var(--md-default-fg-color--lighter);
    border-radius: 0.5rem;
    padding: 0.6rem 0.8rem;
  }

  .result .value {
    font-size: 0.8rem;
    line-height: 1.35;
    margin-top: 0.3rem;
  }

  .sentence-block {
    margin-top: 0.6rem;
  }

  .result.wide {
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

  .actions {
    margin: 0.45rem 0 0.35rem;
    display: flex;
    align-items: center;
    gap: 0.75rem;
    flex-wrap: wrap;
  }

  .actions button {
    font-size: 0.8rem;
    border: 1px solid var(--md-default-fg-color--lighter);
    background: transparent;
    border-radius: 0.5rem;
    padding: 0.4rem 0.7rem;
    cursor: pointer;
  }
</style>

<script>
/* ==== Fonctions globales, simples et sans dépendance ==== */
function vol_num(v){
  if (!v) return NaN;
  v = String(v).replace(/\s/g,'').replace(',', '.');
  return Number.parseFloat(v);
}

function roundVolume(v){
  return Number.isFinite(v) ? Math.round(v).toString() : '—';
}

function computeVolumes(){
  const d1 = vol_num(document.getElementById('vol-d1').value);
  const d2 = vol_num(document.getElementById('vol-d2').value);
  const d3 = vol_num(document.getElementById('vol-d3').value);

  let ellip = NaN;
  let lamb = NaN;
  if ([d1,d2,d3].every(Number.isFinite)) {
    ellip = (Math.PI/6) * d1 * d2 * d3 / 1000;  // mm³ -> cc
    lamb = 0.71 * d1 * d2 * d3 / 1000;         // mm³ -> cc
  }

  document.getElementById('v-ellip').textContent = roundVolume(ellip);
  document.getElementById('v-lambert').textContent = roundVolume(lamb);
}

function computeLambertVolumes(){
  const right = ['lambert-d1-r','lambert-d2-r','lambert-d3-r'].map(id => vol_num(document.getElementById(id).value));
  const left = ['lambert-d1-l','lambert-d2-l','lambert-d3-l'].map(id => vol_num(document.getElementById(id).value));

  let rightVol = NaN;
  let leftVol = NaN;

  if (right.every(Number.isFinite)) {
    rightVol = 0.71 * right[0] * right[1] * right[2] / 1000;
  }

  if (left.every(Number.isFinite)) {
    leftVol = 0.71 * left[0] * left[1] * left[2] / 1000;
  }

  document.getElementById('v-lambert-right').textContent = roundVolume(rightVol);
  document.getElementById('v-lambert-left').textContent = roundVolume(leftVol);

  const rightText = Number.isFinite(rightVol) ? Math.round(rightVol).toString() : '—';
  const leftText = Number.isFinite(leftVol) ? Math.round(leftVol).toString() : '—';
  const sentence = `Volumes testiculaires selon la formule de Lambert estimés à ${rightText} cc à droite et ${leftText} cc à gauche (normal entre 12 et 20 cc).`;
  document.getElementById('lambert-sentence').textContent = sentence;

  const copyBtn = document.getElementById('copy-lambert-btn');
  copyBtn.disabled = !Number.isFinite(rightVol) || !Number.isFinite(leftVol);
}

function copyLambertSentence(){
  const sentence = document.getElementById('lambert-sentence').textContent;
  const button = document.getElementById('copy-lambert-btn');
  const msg = document.getElementById('lambert-copied');

  if (!sentence || sentence.includes('—') || button.disabled) {
    return;
  }

  const doCopy = (text) => {
    if (navigator.clipboard?.writeText) {
      navigator.clipboard.writeText(text).then(() => {
        msg.textContent = 'Copié ✓';
        window.setTimeout(() => {
          msg.textContent = '';
        }, 1500);
      }).catch(() => {
        msg.textContent = 'Échec';
        window.setTimeout(() => {
          msg.textContent = '';
        }, 1500);
      });
    } else {
      const ta = document.createElement('textarea');
      ta.value = text;
      document.body.appendChild(ta);
      ta.select();
      try { document.execCommand('copy'); } catch (e) {}
      document.body.removeChild(ta);
      msg.textContent = 'Copié ✓';
      window.setTimeout(() => {
        msg.textContent = '';
      }, 1500);
    }
  };

  doCopy(sentence);
}

function clearVolumes(){
  document.getElementById('vol-d1').value = '';
  document.getElementById('vol-d2').value = '';
  document.getElementById('vol-d3').value = '';
  computeVolumes();
}

function clearLambertVolumes(){
  ['lambert-d1-r','lambert-d2-r','lambert-d3-r','lambert-d1-l','lambert-d2-l','lambert-d3-l'].forEach((id) => {
    document.getElementById(id).value = '';
  });
  computeLambertVolumes();
}
</script>