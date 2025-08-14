# Ellipsoïde

<div class="box md-typeset" data-formula="ellipsoide">
  <form onsubmit="return false;">
    <div class="row3">
      <input type="text" inputmode="decimal" placeholder="mm" />
      <input type="text" inputmode="decimal" placeholder="mm" />
      <input type="text" inputmode="decimal" placeholder="mm" />
    </div>

    <div class="results">
      <div class="result">
        <div class="formula">largeur × épaisseur × hauteur × π/6 (0,52)</div>
        <div class="value"><span class="out">—</span> cc</div>
      </div>
    </div>

    <div class="actions">
      <button class="clear" type="button">Effacer</button>
    </div>
  </form>
</div>
</br>

# Lambert

<div class="box md-typeset" data-formula="lambert">
  <form onsubmit="return false;">
    <div class="row3">
      <input type="text" inputmode="decimal" placeholder="mm" />
      <input type="text" inputmode="decimal" placeholder="mm" />
      <input type="text" inputmode="decimal" placeholder="mm" />
    </div>

    <div class="results">
      <div class="result">
        <div class="formula">longueur × épaisseur × largeur × 0,71</div>
        <div class="value"><span class="out">—</span> cc</div>
      </div>
    </div>

    <div class="actions">
      <button class="clear" type="button">Effacer</button>
    </div>
  </form>
</div>


<script>
(function () {
  function num(v) {
    if (!v) return NaN;
    v = String(v).replace(/\s/g,'').replace(',', '.');
    return Number.parseFloat(v);
  }

  function computeFor(container) {
    const [i1,i2,i3] = container.querySelectorAll('input');
    const outEl = container.querySelector('.out');
    const mode = container.getAttribute('data-formula'); // "ellipsoide" | "lambert"

    const a = num(i1.value), b = num(i2.value), c = num(i3.value);
    let Vcc = NaN;

    if ([a,b,c].every(Number.isFinite)) {
      if (mode === 'ellipsoide') {
        // V(mm³) = (π/6) * L * E * H  -> cc
        Vcc = (Math.PI/6) * a * b * c / 1000;
      } else {
        // Lambert: V(mm³) = L * E * l * 0.71  -> cc
        Vcc = (a * b * c * 0.71) / 1000;
      }
    }

    // 0 décimale en cc
    outEl.textContent = Number.isFinite(Vcc) ? Math.round(Vcc).toString() : '—';
  }

  function attach(container) {
    const inputs = container.querySelectorAll('input');
    const clearBtn = container.querySelector('.clear');
    inputs.forEach(inp => inp.addEventListener('input', () => computeFor(container)));
    clearBtn.addEventListener('click', () => { inputs.forEach(inp => inp.value = ''); computeFor(container); });
    computeFor(container);
  }

  document.querySelectorAll('.box').forEach(attach);
})();
</script>