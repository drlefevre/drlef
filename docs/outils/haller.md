# Indice de [Haller](https://radiopaedia.org/articles/haller-index?lang=gb){:target="_blank"}

<figure markdown="span">
    ![](assets/haller.jpg){width="350"}
</figure>

<div class="box md-typeset" id="haller-box">
  <form onsubmit="return false;" oninput="hallerCompute()">
    <div class="pairs" style="margin-top:.4rem">
      <div class="pair">
        <input id="a" type="text" inputmode="decimal" placeholder="A (mm)" />
        <input id="b" type="text" inputmode="decimal" placeholder="B (mm)" />
      </div>
    </div>

    <div class="results">
      <div class="result wide">
        <div class="value" id="haller-phrase">—</div>
        <div class="copy-row">
          <button type="button" class="copy" id="haller-copy" disabled>Copier</button>
          <span class="copied" id="haller-copied" aria-live="polite"></span>
        </div>
      </div>
    </div>

    <div class="actions">
      <button type="button" class="clear" onclick="hallerClear()">Effacer</button>
    </div>
  </form>
</div>

<script>
// ====== Utilitaires ======
function hallerNum(v){
  if (!v) return NaN;
  v = String(v).replace(/\s/g,'').replace(',', '.');
  return Number.parseFloat(v);
}
function fmtFr1(x){
  if(!Number.isFinite(x)) return '—';
  return x.toFixed(1).replace('.', ','); // 1 décimale, virgule
}
function isPos(n){ return Number.isFinite(n) && n > 0; }

// ====== Calcul principal ======
function hallerCompute(){
  const aRaw = hallerNum(document.getElementById('a').value);
  const bRaw = hallerNum(document.getElementById('b').value);

  const phraseEl= document.getElementById('haller-phrase');
  const copyBtn = document.getElementById('haller-copy');

  let phraseHtml = '—';
  let canCopy = false;

  if (isPos(aRaw) && isPos(bRaw)){
    const idx = aRaw / bRaw;

    // Règles :
    // <2 : "Indice de Haller = x,y."
    // 2–<3.2 : léger ; 3.2–≤3.5 : modéré ; >3.5 : sévère
    // + note italique si ≥3.25
    let core = '';
    if (idx < 2){
      core = `Indice de Haller = ${fmtFr1(idx)}.`;
    } else if (idx < 3.2){
      core = `Pectus excavatum léger avec indice de Haller = ${fmtFr1(idx)}.`;
    } else if (idx <= 3.5){
      core = `Pectus excavatum modéré avec indice de Haller = ${fmtFr1(idx)}.`;
    } else {
      core = `Pectus excavatum sévère avec indice de Haller = ${fmtFr1(idx)}.`;
    }

    if (idx >= 3.25){
      phraseHtml = core + '<br><span class="note"><em>Intérêt d\'un avis chirurgical.</em></span>';
    } else {
      phraseHtml = core;
    }
    canCopy = true;
  }

  phraseEl.innerHTML = phraseHtml;
  copyBtn.disabled = !canCopy;
}

// Effacer
function hallerClear(){
  ['a','b'].forEach(id => document.getElementById(id).value='');
  hallerCompute();
}

// Copier la phrase (sans la note)
(function(){
  const btn = document.getElementById('haller-copy');
  const msg = document.getElementById('haller-copied');
  function showCopied(){ msg.textContent = 'Copié ✓'; setTimeout(()=> msg.textContent='', 1500); }
  function getPlainText(){
    const src = document.getElementById('haller-phrase');
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
hallerCompute();
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
}
.result.wide { grid-column: 1 / -1; }
.copy-row { display:flex; align-items:center; gap:.6rem; margin-top:.35rem; }
.copy { border:1px solid var(--md-default-fg-color--lighter); background:transparent; border-radius:.5rem; padding:.35rem .7rem; cursor:pointer; }
.copied { font-size:.8rem; opacity:.8; }
.note { display:inline-block; margin-top:.25rem; opacity:.85; font-style: italic; }
</style>