# [Biométrie neuropédiatrique](https://www.neuroradiologie.fr/index.php/outils/corps-calleux){:target="_blank"}

<figure markdown="span">
    ![](assets/biometries.jpg){width="800"}
</figure>

<div class="calc-mini md-typeset" id="neuro">
  <form onsubmit="return false;">
    <!-- Sélecteurs : Années | Mois | Sexe -->
    <div class="row3">
      <input id="years" type="text" inputmode="numeric" pattern="\d*" placeholder="Années" />
      <input id="months" type="text" inputmode="numeric" pattern="\d*" placeholder="Mois" />
      <select id="sex">
        <option value="G" selected>Garçon</option>
        <option value="F">Fille</option>
      </select>
    </div>

    <!-- Résultats -->
    <div class="box">
      <div class="grid-2">
        <!-- Corps calleux -->
        <div class="panel">
          <h4><a href="https://www.ajnr.org/content/ajnr/32/8/1436.full.pdf" target="_blank" rel="noopener">Corps calleux</a></h4>
          <table class="tbl" id="cc-table">
            <thead><tr><th></th><th>3è p.</th><th>97è p.</th></tr></thead>
            <tbody>
              <tr data-key="APD"><td>Grand axe</td><td class="p3"></td><td class="p97"></td></tr>
              <tr data-key="GT"><td>Genou</td><td class="p3"></td><td class="p97"></td></tr>
              <tr data-key="BT"><td>Corps</td><td class="p3"></td><td class="p97"></td></tr>
              <tr data-key="IT"><td>Isthme</td><td class="p3"></td><td class="p97"></td></tr>
              <tr data-key="ST"><td>Splénium</td><td class="p3"></td><td class="p97"></td></tr>
            </tbody>
          </table>
        </div>

        <!-- Fosse postérieure -->
        <div class="panel">
          <h4><a href="https://www.ajnr.org/content/ajnr/early/2019/10/17/ajnr.A6257.full.pdf" target="_blank" rel="noopener">Fosse postérieure</a></h4>
          <table class="tbl" id="cb-table">
            <thead><tr><th></th><th>3è p.</th><th>97è p.</th></tr></thead>
            <tbody>
              <tr data-key="APD-MP"><td>Mésencéphale</td><td class="p3"></td><td class="p97"></td></tr>
              <tr data-key="APD-P"><td>Pont</td><td class="p3"></td><td class="p97"></td></tr>
              <tr data-key="H-V"><td>Vermis H</td><td class="p3"></td><td class="p97"></td></tr>
              <tr data-key="APD-V"><td>Vermis AP</td><td class="p3"></td><td class="p97"></td></tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </form>
</div>

<style>
/* Base identique à l’outil Fleischner */
.calc-mini{
  max-width:920px; margin:1rem 0 2rem;
  padding:1rem 1rem .75rem;
  border:1px solid var(--md-default-fg-color--lightest);
  border-radius:.9rem; background:var(--md-default-bg-color);
}
/* 3 colonnes : années | mois | sexe */
.row3{ display:grid; grid-template-columns:1fr 1fr 1fr; gap:.6rem; }
.calc-mini input, .calc-mini select{
  width:100%; padding:.55rem .65rem;
  border:1px solid var(--md-default-fg-color--lighter);
  border-radius:.7rem; background: var(--md-code-bg-color);
  font-size: .8rem;
}

/* Boîte résultats */
.box{
  margin-top:.7rem; margin-bottom:0;
  border:1px dashed var(--md-default-fg-color--lighter);
  border-radius:.7rem; padding:.6rem .9rem;
  padding-top:.45rem;
}

/* Deux tableaux côte à côte */
.grid-2{ display:grid; grid-template-columns:1fr 1fr; gap:1rem; align-items:start; }
@media (max-width:820px){ .grid-2{ grid-template-columns:1fr; } }

/* Tables */
.tbl{ width:100%; border-collapse:collapse; margin-top:.35rem; }
.tbl th,.tbl td{ border:1px solid var(--md-default-fg-color--lighter); padding:.4rem .5rem; text-align:left; }
.tbl th{ background:rgba(127,127,127,.08); }

/* Liens titres : respecter le thème et hover */
.panel h4 > a{
  color: var(--md-typeset-a-color);
  text-decoration: none; transition: color .15s ease;
}
.panel h4 > a:hover, .panel h4 > a:focus-visible{
  color: var(--md-accent-fg-color);
}
.panel h4{ margin:.15rem 0 .4rem; }
</style>

<script>
(function(){
  // ===== Util =====
  const fmt1 = (x) => Number.isFinite(x) ? x.toFixed(1).replace('.', ',') : '';
  const clamp = (v, lo, hi) => Math.min(Math.max(v, lo), hi);
  const num = (v) => {
    if(v===null || v===undefined || v==='') return NaN;
    v = String(v).replace(/\s/g,'').replace(',', '.');
    const n = Number.parseFloat(v);
    return Number.isFinite(n) ? n : NaN;
  };
  const closestIndex = (val, arr) => {
    if(!Number.isFinite(val)) return -1;
    let best = -1, md = Infinity;
    for(let i=0;i<arr.length;i++){
      const d = Math.abs(arr[i]-val);
      if(d < md){ md = d; best = i; }
    }
    return best;
  };

  // ===== Données =====
  const CC = {
    ages:["0","0.5","1","1.5","2","2.5","3","4","5","6","7","8","9","10","11","12","13","14","15"].map(parseFloat),
    APD:{p3:[36.8,43.7,47.9,50.6,52.4,53.6,54.4,55.5,56.2,56.8,57.4,57.9,58.5,59.1,59.7,60.4,61,61.7,62.5],
         p97:[62,63.9,66.8,69.1,70.6,71.7,72.5,73.5,74.2,74.8,75.4,76,76.6,77.3,78.1,79,79.9,81.1,82.3]},
    GT:{p3:[2.5,3.7,4.6,5.2,5.7,6,6.3,6.7,6.9,7,7.1,7.2,7.3,7.3,7.4,7.5,7.5,7.6,7.6],
        p97:[8.3,8.9,9.7,10.4,11,11.4,11.8,12.3,12.6,12.9,13,13.1,13.2,13.3,13.4,13.5,13.5,13.6,13.7]},
    BT:{p3:[1.3,1.8,2.2,2.6,2.9,3.1,3.3,3.5,3.7,3.8,3.9,3.9,4,4,4,4,3.9,3.9,3.7],
        p97:[5,5.3,5.7,6.1,6.5,6.8,7,7.4,7.6,7.7,7.8,7.9,7.9,8,8,8,8.1,8.2,8.4]},
    IT:{p3:[1.2,1.4,1.5,1.6,1.7,1.7,1.8,1.9,2,2.1,2.2,2.2,2.3,2.4,2.4,2.4,2.5,2.5,2.5],
        p97:[3.9,4.1,4.3,4.5,4.6,4.8,4.9,5.1,5.3,5.5,5.6,5.7,5.8,5.9,6,6,6.1,6.2,6.2]},
    ST:{p3:[1.9,3.4,4.4,5.1,5.6,6,6.2,6.7,6.9,7.2,7.4,7.5,7.6,7.7,7.7,7.7,7.5,7.1,6.3],
        p97:[9,9.2,9.9,10.5,10.9,11.3,11.5,11.9,12.2,12.5,12.7,12.8,13,13.1,13.3,13.5,13.7,14.1,14.8]}
  };

  const AGES_CB = ["0.25","0.5","1","1.5","2","2.5","3","4","5","6","7","8","9","10","11","12","13","14","15"].map(parseFloat);
  const CB = {
    "H-V": {
      F:{p3:[21.99,27.18,31.77,33.97,35.22,35.98,36.46,37.00,37.28,37.47,37.61,37.73,37.84,37.95,38.04,38.13,38.21,38.27,38.34],
         p97:[40.54,42.12,44.24,45.58,46.47,47.10,47.56,48.17,48.56,48.84,49.05,49.21,49.34,49.45,49.54,49.61,49.67,49.72,49.77]},
      G:{p3:[20.94,27.92,33.17,35.53,36.82,37.56,38.02,38.49,38.74,38.93,39.11,39.31,39.53,39.76,40.00,40.24,40.49,40.74,40.99],
         p97:[40.39,42.31,44.79,46.29,47.26,47.92,48.40,49.02,49.43,49.73,49.97,50.18,50.37,50.54,50.71,50.86,51.01,51.16,51.30]}
    },
    "APD-V": {
      F:{p3:[10.88,13.65,16.77,18.47,19.48,20.10,20.49,20.90,21.07,21.14,21.17,21.18,21.18,21.19,21.19,21.19,21.19,21.19,21.19],
         p97:[25.57,26.70,28.26,29.21,29.81,30.19,30.43,30.69,30.80,30.84,30.86,30.87,30.87,30.87,30.87,30.87,30.88,30.88,30.88]},
      G:{p3:[9.67,13.43,17.14,19.04,20.13,20.80,21.21,21.64,21.81,21.89,21.91,21.93,21.93,21.93,21.94,21.94,21.94,21.94,21.94],
         p97:[25.54,26.76,28.40,29.40,30.01,30.40,30.65,30.91,31.01,31.06,31.08,31.08,31.09,31.09,31.09,31.09,31.09,31.09,31.09]}
    },
    "APD-P": {
      F:{p3:[12.58,13.29,14.32,15.02,15.50,15.85,16.11,16.48,16.76,17.00,17.22,17.43,17.64,17.85,18.06,18.26,18.46,18.67,18.87],
         p97:[18.36,18.97,19.89,20.51,20.94,21.25,21.49,21.83,22.08,22.30,22.50,22.69,22.88,23.07,23.26,23.44,23.63,23.82,24.00]},
      G:{p3:[12.99,13.82,14.98,15.71,16.19,16.52,16.75,17.08,17.32,17.55,17.77,17.99,18.22,18.46,18.69,18.93,19.17,19.41,19.65],
         p97:[17.47,18.32,19.56,20.40,21.00,21.44,21.78,22.28,22.65,22.96,23.23,23.49,23.73,23.96,24.19,24.41,24.63,24.84,25.05]}
    },
    "APD-MP": {
      both:{p3:[6.68,6.84,7.13,7.39,7.62,7.82,7.99,8.29,8.52,8.71,8.86,9.00,9.11,9.22,9.33,9.44,9.56,9.69,9.83],
            p97:[10.24,10.42,10.76,11.05,11.30,11.53,11.73,12.06,12.32,12.53,12.71,12.86,12.99,13.11,13.23,13.36,13.49,13.63,13.78]}
    }
  };

  // ===== Logique =====
  const yEl = document.getElementById('years');
  const mEl = document.getElementById('months');
  const sEl = document.getElementById('sex');

  function getDecimalAge(){
    const y = num(yEl.value);
    const m = num(mEl.value);
    if(!Number.isFinite(y) && !Number.isFinite(m)) return NaN;
    const ySafe = Number.isFinite(y) ? clamp(y, 0, 15) : 0;
    const mSafe = Number.isFinite(m) ? clamp(Math.floor(m), 0, 11) : 0;
    return clamp(ySafe + mSafe/12, 0, 15);
  }

  function update(){
    const age = getDecimalAge();
    const sex = sEl.value; // "G" | "F"

    // --- Corps calleux (snap à l'âge publié le + proche) ---
    let ccIdx = -1;
    if(Number.isFinite(age)) ccIdx = closestIndex(age, CC.ages);
    document.querySelectorAll('#cc-table tbody tr').forEach(tr=>{
      const key = tr.getAttribute('data-key');
      const c3 = tr.querySelector('.p3'), c97 = tr.querySelector('.p97');
      if(ccIdx === -1){ c3.textContent=''; c97.textContent=''; }
      else { c3.textContent = fmt1(CC[key].p3[ccIdx]); c97.textContent = fmt1(CC[key].p97[ccIdx]); }
    });

    // --- Fosse postérieure (débute à 3 mois : si <0.25 → vide) ---
    let cbIdx = -1;
    if(Number.isFinite(age) && age >= AGES_CB[0]) cbIdx = closestIndex(age, AGES_CB);
    document.querySelectorAll('#cb-table tbody tr').forEach(tr=>{
      const key = tr.getAttribute('data-key');
      const c3 = tr.querySelector('.p3'), c97 = tr.querySelector('.p97');
      if(cbIdx === -1){ c3.textContent=''; c97.textContent=''; }
      else{
        let p3, p97;
        if(key==='APD-MP'){ p3 = CB[key].both.p3[cbIdx]; p97 = CB[key].both.p97[cbIdx]; }
        else { p3 = CB[key][sex].p3[cbIdx]; p97 = CB[key][sex].p97[cbIdx]; }
        c3.textContent = fmt1(p3); c97.textContent = fmt1(p97);
      }
    });
  }

  [yEl, mEl, sEl].forEach(el => el.addEventListener('input', update));
  update();
})();
</script>