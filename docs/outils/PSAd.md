# [Densité de PSA](https://radiopaedia.org/articles/psa-density){:target="_blank"}

<div class="box md-typeset" id="psad-box">
  <form onsubmit="return false;" oninput="psadCompute()">
    <div class="pairs" style="margin-top:.4rem">
      <div class="pair">
        <input id="psa" type="text" inputmode="decimal" placeholder="PSA (ng/ml)" />
        <input id="vol" type="text" inputmode="decimal" placeholder="Volume prostatique (cc)" />
      </div>
    </div>

    <div class="results">
      <div class="result wide">
        <div class="value" id="psad-phrase">—</div>
        <div class="copy-row">
          <button type="button" class="copy" id="psad-copy" disabled>Copier</button>
          <span class="copied" id="psad-copied" aria-live="polite"></span>
        </div>
      </div>
    </div>

    <div class="actions">
      <button type="button" class="clear" onclick="psadClear()">Effacer</button>
    </div>
  </form>
</div>

<script>
// ====== Utilitaires ======
function psadNum(v){
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
function psadCompute(){
  const psa = psadNum(document.getElementById('psa').value);
  const vol = psadNum(document.getElementById('vol').value);

  const phraseEl= document.getElementById('psad-phrase');
  const copyBtn = document.getElementById('psad-copy');

  let phraseHtml = '—';
  let canCopy = false;

  if (isPos(psa) && isPos(vol)){
    const psad = (psa / vol).toFixed(2).replace('.', ',');
    let core = `Densité de PSA = ${psad} ng/mL/cc, `;

    if (psa/vol > 0.15) {
      core += `suspecte.`;
    } else {
      core += `peu suspecte.`;
    }

    phraseHtml = core;
    canCopy = true;
  }

  phraseEl.innerHTML = phraseHtml;
  copyBtn.disabled = !canCopy;
}

// Effacer
function psadClear(){
  ['psa','vol'].forEach(id => document.getElementById(id).value='');
  psadCompute();
}

// Copier la phrase
(function(){
  const btn = document.getElementById('psad-copy');
  const msg = document.getElementById('psad-copied');
  function showCopied(){ msg.textContent = 'Copié ✓'; setTimeout(()=> msg.textContent='', 1500); }
  function getPlainText(){
    const src = document.getElementById('psad-phrase');
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
psadCompute();
</script>

<style>
.box {
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
.result.wide { grid-column: 1 / -1; }
.copy-row { display:flex; align-items:center; gap:.6rem; margin-top:.35rem; }
.copy { border:1px solid var(--md-default-fg-color--lighter); background:transparent; border-radius:.5rem; padding:.35rem .7rem; cursor:pointer; }
.copied { font-size:.8rem; opacity:.8; }
.note { display:inline-block; margin-top:.25rem; opacity:.85; font-style: italic; }
</style>

```
Intérêt d'un contrôle du taux de PSA et d'une IRM en cas d'évolution défavorable.
```