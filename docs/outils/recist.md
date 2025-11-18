# [RECIST 1.1](https://recist.eortc.org/recist-1-1-2/){:target="_blank"}

<div class="box md-typeset" id="recist-sum">
  <form onsubmit="return false;" oninput="recistCompute()">
    <div class="pairs" style="margin-top:.4rem">
      <div class="pair"><input id="l1prev" type="text" inputmode="decimal" placeholder="L1 baseline/nadir (mm)" /><input id="l1now" type="text" inputmode="decimal" placeholder="L1 actuelle (mm)" /></div>
      <div class="pair"><input id="l2prev" type="text" inputmode="decimal" placeholder="L2 baseline/nadir (mm)" /><input id="l2now" type="text" inputmode="decimal" placeholder="L2 actuelle (mm)" /></div>
      <div class="pair"><input id="l3prev" type="text" inputmode="decimal" placeholder="L3 baseline/nadir (mm)" /><input id="l3now" type="text" inputmode="decimal" placeholder="L3 actuelle (mm)" /></div>
      <div class="pair"><input id="l4prev" type="text" inputmode="decimal" placeholder="L4 baseline/nadir (mm)" /><input id="l4now" type="text" inputmode="decimal" placeholder="L4 actuelle (mm)" /></div>
      <div class="pair"><input id="l5prev" type="text" inputmode="decimal" placeholder="L5 baseline/nadir (mm)" /><input id="l5now" type="text" inputmode="decimal" placeholder="L5 actuelle (mm)" /></div>
    </div>

    <div class="results">
      <div class="result">
        <div class="title">Somme</div>
        <div class="value"><span id="recist-sum-now">—</span> mm</div>
      </div>
      <div class="result">
        <div class="title">Évolution</div>
        <div class="value"><span id="recist-delta">—</span> %</div>
      </div>
      <div class="result wide">
        <div class="title">Interprétation</div>
        <div class="value" id="recist-phrase">—</div>
        <div class="copy-row">
          <button type="button" class="copy" id="recist-copy" disabled>Copier</button>
          <span class="copied" id="recist-copied" aria-live="polite"></span>
        </div>
      </div>
    </div>

    <div class="actions">
      <button type="button" class="clear" onclick="recistClear()">Effacer</button>
    </div>
  </form>
</div>


<script>
// ====== Utilitaires ======
function recistNum(v){
  if (!v) return NaN;
  v = String(v).replace(/\s/g,'').replace(',', '.');
  return Number.parseFloat(v);
}
function recistFmtInt(x){
  if(!Number.isFinite(x)) return '—';
  return Math.round(x).toString();
}

// utilitaire : <10 mm => 0
function recistClamp(v){
  return (Number.isFinite(v) && v >= 10) ? v : 0;
}

// ====== Calcul principal ======
function recistCompute(){
  const ids = ['1','2','3','4','5'];

  let sumPrev = 0, nPrev = 0;
  let sumNow  = 0, nNow  = 0;

  ids.forEach(i => {
    const pRaw = recistNum(document.getElementById('l'+i+'prev').value);
    const nRaw = recistNum(document.getElementById('l'+i+'now').value);

    const p = recistClamp(pRaw);
    const n = recistClamp(nRaw);

    if (p > 0){ sumPrev += p; nPrev++; }
    if (n > 0){ sumNow  += n; nNow++;  }
  });

  // Affichage somme actuelle
  document.getElementById('recist-sum-now').textContent =
    (nNow>0) ? recistFmtInt(sumNow) : '—';

  // % d'évolution (entier) vs somme précédente (seulement si référence valable)
  const deltaEl = document.getElementById('recist-delta');
  let deltaPct = NaN, deltaInt = NaN, diffAbs = NaN;

  if (nPrev>0 && sumPrev>0 && nNow>0){
    diffAbs  = sumNow - sumPrev;          // mm
    deltaPct = (diffAbs / sumPrev) * 100; // %
    deltaInt = Math.round(deltaPct);      // entier
  }
  deltaEl.textContent = Number.isFinite(deltaInt) ? String(deltaInt) : '—';

  // Interprétation RECIST 1.1
  const phraseEl = document.getElementById('recist-phrase');
  const copyBtn  = document.getElementById('recist-copy');
  let phraseHtml = '—';
  let canCopy = false;

  if (Number.isFinite(deltaPct) && Number.isFinite(diffAbs)){
    const isPD = (deltaPct >= 20) && (diffAbs >= 5); // PD: +≥20% ET +≥5 mm
    const isPR = (deltaPct <= -30);                  // PR: -≥30%
    const isSD = !isPD && !isPR;

    if (isPD){
      phraseHtml = 'Progression lésionnelle selon les critères RECIST 1.1.';
    } else if (isPR){
      phraseHtml = 'Réponse partielle selon les critères RECIST 1.1.'
                 + '<br><span class="note">Sauf nouvelle lésion ou progression non équivoque des lésions non cibles.</span>';
    } else if (isSD){
      phraseHtml = 'Stabilité lésionnelle selon les critères RECIST 1.1.'
                 + '<br><span class="note">Sauf nouvelle lésion ou progression non équivoque des lésions non cibles.</span>';
    }
    canCopy = true;
  }

  phraseEl.innerHTML = phraseHtml;
  copyBtn.disabled = !canCopy;
}

// Effacer
function recistClear(){
  ['1','2','3','4','5'].forEach(i=>{
    document.getElementById('l'+i+'prev').value='';
    document.getElementById('l'+i+'now').value='';
  });
  recistCompute();
}

// Copier la phrase (sans la note)
(function(){
  const btn = document.getElementById('recist-copy');
  const msg = document.getElementById('recist-copied');
  function showCopied(){ msg.textContent = 'Copié ✓'; setTimeout(()=> msg.textContent='', 1500); }
  function getPlainText(){
    const src = document.getElementById('recist-phrase');
    const clone = src.cloneNode(true);
    clone.querySelectorAll('.note').forEach(n=>n.remove());
    clone.querySelectorAll('br').forEach(b=>b.remove());
    const text = clone.textContent || clone.innerText || '';
    return text.trim();
  }
  function fallbackCopy(text){
    const ta = document.createElement('textarea'); ta.value = text;
    document.body.appendChild(ta); ta.select(); try{ document.execCommand('copy'); }catch(e){}
    document.body.removeChild(ta); showCopied();
  }
  function copy(){
    const text = getPlainText();
    if(!text || btn.disabled) return;
    if (navigator.clipboard?.writeText) {
      navigator.clipboard.writeText(text).then(()=>showCopied(), ()=>fallbackCopy(text));
    } else { fallbackCopy(text); }
  }
  btn.addEventListener('click', copy);
})();

// init
recistCompute();
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

!!! warning "lésion cible ssi tumeur de plus grand diamètre ≥ 10 mm ou gg de petit axe ≥ 15 mm"

!!! info "[Radiology Assistant](https://staging.radiologyassistant.nl/more/recist-1-1/recist-1-1-1){:target="_blank"}"
    - ↬ 5 lésions cibles, max. 2 par organe et max. 2 gg
    - somme des plus grands diamètres et des petits axes pour les gg
    - progression ↗ ≥ 20% vs nadir / réponse partielle ↘ ≥ 30% vs baseline
    - lésions non cibles si < 10 mm, gg 10-15 mm, lésions non mesurables