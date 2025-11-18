# Critères de [Cheson](https://cerf.radiologie.fr/sites/cerf.radiologie.fr/files/files/Enseignement/pdf/09%20Imagerie%20lymphome_DES_2019.pdf){:target="_blank"}

<div class="box md-typeset" id="cheson-spd">
  <form onsubmit="return false;" oninput="chesonCompute()">
    <div class="row1">
      <input id="spd0" type="text" inputmode="decimal" placeholder="SPD baseline/nadir (mm²)" />
    </div>

    <div class="pairs" style="margin-top:.4rem">
      <div class="pair"><input id="l1d1" type="text" inputmode="decimal" placeholder="L1 D1 (mm)" /><input id="l1d2" type="text" inputmode="decimal" placeholder="L1 D2 (mm)" /></div>
      <div class="pair"><input id="l2d1" type="text" inputmode="decimal" placeholder="L2 D1 (mm)" /><input id="l2d2" type="text" inputmode="decimal" placeholder="L2 D2 (mm)" /></div>
      <div class="pair"><input id="l3d1" type="text" inputmode="decimal" placeholder="L3 D1 (mm)" /><input id="l3d2" type="text" inputmode="decimal" placeholder="L3 D2 (mm)" /></div>
      <div class="pair"><input id="l4d1" type="text" inputmode="decimal" placeholder="L4 D1 (mm)" /><input id="l4d2" type="text" inputmode="decimal" placeholder="L4 D2 (mm)" /></div>
      <div class="pair"><input id="l5d1" type="text" inputmode="decimal" placeholder="L5 D1 (mm)" /><input id="l5d2" type="text" inputmode="decimal" placeholder="L5 D2 (mm)" /></div>
      <div class="pair"><input id="l6d1" type="text" inputmode="decimal" placeholder="L6 D1 (mm)" /><input id="l6d2" type="text" inputmode="decimal" placeholder="L6 D2 (mm)" /></div>
    </div>

    <div class="results">
      <div class="result">
        <div class="title">SPD</div>
        <div class="value"><span id="cheson-spd-now">—</span> mm²</div>
      </div>
      <div class="result">
        <div class="title">Évolution</div>
        <div class="value"><span id="cheson-delta">—</span> %</div>
      </div>
      <div class="result wide">
        <div class="title">Interprétation</div>
        <div class="value" id="cheson-phrase">—</div>
        <div class="copy-row">
          <button type="button" class="copy" id="cheson-copy" disabled>Copier</button>
          <span class="copied" id="cheson-copied" aria-live="polite"></span>
        </div>
      </div>
    </div>

    <div class="actions">
      <button type="button" class="clear" onclick="chesonClear()">Effacer</button>
    </div>
  </form>
</div>

<script>
// ====== Utilitaires ======
function chesonNum(v){
  if (!v) return NaN;
  v = String(v).replace(/\s/g,'').replace(',', '.');
  return Number.parseFloat(v);
}
function chesonFmtInt(x){
  if(!Number.isFinite(x)) return '—';
  return Math.round(x).toString();
}
function chesonFmtPctValue(x){
  if(!Number.isFinite(x)) return NaN;
  return Math.round(x); // entier
}
function chesonPctToStr(val){
  if(!Number.isFinite(val)) return '—';
  return Math.round(val) + '%';
}

// ====== Calcul principal ======
function chesonCompute(){
  const spd0 = chesonNum(document.getElementById('spd0').value); // mm²
  const ids = ['1','2','3','4','5','6'];
  let spdNow = 0; let nUsed = 0;
  ids.forEach(i => {
    const a = chesonNum(document.getElementById('l'+i+'d1').value);
    const b = chesonNum(document.getElementById('l'+i+'d2').value);
    if (Number.isFinite(a) && Number.isFinite(b) && a>0 && b>0){
      spdNow += a*b; nUsed++;
    }
  });
  if(nUsed===0) spdNow = NaN;

  // Affichage SPD actuel
  document.getElementById('cheson-spd-now').textContent = chesonFmtInt(spdNow);

  // Pourcentage d'évolution
  let delta = NaN;
  if (Number.isFinite(spd0) && spd0>0 && Number.isFinite(spdNow)){
    delta = ((spdNow - spd0) / spd0) * 100;
  }
  const deltaVal = chesonFmtPctValue(delta);
  document.getElementById('cheson-delta').textContent = Number.isFinite(deltaVal) ? String(deltaVal) : '—';

  // Interprétation (SPD) : PR ≤ -50 %, SD (-50 ; +50), PD ≥ +50 %
  const phraseEl = document.getElementById('cheson-phrase');
  const copyBtn = document.getElementById('cheson-copy');
  let phraseHtml = '—';
  let canCopy = false;

  if (Number.isFinite(deltaVal)){
    const absPct = Math.abs(deltaVal);
    const isIncrease = deltaVal > 0;

    let cat = 'Stabilité lésionnelle';
    if (deltaVal >= 50) cat = 'Progression lésionnelle';
    else if (deltaVal <= -50) cat = 'Réponse partielle';

    phraseHtml = `${cat} selon les critères de Cheson.`;

    // Note additionnelle pour la RP (italique, non copiée)
    if (deltaVal <= -50){
      phraseHtml += `<br><span class=\"note\">Sauf nouvelle atteinte ganglionnaire ou augmentation en taille des autres ganglions, et seulement si les nodules parenchymateux ont régressé d\u2019au moins 50% (SPD).</span>`;
    }
    else if (deltaVal <= 50){
      phraseHtml += `<br><span class=\"note\">⚠ ↗ SPD ≥ 50% d’un seul ganglion suffit pour parler de progression.</span>`;
    }
    canCopy = true;
  }

  phraseEl.innerHTML = phraseHtml;
  copyBtn.disabled = !canCopy;
}

function chesonClear(){
  document.getElementById('spd0').value = '';
  ['1','2','3','4','5','6'].forEach(i=>{
    document.getElementById('l'+i+'d1').value='';
    document.getElementById('l'+i+'d2').value='';
  });
  chesonCompute();
}

// Copier la phrase (sans la note)
(function(){
  const btn = document.getElementById('cheson-copy');
  const msg = document.getElementById('cheson-copied');
  function showCopied(){ msg.textContent = 'Copié \u2713'; setTimeout(()=> msg.textContent='', 1500); }
  function getPlainText(){
    const src = document.getElementById('cheson-phrase');
    const clone = src.cloneNode(true);
    // Supprimer la note et les éventuels sauts de ligne
    clone.querySelectorAll('.note').forEach(n=>n.remove());
    clone.querySelectorAll('br').forEach(b=>b.remove());
    const text = clone.textContent || clone.innerText || '';
    return text.trim();
  }
  function copy(){
    const text = getPlainText();
    if(!text || btn.disabled) return;
    if (navigator.clipboard?.writeText) {
      navigator.clipboard.writeText(text).then(()=>showCopied(), ()=>fallbackCopy(text));
    } else { fallbackCopy(text); }
  }
  function fallbackCopy(text){
    const ta = document.createElement('textarea'); ta.value = text;
    document.body.appendChild(ta); ta.select(); try{ document.execCommand('copy'); }catch(e){}
    document.body.removeChild(ta); showCopied();
  }
  btn.addEventListener('click', copy);
})();

// init
chesonCompute();
</script>

<style>
.box {
  max-width: 820px;
  margin: 1rem 0 2rem;
  padding: 1rem 1rem .5rem;
  border: 1px solid var(--md-default-fg-color--lightest);
  border-radius: .75rem;
  background: var(--md-default-bg-color);
}
.row1 { display:grid; grid-template-columns: 1fr; gap:.6rem; }
.pairs { display:grid; grid-template-columns: 1fr; gap:.45rem; }
.pair { display:grid; grid-template-columns: repeat(2, 1fr); gap:.6rem; }
.box input {
  width: 100%;
  padding: .55rem .65rem;
  border: 1px solid var(--md-default-fg-color--lighter);
  border-radius: .5rem;
  background: var(--md-code-bg-color);
  font-size: .8rem;
}
.results {
  display:grid; grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap:.75rem; margin:.6rem 0 .6rem;
}
.result { border:1px dashed var(--md-default-fg-color--lighter); border-radius:.5rem; padding:.6rem .8rem; }
.result.wide { grid-column: 1 / -1; }
.result .value { font-size:.8rem; line-height:1.35; margin-top:.3rem; }
.copy-row { display:flex; align-items:center; gap:.6rem; margin-top:.35rem; }
.copy { border:1px solid var(--md-default-fg-color--lighter); background:transparent; border-radius:.5rem; padding:.35rem .7rem; cursor:pointer; }
.copied { font-size:.8rem; opacity:.8; }
.actions { margin:.25rem 0 .5rem; display:flex; align-items:center; gap:.75rem; flex-wrap:wrap; }
.actions button { font-size:.8rem; border:1px solid var(--md-default-fg-color--lighter); background:transparent; border-radius:.5rem; padding:.4rem .7rem; cursor:pointer; }
.note { display:inline-block; margin-top:.25rem; opacity:.85; font-style: italic; }
</style>


!!! info "Bilan initial"
    - ↬ 6 lésions cibles = gg > 15 mm de grand axe et lésions extra-gg > 10 mm
    - médiastin bulky si > 7 cm ou > 1/3 diamètre = retentissement trachée / VCS ?
    - hypertrophie splénique si > 13 cm
    - lésions ostéolytiques ?