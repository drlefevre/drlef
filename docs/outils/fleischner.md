# [Fleischner 2017](https://pubs.rsna.org/doi/10.1148/radiol.2017161659){:target="_blank"}

<div class="calc-mini md-typeset" id="fleischner">
  <form onsubmit="return false;">
    <div class="row4">
      <input id="diam" type="text" inputmode="decimal" placeholder="Diam. moyen (mm)" />
      <select id="type">
        <option value="solid" selected>Solide</option>
        <option value="part">Mixte</option>
        <option value="ggn">Verre dépoli</option>
      </select>
      <select id="count">
        <option value="single" selected>Unique</option>
        <option value="multiple">Multiples</option>
      </select>
      <select id="risk">
        <option value="low">Risque bas</option>
        <option value="high" selected>Risque élevé</option>
      </select>
    </div>

    <div class="box">
      <div class="value" id="rec">—</div>
      <div class="copy-row">
        <button class="copy" type="button" id="copyBtn" disabled>Copier</button>
        <span class="copied" id="copiedMsg" aria-live="polite"></span>
      </div>
    </div>
  </form>
</div>

<style>
.calc-mini {
  max-width: 920px;
  margin: 1rem 0 2rem;
  padding: 1rem 1rem .75rem;
  border: 1px solid var(--md-default-fg-color--lightest);
  border-radius: .9rem;
  background: var(--md-default-bg-color);
}
.row4 { display:grid; grid-template-columns: 1.2fr 1fr 1fr 1fr; gap:.6rem; }
.calc-mini input, .calc-mini select {
  width:100%; padding:.55rem .65rem;
  border:1px solid var(--md-default-fg-color--lighter);
  border-radius:.7rem; background: var(--md-code-bg-color);
  font-size: .8rem;
}

.box { margin-top:.7rem; margin-bottom: 0; border:1px dashed var(--md-default-fg-color--lighter); border-radius:.7rem; padding:.75rem .9rem; }

.copy-row { display:flex; align-items:center; gap:.75rem; margin-top:.5rem; }
.copy { border:1px solid var(--md-default-fg-color--lighter); background:transparent; border-radius:.6rem; padding:.35rem .7rem; cursor:pointer; }
.copied { font-size:.8rem; opacity:.8; }

</style>

<script>
(function () {
  const $ = (id) => document.getElementById(id);
  const rec = $('rec'), copyBtn = $('copyBtn'), copiedMsg = $('copiedMsg');

  function num(v){ if(!v) return NaN; v = String(v).replace(/\s/g,'').replace(',', '.'); return Number.parseFloat(v); }
  function fmtMm(x){
    if(!Number.isFinite(x)) return '';
    const isInt = Math.abs(x - Math.round(x)) < 1e-9;
    return (isInt ? Math.round(x).toString() : (Math.round(x*10)/10).toFixed(1)).replace('.', ',');
  }
  function band(mm){ if(!Number.isFinite(mm) || mm<=0) return null; if(mm<6) return 'lt6'; if(mm<=8) return '6to8'; return 'gt8'; }
  function capFirst(s){ return s ? s.charAt(0).toUpperCase() + s.slice(1) : s; }

  // Recommandations (texte de l'action après "pour lequel[s]")
  function actSolid(b, count, risk){
    if(b==='lt6'){
      return risk==='high'
        ? "il peut être proposé de réaliser un contrôle TDM à 12 mois"
        : "aucun suivi n’est recommandé";
    }
    if(b==='6to8'){
      if(count==='single') return risk==='high'
        ? "il est recommandé de réaliser un contrôle TDM dans 6 à 12 mois puis à 18 à 24 mois"
        : "il est recommandé de réaliser un contrôle TDM dans 6 à 12 mois, puis d’envisager un contrôle à 18 à 24 mois";
      return risk==='high'
        ? "il est recommandé de réaliser un contrôle TDM dans 3 à 6 mois puis à 18 à 24 mois"
        : "il est recommandé de réaliser un contrôle TDM dans 3 à 6 mois, puis d’envisager un contrôle à 18 à 24 mois";
    }
    // >8 mm
    if(count==='single')
      return "il est recommandé une réévaluation à court terme (~3 mois) avec TDM et discussion TEP-TDM et/ou biopsie selon le contexte";
    return "il est recommandé de réaliser un contrôle TDM dans 3 à 6 mois, puis d’envisager un contrôle à 18 à 24 mois";
  }

  function actGGN(b, count){
    if(b==='lt6'){
      return count==='single'
        ? "aucun suivi n’est recommandé"
        : "il est recommandé de réaliser un contrôle TDM dans 3 à 6 mois ; si stable, d’envisager des contrôles à 2 et 4 ans";
    }
    if(count==='single')
      return "il est recommandé de réaliser un contrôle TDM dans 6 à 12 mois pour confirmer la persistance, puis tous les 2 ans jusqu’à 5 ans ; en cas de croissance ou de densification, envisager une résection";
    return "il est recommandé de réaliser un contrôle TDM dans 3 à 6 mois ; la suite de la prise en charge sera guidée par le nodule le plus suspect";
  }

  function actPart(b, count){
    if(b==='lt6'){
      return count==='single'
        ? "aucun suivi n’est recommandé"
        : "il est recommandé de réaliser un contrôle TDM dans 3 à 6 mois ; si stable, d’envisager des contrôles à 2 et 4 ans";
    }
    if(count==='single')
      return "il est recommandé de réaliser un contrôle TDM dans 3 à 6 mois pour confirmer la persistance ; si inchangé et composante solide < 6 mm : contrôle annuel pendant 5 ans ; si composante solide ≥ 6 mm : suspicion élevée, discussion spécialisée";
    return "il est recommandé de réaliser un contrôle TDM dans 3 à 6 mois ; la suite de la prise en charge sera guidée par le nodule le plus suspect";
  }

  function buildSentence(d, type, count, action){
    const typeLbl = type==='solid' ? "solide"
                   : type==='part' ? "mixte"
                                    : "en verre dépoli";
    if(count==='single'){
      const head = `Nodule pulmonaire ${typeLbl} unique de ${fmtMm(d)} mm de diamètre moyen`;
      return `${head} pour lequel ${action} (selon la recommandation Fleischner 2017).`;
    }
    // Multiples
    const sentence1 = `Multiples nodules pulmonaires, dont le plus volumineux est ${typeLbl} et mesure ${fmtMm(d)} mm de diamètre moyen.`;
    return `${sentence1} ${capFirst(action)} (selon Fleischner 2017).`;
  }

  function compute(){
    const d = num($('diam').value);
    const t = $('type').value, c = $('count').value, r = $('risk').value;
    const b = band(d);
    if(!b){ rec.textContent = "—"; copyBtn.disabled = true; return; }

    const action = (t==='solid') ? actSolid(b, c, r)
                   : (t==='ggn') ? actGGN(b, c)
                                 : actPart(b, c);

    rec.textContent = buildSentence(d, t, c, action);
    copyBtn.disabled = false;
  }

  function copyText(){
    const text = rec.textContent;
    if(!text || text==='—') return;
    if (navigator.clipboard?.writeText) {
      navigator.clipboard.writeText(text).then(()=>showCopied(), ()=>fallbackCopy(text));
    } else { fallbackCopy(text); }
  }
  function fallbackCopy(text){
    const ta = document.createElement('textarea'); ta.value = text;
    document.body.appendChild(ta); ta.select(); try{ document.execCommand('copy'); }catch(e){}
    document.body.removeChild(ta); showCopied();
  }
  function showCopied(){ copiedMsg.textContent = "Copié ✓"; setTimeout(()=> copiedMsg.textContent="", 1500); }

  ['diam','type','count','risk'].forEach(id => $(id).addEventListener('input', compute));
  $('copyBtn').addEventListener('click', copyText);

  // Rendez ce gestionnaire tolérant si le bouton .clear n'existe pas dans le DOM
  const clearBtn = document.querySelector('#fleischner .clear');
  if (clearBtn) {
    clearBtn.addEventListener('click', ()=>{
      $('diam').value=''; $('type').value='solid'; $('count').value='single'; $('risk').value='low';
      compute();
    });
  }

  compute();
})();
</script>

!!! info "**Risque élevé** si tabagisme, antécédent familial de cancer du poumon, emphysème, fibrose pulmonaire, âge avancé"

!!! warning "**Non applicable** si < 35 ans, antécédent de cancer, immunodépression, dépistage du cancer du poumon, nodule sous-pleural ou péri-scissural"

<figure markdown="span">  
    [CAT nodule pulmonaire selon Lederlin](https://cerf.radiologie.fr/sites/cerf.radiologie.fr/files/Enseignement/DES/Archives-Documents/02%20ML%20CAT%20d%C3%A9couverte%20fortuite%20nodule%20pulmonaire.pdf){:target="_blank"}  
    [Algorithme dépistage selon la SIT](https://www.radiologie.fr/sites/www.radiologie.fr/files/medias/documents/Algorithme%20dépistage%20cancer%20bronchique%20validé.pdf){:target="_blank"}  
</figure>