# Calcul d'une sténose selon [NASCET](https://www.ahajournals.org/doi/epdf/10.1161/01.STR.22.6.711){:target="_blank"}

<figure markdown="span">
    ![](assets/nascet.jpg){width="300"}
</figure>

<div class="box md-typeset">
  <form onsubmit="return false;">
    <div class="row2">
        <input id="a" type="text" inputmode="decimal" placeholder="A" />
        <input id="b" type="text" inputmode="decimal" placeholder="B" />
    </div>

    <div class="results">
      <div class="result">
        <div class="value"><span id="nascet">—</span> %</div>
      </div>
    </div>

    <div class="actions">
      <button id="clear" type="button">Effacer</button>
    </div>
  </form>
</div>

<script>
(function () {
  const $ = (id) => document.getElementById(id);

  function parseNum(v) {
    if (!v) return NaN;
    // accepte 5,2 ou 5.2
    v = String(v).replace(/\s/g, '').replace(',', '.');
    return Number.parseFloat(v);
  }
  const round1 = (x) => Number.isFinite(x) ? Math.round(x*10)/10 : NaN;
  const clampPct = (x) => Number.isFinite(x) ? Math.max(0, Math.min(100, x)) : NaN;

  function compute() {
    const A = parseNum($('a').value);
    const B = parseNum($('b').value);

    let warn = [];
    if (Number.isFinite(A) && A <= 0) warn.push("A doit être > 0");
    if (Number.isFinite(B) && B < 0)  warn.push("B ne peut pas être négatif");
    if (Number.isFinite(A) && Number.isFinite(B) && B > A) warn.push("B > A (vérifier les mesures)");

    const nascet = clampPct((1 - (B / A)) * 100);
    $('nascet').textContent = Number.isFinite(nascet) ? Math.round(nascet).toString() : '—';
  }

  ['a','b'].forEach(id => $(''+id).addEventListener('input', compute));
  $('clear').addEventListener('click', () => { ['a','b'].forEach(id => $(id).value=''); compute(); });
  compute();
})();
</script>

<figure markdown="span">
    (1 − B / A) × 100  
    50% < sténose modérée < 70% < sévère < 90% < pré-occlusive  
    (vide de flux ARM = > 80% si calibre Ⓝ en aval / > 90% si réduit)  
    = athérome linéaire/sessile/pédiculé ± Ca2+/ulcéré
</figure>