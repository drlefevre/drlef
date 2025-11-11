# Volumes

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

<script>
/* ==== Fonctions globales, simples et sans dépendance ==== */
function vol_num(v){
  if (!v) return NaN;
  v = String(v).replace(/\s/g,'').replace(',', '.');
  return Number.parseFloat(v);
}

function computeVolumes(){
  const d1 = vol_num(document.getElementById('vol-d1').value);
  const d2 = vol_num(document.getElementById('vol-d2').value);
  const d3 = vol_num(document.getElementById('vol-d3').value);

  let ellip = NaN, lamb = NaN;
  if ([d1,d2,d3].every(Number.isFinite)) {
    ellip = (Math.PI/6) * d1 * d2 * d3 / 1000;  // mm³ -> cc
    lamb  = (0.71) * d1 * d2 * d3 / 1000;       // mm³ -> cc
  }

  document.getElementById('v-ellip').textContent   = Number.isFinite(ellip) ? Math.round(ellip).toString() : '—';
  document.getElementById('v-lambert').textContent = Number.isFinite(lamb)  ? Math.round(lamb).toString()  : '—';
}

function clearVolumes(){
  document.getElementById('vol-d1').value = '';
  document.getElementById('vol-d2').value = '';
  document.getElementById('vol-d3').value = '';
  computeVolumes();
}
</script>