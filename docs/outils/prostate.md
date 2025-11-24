# IRM Prostate

<script src="https://cdnjs.cloudflare.com/ajax/libs/fabric.js/5.3.1/fabric.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">

<div class="box md-typeset" id="prostate-box">
  
  <div class="pairs">
    <div style="grid-column: 1 / -1; margin-bottom: 0.4rem;">
        <input id="psa" type="number" inputmode="decimal" placeholder="PSA (ng/ml)" oninput="updateReport()" />
    </div>

    <div class="pair">
        <select id="vol-method" onchange="toggleVolInputs(); updateReport()">
            <option value="auto" selected>Contourage automatique</option>
            <option value="manual">3 diamètres</option>
        </select>
        <input id="d1" type="number" class="manual-input" inputmode="decimal" placeholder="mm" oninput="updateReport()" style="display:none;"/>
        <input id="vol-auto" type="number" class="auto-input" inputmode="decimal" placeholder="cc" oninput="updateReport()" />
    </div>

    <div class="pair manual-input" id="dims-row-2" style="display:none;">
        <input id="d2" type="number" inputmode="decimal" placeholder="mm" oninput="updateReport()" />
        <input id="d3" type="number" inputmode="decimal" placeholder="mm" oninput="updateReport()" />
    </div>
  </div>

  <div class="canvas-container-wrapper">
      <div class="canvas-container">
          <canvas id="prostateCanvas"></canvas>
          <div class="canvas-controls">
              <button type="button" class="action-btn" onclick="addLesionVisual()">
                <i class="fas fa-plus"></i> Lésion
              </button>
              <button type="button" class="action-btn" onclick="copySchema()">
                <i class="fas fa-copy"></i>
              </button>
          </div>
      </div>
  </div>

  <div id="lesion-rows-container"></div>

  <div class="final-actions">
      <button type="button" class="btn-main-copy" id="btn-copy-full" onclick="copyFullReport()">
         CR + schéma
      </button>
      
      <button type="button" class="btn-reset" onclick="fullReset()">
         Effacer
      </button>
  </div>
</div>

<style>
/* --- Style Global --- */
.box {
  max-width: 820px;
  margin: 1rem 0 2rem;
  padding: 1rem 1rem 1.5rem;
  border: 1px solid var(--md-default-fg-color--lightest);
  border-radius: .75rem;
  background: var(--md-default-bg-color);
  font-family: sans-serif;
}
.pairs { display:grid; grid-template-columns: 1fr; gap:.45rem; }
.pair { display:grid; grid-template-columns: repeat(2, 1fr); gap:.6rem; }

.box input, .box select {
  width: 100%; padding: .4rem .6rem;
  border: 1px solid var(--md-default-fg-color--lighter); 
  border-radius: .4rem; background: var(--md-code-bg-color);
  font-size: .8rem; color: var(--md-default-fg-color);
  margin: 0;
}
.box input:focus, .box select:focus { border-color: var(--md-default-fg-color--light); }

input[type=number]::-webkit-inner-spin-button, 
input[type=number]::-webkit-outer-spin-button { 
  -webkit-appearance: none; margin: 0; 
}
input[type=number] { -moz-appearance: textfield; }

/* Canvas */
.canvas-container-wrapper { display: flex; justify-content: center; margin-top: 1rem; width: 100%; }
.canvas-container {
    position: relative; background: white;
    border: 1px solid var(--md-default-fg-color--lighter);
    border-radius: .5rem; overflow: hidden;
    box-shadow: 0 2px 4px -1px rgba(0,0,0,0.1);
}
.canvas-controls {
    display: flex; justify-content: center; gap: 8px; padding: 5px;
    background: var(--md-code-bg-color);
    border-top: 1px solid var(--md-default-fg-color--lighter);
}
.action-btn {
    border: 1px solid var(--md-default-fg-color--lighter); background: white;
    border-radius: .4rem; padding: .25rem .6rem; cursor: pointer;
    font-size: 0.75rem; display: flex; align-items: center; gap: 4px;
}
.action-btn:hover { background: #f0f0f0; }

/* Lignes Lésions */
#lesion-rows-container { margin-top: 15px; }
.lesion-row {
    background: var(--md-code-bg-color); padding: 4px 6px;
    border-radius: 4px; margin-bottom: 6px;
    border-left: 5px solid #ccc;
    display: flex; align-items: center; gap: 6px;
    flex-wrap: nowrap; 
}
.lesion-name { font-weight: bold; font-size: 0.85rem; min-width: 18px; flex-shrink: 0; }

/* BOUTONS PI-RADS */
.pi-btn {
    width: 24px; height: 24px;
    display: flex; justify-content: center; align-items: center;
    border-radius: 4px; cursor: pointer;
    font-weight: bold; font-family: sans-serif; font-size: .8rem; line-height: 1; 
    opacity: 0.35; transition: 0.2s ease-in-out; 
    color: black; 
    flex-shrink: 0;
    user-select: none;
}
.pi-btn:hover { opacity: 0.7; }
.pi-btn.selected { 
    opacity: 1; transform: scale(1.1); 
    box-shadow: 0 1px 3px rgba(0,0,0,0.3); z-index: 1;
}

/* BOUTONS ZP/ZT */
.zone-btn {
    width: 24px; height: 24px;
    display: flex; justify-content: center; align-items: center;
    border-radius: 4px; cursor: pointer;
    font-weight: bold; font-family: sans-serif; font-size: .7rem;
    background: white; color: black;
    border: 1px solid transparent; 
    transition: 0.2s; flex-shrink: 0;
    opacity: 0.5;
}
.zone-btn:hover { opacity: 0.8; background: #f9f9f9; }
.zone-btn.selected {
    opacity: 1; background: white; color: black;
    box-shadow: 0 1px 3px rgba(0,0,0,0.2);
    transform: scale(1.1); z-index: 1;
    font-weight: 800;
}

.pirads-selector { display: flex; gap: 2px; }
.zone-selector { display: flex; gap: 2px; margin-left: 6px; margin-right: 4px; }

.lesion-inputs-wrapper {
    display: grid; grid-template-columns: 1fr 1fr; gap: 4px;
    flex-grow: 1; 
}
.lesion-input {
    width: 100% !important;
    background: white !important; border: 1px solid #ddd !important;
    height: 30px !important; padding: 0 2px !important; margin: 0 !important;
    font-size: 0.8rem !important; text-align: center;
}

/* Couleurs */
.c-green  { background: #4cd137; }
.c-yellow { background: #c4e538; }
.c-gold   { background: #fbc531; }
.c-orange { background: #e17055; }
.c-red    { background: #e84118; }

.trash-btn {
    border: none; background: transparent; cursor: pointer; 
    color: #555; opacity: 0.5; padding: 0; 
    display: flex; justify-content: center; align-items: center; 
    height: 24px; width: 24px; flex-shrink: 0;
}
.trash-btn:hover { opacity: 1; color: #d63031; }

.final-actions {
    margin-top: 1.5rem; display: flex; flex-direction: column; align-items: center; gap: 12px;
}
.btn-main-copy {
    background-color: #f0f0f0; border: 1px solid #ccc; color: #333;
    padding: 10px 20px; font-size: 1rem; font-weight: bold;
    border-radius: 6px; cursor: pointer; transition: all 0.2s;
    display: flex; align-items: center; gap: 8px;
}
.btn-main-copy:hover {
    background-color: #e0e0e0; border-color: #bbb;
    transform: translateY(-1px); box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.btn-reset {
    background: transparent; border: none; color: #888;
    cursor: pointer; font-size: 0.85rem; text-decoration: underline;
}
.btn-reset:hover { color: #d63031; }

#report-text { display: none; }
</style>

<textarea id="report-text"></textarea>

<script>
// --- CONFIGURATION ---
const PIRADS_CONF = [
    { k: '1',   lbl: '1',  v: 1, ord: 10, c: '#4cd137', cls: 'c-green' },
    { k: '2',   lbl: '2',  v: 2, ord: 20, c: '#c4e538', cls: 'c-yellow' },
    { k: '2+1', lbl: '+1', v: 3, ord: 29, c: '#fbc531', cls: 'c-gold' }, 
    { k: '3',   lbl: '3',  v: 3, ord: 30, c: '#fbc531', cls: 'c-gold' },
    { k: '3+1', lbl: '+1', v: 4, ord: 39, c: '#e17055', cls: 'c-orange' }, 
    { k: '4',   lbl: '4',  v: 4, ord: 40, c: '#e17055', cls: 'c-orange' },
    { k: '5',   lbl: '5',  v: 5, ord: 50, c: '#e84118', cls: 'c-red' }
];

const TARGET_WIDTH = 320; 
let CANVAS_HEIGHT = 400; 
let canvas;
let uniqueIdCounter = 0; 
let lesions = []; 
let currentReportData = {}; 

window.addEventListener('load', function() {
    initCanvas();
    toggleVolInputs(); 
    updateReport(); 
});

function initCanvas() {
    canvas = new fabric.Canvas('prostateCanvas');
    const imagePath = '../assets/prostate.jpg'; 
    fabric.Image.fromURL(imagePath, function(img) {
        if(!img) {
            fabric.Image.fromURL('assets/prostate.jpg', function(img2) {
                if(img2) setupBackground(img2);
            });
            return;
        }
        setupBackground(img);
    });
}

function setupBackground(img) {
    const ratio = img.height / img.width;
    CANVAS_HEIGHT = TARGET_WIDTH * ratio;
    canvas.setDimensions({ width: TARGET_WIDTH, height: CANVAS_HEIGHT });
    img.set({ scaleX: TARGET_WIDTH/img.width, scaleY: CANVAS_HEIGHT/img.height, originX: 'left', originY: 'top' });
    canvas.setBackgroundImage(img, canvas.renderAll.bind(canvas));
}

function toggleVolInputs() {
    const method = document.getElementById('vol-method').value;
    const manualInputs = document.querySelectorAll('.manual-input');
    const autoInput = document.querySelector('.auto-input');
    
    if (method === 'manual') {
        manualInputs.forEach(el => el.style.display = (el.tagName === 'DIV' || el.tagName === 'INPUT') ? '' : 'block');
        document.getElementById('dims-row-2').style.display = 'grid';
        autoInput.style.display = 'none';
        document.getElementById('d1').style.display = 'block';
    } else {
        manualInputs.forEach(el => el.style.display = 'none');
        autoInput.style.display = 'block';
    }
}

function addLesionVisual() {
    uniqueIdCounter++;
    const internalId = uniqueIdCounter;
    
    const circle = new fabric.Ellipse({
        rx: 25, ry: 25, fill: '#fbc531', stroke: 'black', strokeWidth: 1,
        originX: 'center', originY: 'center', opacity: 0.85 
    });
    const text = new fabric.Text("?", {
        fontSize: 16, fontFamily: 'Arial', fontWeight: 'bold',
        originX: 'center', originY: 'center', fill: 'black', padding: 5
    });
    const group = new fabric.Group([circle, text], {
        left: TARGET_WIDTH / 2, top: CANVAS_HEIGHT / 2, 
        hasControls: true, hasBorders: true, lockRotation: false,
        cornerSize: 8, transparentCorners: false
    });
    group.internalId = internalId;
    
    group.on('scaling', function() {
        this.item(1).set({ scaleX: 1/this.scaleX, scaleY: 1/this.scaleY });
    });

    canvas.add(group);
    canvas.setActiveObject(group);
    
    lesions.push({ 
        internalId: internalId, fabricObj: group, 
        piradsKey: '3', val: 3, ord: 30, size: '',
        zoneType: 'ZP', 
        zoneText: '' 
    });
    
    addLesionRow(internalId);
    sortAndRenameLesions(); 
}

function addLesionRow(internalId) {
    const container = document.getElementById('lesion-rows-container');
    const row = document.createElement('div');
    row.className = 'lesion-row';
    row.id = `lesion-row-${internalId}`;
    
    let btns = PIRADS_CONF.map(p => {
        let sel = (p.k === '3') ? 'selected' : ''; 
        return `<div class="pi-btn ${p.cls} ${sel}" data-key="${p.k}" onclick="setLesionScore(${internalId}, '${p.k}')">${p.lbl}</div>`;
    }).join('');

    row.innerHTML = `
        <div class="lesion-name">L?</div>
        
        <div class="pirads-selector">${btns}</div>
        
        <div class="zone-selector">
            <div class="zone-btn selected" onclick="setLesionZoneType(${internalId}, 'ZP', this)">ZP</div>
            <div class="zone-btn" onclick="setLesionZoneType(${internalId}, 'ZT', this)">ZT</div>
        </div>

        <div class="lesion-inputs-wrapper">
            <input type="text" class="lesion-input" id="zone-txt-${internalId}" placeholder="zone" oninput="updateLesionData(${internalId})">
            <input type="number" class="lesion-input" id="size-${internalId}" placeholder="mm" oninput="updateLesionData(${internalId})" onblur="sortAndRenameLesions()">
        </div>
        
        <button class="trash-btn" onclick="removeLesion(${internalId})"><i class="fas fa-trash"></i></button>
    `;
    container.appendChild(row);
}

function setLesionScore(id, key) {
    let l = lesions.find(x => x.internalId === id);
    let conf = PIRADS_CONF.find(x => x.k === key);
    if (!l || !conf) return;

    l.piradsKey = key;
    l.val = conf.v;
    l.ord = conf.ord; 

    if(l.fabricObj) {
        l.fabricObj.item(0).set('fill', conf.c);
        canvas.requestRenderAll();
    }
    
    sortAndRenameLesions(); 
}

function setLesionZoneType(id, type, elem) {
    let l = lesions.find(x => x.internalId === id);
    if(!l) return;
    l.zoneType = type;
    
    let parent = elem.parentElement;
    for(let child of parent.children) child.classList.remove('selected');
    elem.classList.add('selected');
    updateReport();
}

function updateLesionData(id) {
    let l = lesions.find(x => x.internalId === id);
    if(l) {
        l.zoneText = document.getElementById(`zone-txt-${id}`).value;
        l.size = document.getElementById(`size-${id}`).value;
        updateReport();
    }
}

function sortAndRenameLesions() {
    lesions.sort((a, b) => {
        if (b.ord !== a.ord) return b.ord - a.ord; 
        return (parseFloat(b.size)||0) - (parseFloat(a.size)||0); 
    });

    const container = document.getElementById('lesion-rows-container');
    lesions.forEach((l, index) => {
        const newLabel = "L" + (index + 1);
        l.label = newLabel;
        
        if (l.fabricObj) { 
            l.fabricObj.item(1).set('text', newLabel); 
            l.fabricObj.addWithUpdate(); 
        }
        
        const row = document.getElementById(`lesion-row-${l.internalId}`);
        if(row) { 
            row.querySelector('.lesion-name').innerText = newLabel; 
            
            // Mise à jour visuelle des boutons
            let conf = PIRADS_CONF.find(c => c.k === l.piradsKey);
            if(conf) row.style.borderLeftColor = conf.c;
            
            const btns = row.querySelectorAll('.pi-btn');
            btns.forEach(b => {
                b.classList.remove('selected');
                // C'est ici que data-key est indispensable
                if(b.getAttribute('data-key') === l.piradsKey) {
                    b.classList.add('selected');
                }
            });

            container.appendChild(row); 
        }
    });
    
    canvas.requestRenderAll();
    updateReport();
}

function removeLesion(internalId) {
    const lesionObj = lesions.find(l => l.internalId === internalId);
    if(lesionObj) canvas.remove(lesionObj.fabricObj);
    const row = document.getElementById(`lesion-row-${internalId}`);
    if(row) row.remove();
    lesions = lesions.filter(l => l.internalId !== internalId);
    sortAndRenameLesions();
}

// --- LOGIQUE TRADUCTION ---
function translateZoneInput(input) {
    if (!input) return "";
    let s = input.toLowerCase().trim();
    let match = s.match(/^(\d+)([ap])$/);
    if(!match) return input; 

    let num = parseInt(match[1]);
    let suffix = match[2]; 

    let height = "";
    if([1, 2, 7, 8, 13].includes(num)) height = "basale";
    else if([3, 4, 9, 10, 14].includes(num)) height = "médio-lobaire";
    else if([5, 6, 11, 12, 15].includes(num)) height = "apicale";
    else return input; 

    let depth = (suffix === 'a') ? "antérieure" : "postérieure";

    let side = "";
    if([1, 2, 3, 4, 5, 6].includes(num)) side = "droite";
    else if([7, 8, 9, 10, 11, 12].includes(num)) side = "gauche";

    if([13, 14, 15].includes(num)) {
        return height; 
    }

    return `${height} ${depth} ${side}`;
}

function formatLesionList(lesionArray) {
    if(lesionArray.length === 0) return "";
    
    let descs = lesionArray.map(l => {
        let loc = translateZoneInput(l.zoneText);
        let sz = (l.size) ? ` de ${l.size} mm` : "";
        let orig = l.zoneText ? `dans z${l.zoneText}` : "";
        return `${loc} PI-RADS ${l.piradsKey}${sz} (« ${l.label} » sur le schéma ${orig})`;
    });

    if(descs.length === 1) {
        return "On y individualise une lésion " + descs[0] + ".";
    } else {
        let last = descs.pop();
        return "On y individualise plusieurs lésions : " + descs.join(", ") + (descs.length > 0 ? ", et " : "") + last + ".";
    }
}

function updateReport() {
    let vol = 0;
    const isManual = document.getElementById('vol-method').value === 'manual';
    if (isManual) {
        const d1 = parseFloat(document.getElementById('d1').value) || 0;
        const d2 = parseFloat(document.getElementById('d2').value) || 0;
        const d3 = parseFloat(document.getElementById('d3').value) || 0;
        vol = (d1 * d2 * d3 * 0.52 / 1000);
    } else {
        vol = parseFloat(document.getElementById('vol-auto').value) || 0;
    }
    
    currentReportData.vol = vol.toFixed(0);
    const psa = parseFloat(document.getElementById('psa').value) || 0;
    
    let txt = "";
    currentReportData.indication = "";
    
    if (psa > 0) {
        let psaStr = `Taux de PSA récemment dosé à ${psa.toString().replace('.', ',')} ng/ml.`;
        txt += psaStr + "\n";
        currentReportData.indication = psaStr;
    }

    txt += "Analyse PI-RADS multiparamétrique (T2, diffusion, et perfusion).\n\n";
    currentReportData.technique = "Analyse PI-RADS multiparamétrique (T2, diffusion, et perfusion).";
    
    let resTxt = `Le volume de la glande est estimé à ${vol.toFixed(0)} cc.\n`;
    
    if(psa > 0 && vol > 0) {
        let densStr = (psa / vol).toFixed(2).replace('.', ',');
        resTxt += `Densité de PSA évaluée à ${densStr} ng/mL/cc.\n`;
    }

    resTxt += "Pas d'épaississement significatif du détrusor.\n"; 
    resTxt += "Pas de dilatation des cavités pyélocalicielles.\n\n";

    let pzLesions = [];
    let tzLesions = [];
    let asLesions = [];

    lesions.forEach(l => {
        let zRaw = (l.zoneText || "").trim().toLowerCase();
        if(['13a', '14a', '15a'].includes(zRaw)) {
            asLesions.push(l);
        } else {
            if (l.zoneType === 'ZP') pzLesions.push(l);
            else tzLesions.push(l);
        }
    });

    resTxt += "La zone périphérique présente quelques remaniements ne gênant pas l'interprétation du signal. ";
    if (pzLesions.length > 0) resTxt += formatLesionList(pzLesions);
    else resTxt += "Pas de lésion significative décelable.";
    resTxt += "\n";

    resTxt += "La zone de transition présente quelques remaniements ne gênant pas l'interprétation du signal. ";
    if (tzLesions.length > 0) resTxt += formatLesionList(tzLesions);
    else resTxt += "Pas de lésion significative décelable.";
    resTxt += "\n";

    if (asLesions.length > 0) {
        let sfmaDescs = asLesions.map(l => {
             let loc = translateZoneInput(l.zoneText);
             let sz = (l.size) ? ` de ${l.size} mm` : "";
             let orig = l.zoneText ? `dans z${l.zoneText}` : "";
             return `une lésion ${loc} PI-RADS ${l.piradsKey}${sz} (« ${l.label} » sur le schéma ${orig})`;
        });
        resTxt += "Le stroma fibromusculaire antérieur présente " + sfmaDescs.join(" et ") + ".";
    } else {
        resTxt += "Le stroma fibromusculaire antérieur est fin et très hypointense en T2.";
    }
    resTxt += "\n\n";

    resTxt += "Vésicules séminales symétriques d'aspect normal.\nPas d'adénopathie pelvienne significative.\nPas de lésion osseuse suspecte.";
    
    currentReportData.resultat = resTxt;
    txt += resTxt;

    // --- CONCLUSION ---
    let conclusionTxt = "";
    let sigLesions = lesions.filter(l => l.val >= 3);
    
    if(sigLesions.length > 0) {
        let groups = {};
        sigLesions.forEach(l => {
            let k = l.piradsKey;
            if(!groups[k]) groups[k] = [];
            groups[k].push(l);
        });

        let sortedKeys = Object.keys(groups).sort((a,b) => {
            let ordA = PIRADS_CONF.find(p => p.k === a).ord;
            let ordB = PIRADS_CONF.find(p => p.k === b).ord;
            return ordB - ordA; 
        });

        let conclusionSentences = [];

        sortedKeys.forEach(key => {
            let group = groups[key];
            let scoreVal = group[0].val; 
            
            let riskText = "équivoque";
            if (scoreVal === 4) riskText = "suspecte";
            if (scoreVal === 5) riskText = "très suspecte";
            if (group.length > 1) {
                if(scoreVal === 3) riskText = "équivoques"; else riskText += "s";
            }
            
            let cPZ = [], cTZ = [], cAS = [];
            group.forEach(l => {
                let zRaw = (l.zoneText || "").trim().toLowerCase();
                if(['13a', '14a', '15a'].includes(zRaw)) cAS.push(l);
                else if(l.zoneType === 'ZP') cPZ.push(l);
                else cTZ.push(l);
            });

            let segments = [];
            const fmt = (arr) => arr.map(l => {
                let z = translateZoneInput(l.zoneText);
                let sz = (l.size) ? ` de ${l.size} mm` : "";
                return `${z}${sz} (« ${l.label} »)`;
            }).join(" et ");

            if(cPZ.length > 0) segments.push(`dans la zone périphérique ${fmt(cPZ)}`);
            if(cTZ.length > 0) segments.push(`dans la zone de transition ${fmt(cTZ)}`);
            if(cAS.length > 0) segments.push(`dans le stroma fibromusculaire antérieur ${fmt(cAS)}`);

            let sentence = `Lésion${group.length > 1 ? 's' : ''} ${riskText} (PI-RADS ${key}) ${segments.join(" et ")}.`;
            conclusionSentences.push(sentence);
        });
        conclusionTxt = conclusionSentences.join("\n");
        txt += conclusionTxt;
    }
    
    currentReportData.conclusion = conclusionTxt;
    const area = document.getElementById('report-text');
    area.value = txt;
}

// Fonction Fallback (pour quand HTTPS n'est pas dispo)
function copyHtmlLegacy(htmlContent) {
    const tempDiv = document.createElement("div");
    tempDiv.innerHTML = htmlContent;
    tempDiv.style.position = "fixed";
    tempDiv.style.left = "-9999px";
    tempDiv.style.top = "0";
    document.body.appendChild(tempDiv);
    
    const range = document.createRange();
    range.selectNode(tempDiv);
    const selection = window.getSelection();
    selection.removeAllRanges();
    selection.addRange(range);
    
    try {
        const successful = document.execCommand('copy');
        if(successful) {
            showCopyFeedback(true);
        } else {
            alert("Copie impossible via méthode secours.");
        }
    } catch (err) {
        alert("Erreur copie secours : " + err);
    }
    
    document.body.removeChild(tempDiv);
}

function showCopyFeedback(success) {
    const btn = document.getElementById('btn-copy-full');
    const originalHtml = btn.innerHTML;
    if(success) {
        btn.innerHTML = '<i class="fas fa-check"></i> Copié !';
        btn.style.borderColor = "green";
        btn.style.color = "green";
    }
    setTimeout(() => {
        btn.innerHTML = originalHtml;
        btn.style.borderColor = "";
        btn.style.color = "";
    }, 2000);
}

async function copyFullReport() {
    canvas.discardActiveObject().renderAll();
    const imgData = canvas.toDataURL({ format: 'png', multiplier: 1.5, quality: 1 });

    const formatHTML = (text) => text ? text.replace(/\n/g, '<br>') : "";

    let htmlContent = `
        <div style="font-family: Calibri, sans-serif; font-size: 11pt; color: #000;">
            <p><strong style="text-decoration: underline;">INDICATION</strong><br>${formatHTML(currentReportData.indication)}</p>
            <p><strong style="text-decoration: underline;">TECHNIQUE</strong><br>${formatHTML(currentReportData.technique)}</p>
            <p><strong style="text-decoration: underline;">RESULTAT</strong><br>${formatHTML(currentReportData.resultat)}</p>
            <p><strong style="text-decoration: underline;">CONCLUSION</strong><br><strong>${formatHTML(currentReportData.conclusion)}</strong></p>

            <p style="text-align: center; margin: 10px 0;">
                <img src="${imgData}" style="max-width: 200px; height: auto;" alt="Schéma Prostatique">
            </p>
        </div>
    `;

    try {
        const blobHtml = new Blob([htmlContent], { type: 'text/html' });
        const plainText = document.getElementById('report-text').value;
        const blobText = new Blob([plainText], { type: 'text/plain' });

        const data = [new ClipboardItem({
            'text/html': blobHtml,
            'text/plain': blobText
        })];

        await navigator.clipboard.write(data);
        showCopyFeedback(true);

    } catch (err) {
        console.warn("Clipboard API failed (HTTPS?), trying legacy execCommand...", err);
        copyHtmlLegacy(htmlContent);
    }
}

function copySchema() {
    canvas.discardActiveObject().renderAll();
    const dataURL = canvas.toDataURL({ format: 'png', quality: 1, multiplier: 1 });
    try {
        const byteString = atob(dataURL.split(',')[1]);
        const mimeString = dataURL.split(',')[0].split(':')[1].split(';')[0];
        const ab = new ArrayBuffer(byteString.length);
        const ia = new Uint8Array(ab);
        for (let i = 0; i < byteString.length; i++) {
            ia[i] = byteString.charCodeAt(i);
        }
        const blob = new Blob([ab], {type: mimeString});
        const item = new ClipboardItem({ [mimeString]: blob });
        navigator.clipboard.write([item]).then(() => {
            const btn = document.querySelector('button[onclick="copySchema()"]');
            if(btn) {
                const originalHtml = btn.innerHTML;
                btn.innerHTML = '<i class="fas fa-check"></i>';
                setTimeout(() => btn.innerHTML = originalHtml, 1500);
            }
        });
    } catch (e) {
        console.error(e);
        // Fallback image seule simple (juste alert car execCommand image est complexe)
        alert("Erreur copie image (HTTPS requis). Essayez 'Copier CR + Schéma' qui a une méthode de secours.");
    }
}

function fullReset() {
    document.querySelectorAll('input').forEach(i => i.value = '');
    document.getElementById('vol-method').value = 'auto'; 
    [...lesions].forEach(l => removeLesion(l.internalId));
    uniqueIdCounter = 0;
    toggleVolInputs(); 
    updateReport();
}
</script>