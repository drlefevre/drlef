# [MR parkinsonism index](https://radiopaedia.org/articles/magnetic-resonance-parkinsonism-index-1){:target="_blank"}

<figure markdown="span">
    ![](assets/mrpi.jpg){width="350"}
</figure>

<div class="box md-typeset" id="mrpi-box">
  <form onsubmit="return false;" oninput="mrpiCompute()">
    <div class="pairs" style="margin-top:.4rem">
      <div class="pair">
        <input id="mrpi-pons" type="text" inputmode="decimal" placeholder="Surface du pont (mm²)" />
        <input id="mrpi-midbrain" type="text" inputmode="decimal" placeholder="Surface du mésencéphale (mm²)" />
      </div>
      <div class="pair">
        <input id="mrpi-mcp" type="text" inputmode="decimal" placeholder="Largeur du PCM (mm)" />
        <input id="mrpi-scp" type="text" inputmode="decimal" placeholder="Largeur du PCS (mm)" />
      </div>
    </div>

    <div class="results">
      <div class="result wide">
        <div class="value" id="mrpi-phrase">—</div>
        <div class="copy-row">
          <button type="button" class="copy" id="mrpi-copy" disabled>Copier</button>
          <span class="copied" id="mrpi-copied" aria-live="polite"></span>
        </div>
      </div>
    </div>

    <div class="actions">
      <button type="button" class="clear" onclick="mrpiClear()">Effacer</button>
    </div>
  </form>
</div>

<script>
// ====== Utilitaires ======
function mrpiNum(v){
  if (!v) return NaN;
  v = String(v).replace(/\s/g,'').replace(',', '.');
  return Number.parseFloat(v);
}
function mrpiFmtFr2(x){
  if(!Number.isFinite(x)) return '—';
  return x.toFixed(2).replace('.', ',');
}
function mrpiIsPos(n){ return Number.isFinite(n) && n > 0; }

// ====== Calcul principal ======
function mrpiCompute(){
  const pons = mrpiNum(document.getElementById('mrpi-pons').value);
  const midbrain = mrpiNum(document.getElementById('mrpi-midbrain').value);
  const mcp = mrpiNum(document.getElementById('mrpi-mcp').value);
  const scp = mrpiNum(document.getElementById('mrpi-scp').value);

  const phraseEl = document.getElementById('mrpi-phrase');
  const copyBtn = document.getElementById('mrpi-copy');

  let phrase = '—';
  let canCopy = false;

  if (mrpiIsPos(pons) && mrpiIsPos(midbrain) && mrpiIsPos(mcp) && mrpiIsPos(scp)){
    const mrpi = (pons / midbrain) * (mcp / scp);
    phrase = `MR parkinsonism index = ${mrpiFmtFr2(mrpi)} (suspect de PSP si ≥ 13,55).`;
    canCopy = true;
  }

  phraseEl.textContent = phrase;
  copyBtn.disabled = !canCopy;
}

// Effacer
function mrpiClear(){
  ['mrpi-pons','mrpi-midbrain','mrpi-mcp','mrpi-scp'].forEach(id => {
    document.getElementById(id).value = '';
  });
  mrpiCompute();
}

// Copier la phrase
(function(){
  const btn = document.getElementById('mrpi-copy');
  const msg = document.getElementById('mrpi-copied');
  function showCopied(){ msg.textContent = 'Copié ✓'; setTimeout(() => msg.textContent = '', 1500); }
  function fallbackCopy(text){
    const ta = document.createElement('textarea');
    ta.value = text;
    document.body.appendChild(ta);
    ta.select();
    try{ document.execCommand('copy'); }catch(e){}
    document.body.removeChild(ta);
    showCopied();
  }
  function copy(){
    const text = document.getElementById('mrpi-phrase').textContent.trim();
    if(!text || btn.disabled) return;
    if (navigator.clipboard?.writeText) {
      navigator.clipboard.writeText(text).then(() => showCopied(), () => fallbackCopy(text));
    } else {
      fallbackCopy(text);
    }
  }
  btn.addEventListener('click', copy);
})();

// init
mrpiCompute();
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
</style>