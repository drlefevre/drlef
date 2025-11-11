# Dose efficace en TDM

<div class="box md-typeset" id="ct-dose-unified">
  <form onsubmit="return false;">
    <div class="pairs" style="margin-top:.4rem">
      <!-- Ligne de contrôle en 3 parties -->
      <div class="pair three">
        <!-- 1) Étage : TAP par défaut dans le menu -->
        <div>
          <select id="uni-stage" aria-label="Étage">
            <option value="crane">Crâne</option>
            <option value="tap" selected>TAP</option>
            <option value="thorax">Thorax</option>
            <option value="abdpelv">AP</option>
          </select>
        </div>

        <!-- 2) Mode : DLP totale (défaut) ou Nb d'acquisitions -->
        <div>
          <select id="uni-mode" aria-label="Mode de calcul">
            <option value="acq">Nb d’acquisitions</option>
            <option value="dlp" selected>DLP totale</option>
          </select>
        </div>

        <!-- 3) Saisie -->
        <div>
          <input id="uni-value" type="text" inputmode="decimal" placeholder="mGy.cm" />
        </div>
      </div>
    </div>

    <div class="results">
      <div class="result">
        <div class="title">
          <a id="uni-source-label" href="https://www.sprmn.pt/pdf/RP154.pdf" target="_blank" rel="noopener">
            Selon les constantes RP154
          </a>
        </div>
        <div class="value"><span id="uni-dose">—</span> mSv</div>
      </div>
      <div class="result">
        <div class="title">Années d'exposition naturelle</div>
        <div class="value"><span id="uni-natural">—</span></div>
      </div>
    </div>

    <div class="actions">
      <button type="button" class="clear" id="uni-clear">Effacer</button>
    </div>
  </form>
</div>

<script>
(function(){
  // ====== Constantes (adultes 60–80 kg) ======
  // Mode "acq" (par acquisition) — dérivé IRSN
  const DOSE_PER_ACQ = { tap:16.2, thorax:6, abdpelv:9.5, crane:2.0 };
  // Mode "dlp" (E = k × DLP) — RP154/IRSN
  const K = { tap:0.015, thorax:0.014, abdpelv:0.015, crane:0.0021 };

  const NATURAL_ANNUAL_MSV = 2.4; // mSv/an

  // ====== Utils ======
  const $ = sel => document.querySelector(sel);
  function numFr(v){ if(!v) return NaN; return parseFloat(String(v).replace(/\s/g,'').replace(',','.')); }

  function fmtAdaptive(x, threshold = 2){
    if(!Number.isFinite(x)) return '—';
    return (x > threshold) ? String(Math.round(x)) : x.toFixed(1).replace('.', ',');
  }

  function updateSourceLabel(mode){
    const a = $('#uni-source-label');
    if(!a) return;
    if(mode === 'dlp'){
      a.textContent = 'Selon les constantes RP154';
      a.href = 'https://www.sprmn.pt/pdf/RP154.pdf';
    } else {
      a.textContent = 'Selon IRSN adultes (60-80 kg)';
      a.href = 'https://www.irsn.fr/sites/default/files/documents/expertise/rapports_expertise/IRSN-Rapport-dosimetrie-patient-2010-12.pdf';
    }
  }

  function compute(){
    const stage = $('#uni-stage').value;
    const mode  = $('#uni-mode').value;
    const raw   = numFr($('#uni-value').value);

    let E = NaN;
    if (mode === 'dlp'){
      if (Number.isFinite(raw) && raw>0) E = (K[stage]||0) * raw; // mSv
    } else {
      const n = Number.isFinite(raw) ? Math.max(0, Math.floor(raw)) : NaN;
      if (Number.isFinite(n) && n>0) E = (DOSE_PER_ACQ[stage]||0) * n;
    }

    $('#uni-dose').textContent    = Number.isFinite(E) ? fmtAdaptive(E, 2) : '—';
    $('#uni-natural').textContent = Number.isFinite(E) ? fmtAdaptive(E / NATURAL_ANNUAL_MSV, 2) : '—';
  }

  function setModeUI(mode){
    $('#uni-mode').value = mode;
    const inp = $('#uni-value');
    if (mode === 'dlp'){
      inp.placeholder = 'mGy.cm';
      inp.setAttribute('inputmode','decimal');
    } else {
      inp.placeholder = '';
      inp.setAttribute('inputmode','numeric');
    }
    updateSourceLabel(mode);
    compute();
  }

  // Listeners
  $('#uni-stage').addEventListener('change', compute);
  $('#uni-mode').addEventListener('change', e => setModeUI(e.target.value));
  $('#uni-value').addEventListener('input', compute);
  $('#uni-clear').addEventListener('click', () => {
    $('#uni-stage').value = 'tap';
    setModeUI('dlp');
    $('#uni-value').value = '';
    compute();
  });

  // Init
  setModeUI('dlp');  // DLP par défaut → RP154
  compute();
})();
</script>

<style>
.box {
  max-width: 920px;
  margin: 1rem 0 2rem;
  padding: 1rem 1rem .75rem;
  border: 1px solid var(--md-default-fg-color--lightest);
  border-radius: .9rem;
  background: var(--md-default-bg-color);
}
.pairs { display:grid; grid-template-columns: 1fr; gap:.6rem; }
.pair { display:grid; grid-template-columns: repeat(2, 1fr); gap:.6rem; }
.pair.three { grid-template-columns: repeat(3, 1fr); }
.box input, .box select {
  width: 100%;
  padding: .75rem .9rem;
  border: 1px solid var(--md-default-fg-color--lighter);
  border-radius: .7rem;
  background: var(--md-code-bg-color);
}
/* Unifie la hauteur des champs dans le bloc unifié */
#ct-dose-unified input,
#ct-dose-unified select {
  box-sizing: border-box;
  height: 2rem;
  padding: 0 .65rem;
  font-size: .8rem;
  line-height: 1.2;
}
</style>

<figure markdown="span">
    [Seuil de survenue d’un excès de cancer](https://pmc.ncbi.nlm.nih.gov/articles/PMC283495/pdf/10013761.pdf){:target="_blank"} :  
    10 et 50 mSv pour une exposition aiguë  
    50-100 mSv pour une exposition cumulée
</figure>

</br></br>

|  [Dose reçue à l'utérus en TDM](https://onclepaul.fr/wp-content/uploads/2011/07/La-femme-enceinte-en-imagerie-pire-angoisse-du-radiologue-New-JFR-2020.pdf){:target="_blank"}| mGy |
| :----------: | :-------: |
| `Crâne` | 0,005 |
| `Thorax` | 0,06 |
| `Abdomen` | 8 |
| `Pelvis` | 25 |

!!! info "Risque malformatif si dose **>100–200 mGy**. Loi du tout ou rien ↬ S2"
!!! warning "TDM thoracique = risque cancer sein 0,005%/mSv"