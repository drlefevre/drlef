# [IRM Prostate](https://www.pcih.fr/portal/pst_selectapp.php){:target="_blank"}

<script src="https://cdnjs.cloudflare.com/ajax/libs/fabric.js/5.3.1/fabric.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">

<div class="box md-typeset" id="prostate-box">
  
  <div class="pairs">
    <div style="grid-column: 1 / -1; margin-bottom: 0.4rem; display:flex; gap:0.6rem;">
        <select id="perfusion" onchange="updateReport()">
            <option value="multiparametrique" selected>Multiparamétrique</option>
            <option value="biparametrique">Biparamétrique</option>
        </select>
        <select id="vol-method" onchange="toggleVolInputs(); updateReport()">
            <option value="auto" selected>Contourage automatique</option>
            <option value="manual">3 dimensions</option>
        </select>
    </div>

    <div class="pair">
        <input id="psa" type="number" inputmode="decimal" placeholder="PSA" oninput="updateReport()" />

        <div id="vol-right" style="display:contents;">
            <input id="vol-auto" type="number" class="auto-input" inputmode="decimal" placeholder="cc" oninput="updateReport()" />
            <div id="d1-wrapper" style="display:none; width:100%;">
                <input id="d1" type="number" class="manual-input" inputmode="decimal" placeholder="mm" oninput="updateReport()" style="flex:1; width:100%;" />
            </div>
        </div>
    </div>

    <div class="pair manual-input" id="dims-row-2" style="display:none;">
        <input id="d2" type="number" inputmode="decimal" placeholder="mm" oninput="updateReport()" />
        <input id="d3" type="number" inputmode="decimal" placeholder="mm" oninput="updateReport()" />
    </div>

    <div class="pair">
        <div style="display:flex; align-items:center; gap:0.3rem; flex:1;">
            <span style=" width:1.2rem;">ZP</span>
            <select id="zone-zp" onchange="updateReport()" style="flex:1;">
                <option value="pas">Pas de remaniements</option>
                <option value="quelques" selected>Quelques remaniements</option>
                <option value="multiples">Multiples remaniements</option>
                <option value="hypointense">Hypointensité T2 diffuse</option>
            </select>
        </div>
        <div style="display:flex; align-items:center; gap:0.3rem; flex:1;">
            <span style=" width:1.2rem;">ZT</span>
            <select id="zone-zt" onchange="updateReport()" style="flex:1;">
                <option value="pas">Pas de remaniements</option>
                <option value="quelques" selected>Quelques remaniements</option>
                <option value="multiples">Multiples remaniements</option>
                <option value="hypointense">Hypointensité T2 diffuse</option>
            </select>
        </div>
    </div>
  </div>

  <div class="canvas-container-wrapper">
      <div class="canvas-container">
          <canvas id="prostateCanvas"></canvas>
          <div class="canvas-controls">
              <button type="button" class="action-btn" onclick="addLesionVisual()">
                <i class="fas fa-plus"></i> Lésion
              </button>
              <button type="button" class="action-btn" id="btn-copy-schema" onclick="copySchema()" title="Copier le schéma">
                <i class="fas fa-copy"></i>
              </button>
              <button type="button" class="action-btn" id="btn-download-schema" onclick="downloadSchema()" title="Télécharger le schéma">
                <i class="fas fa-download"></i>
              </button>
              <button type="button" class="action-btn" onclick="copyReportOnly()" title="Copier le compte-rendu sans schéma" id="btn-copy-cr">
                CR
              </button>
          </div>
      </div>
  </div>

  <div id="lesion-rows-container"></div>

  <div class="final-actions">
      <button type="button" class="btn-main-copy" id="btn-copy-full" onclick="copyFullReport()">
         CR ± schéma
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
.pair { display:grid; grid-template-columns: repeat(2, 1fr); gap:.6rem; align-items: flex-start; }

.box input, .box select {
  width: 100%; padding: .4rem .6rem;
  border: 1px solid var(--md-default-fg-color--lighter); 
  border-radius: .4rem; background: var(--md-code-bg-color);
  font-size: .8rem; color: var(--md-default-fg-color);
  margin: 0; text-align: center;
}
.box input:focus, .box select:focus { border-color: var(--md-default-fg-color--light); }

input[type=number]::-webkit-inner-spin-button, 
input[type=number]::-webkit-outer-spin-button { 
  -webkit-appearance: none; margin: 0; 
}
input[type=number] { -moz-appearance: textfield; }

/* Canvas */
.canvas-container-wrapper { display: flex; justify-content: center; margin-top: 1rem; width: 100%; padding: 0; }
.canvas-container {
    position: relative; background: white;
    border: 1px solid var(--md-default-fg-color--lighter);
    border-radius: .5rem; overflow: hidden;
    box-shadow: 0 2px 4px -1px rgba(0,0,0,0.1);
    width: 100%;
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

.add-circle-btn {
    width: 24px; height: 24px; display:flex; align-items:center; justify-content:center;
    border-radius:4px; background: white; border: 1px solid var(--md-default-fg-color--lighter);
    cursor: pointer; font-weight: bold; flex-shrink: 0; margin-right:6px;
}
.add-circle-btn:hover { background: #f7f7f7; }

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
.c-green { background: #c4e538; }
.c-yellow   { background: #FFD966; }
.c-orange { background: #FFA500; }
.c-red    { background: #FF0000; }

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
    { k: '2',   lbl: '2',  v: 2, ord: 20, c: '#c4e538', cls: 'c-green' },
    { k: '2+1', lbl: '+1', v: 3, ord: 29, c: '#FFD966', cls: 'c-yellow' }, 
    { k: '3',   lbl: '3',  v: 3, ord: 30, c: '#FFD966', cls: 'c-yellow' },
    { k: '3+1', lbl: '+1', v: 4, ord: 39, c: '#FFA500', cls: 'c-orange' }, 
    { k: '4',   lbl: '4',  v: 4, ord: 40, c: '#FFA500', cls: 'c-orange' },
    { k: '5',   lbl: '5',  v: 5, ord: 50, c: '#FF0000', cls: 'c-red' }
];

let TARGET_WIDTH = 0;
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
    const container = document.querySelector('.canvas-container');
    TARGET_WIDTH = container.offsetWidth || 800;
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
    const volAuto = document.getElementById('vol-auto');
    const d1wrap = document.getElementById('d1-wrapper');
    const dimsRow2 = document.getElementById('dims-row-2');

    if (method === 'manual') {
        volAuto.style.display = 'none';
        d1wrap.style.display = 'flex';
        dimsRow2.style.display = 'grid';
    } else {
        volAuto.style.display = 'block';
        d1wrap.style.display = 'none';
        dimsRow2.style.display = 'none';
    }
}

function addLesionVisual() {
    uniqueIdCounter++;
    const internalId = uniqueIdCounter;
    
    const circle = new fabric.Ellipse({
        rx: 25, ry: 25, fill: '#FFD966', stroke: 'black', strokeWidth: 1,
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
        internalId: internalId, fabricObjs: [group], 
        piradsKey: '3', val: 3, ord: 30, size: '',
        zoneType: 'ZP', 
        zoneText: '',
        likertScore: null
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
        return `<div class="pi-btn ${p.cls} ${sel}" data-key="${p.k}" onmousedown="setLesionScore(${internalId}, '${p.k}')">${p.lbl}</div>`;
    }).join('');

    row.innerHTML = `
        <div class="lesion-name">L?</div>
        
        <button class="add-circle-btn" onmousedown="addDuplicateCircle(${internalId})" title="Ajouter un cercle">+</button>

        <div class="pirads-selector">${btns}</div>
        
        <div class="zone-selector">
            <div class="zone-btn selected" onmousedown="setLesionZoneType(${internalId}, 'ZP', this)">ZP</div>
            <div class="zone-btn" onmousedown="setLesionZoneType(${internalId}, 'ZT', this)">ZT</div>
        </div>

        <div class="lesion-inputs-wrapper" style="grid-template-columns: 1fr 1fr 1fr;">
            <input type="text" class="lesion-input" id="zone-txt-${internalId}" placeholder="zone" oninput="updateLesionDataTemp(${internalId})" onblur="updateLesionData(${internalId})">
            <input type="number" class="lesion-input" id="size-${internalId}" placeholder="mm" oninput="updateLesionDataTemp(${internalId})" onblur="updateLesionData(${internalId})">
            <input type="number" class="lesion-input" id="likert-${internalId}" placeholder="Likert" min="1" max="5" oninput="updateLesionDataTemp(${internalId})" onblur="updateLesionData(${internalId})" />
        </div>
        
        <button class="trash-btn" onmousedown="removeLesion(${internalId})"><i class="fas fa-trash"></i></button>
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

    if(l.fabricObjs || l.fabricObj) {
        // Support multiple fabric objects per lesion
        if (Array.isArray(l.fabricObjs)) {
            l.fabricObjs.forEach(obj => {
                try { obj.item(0).set('fill', conf.c); } catch(e){}
            });
        } else if (l.fabricObj) {
            l.fabricObj.item(0).set('fill', conf.c);
        }
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

function updateLesionDataTemp(id) {
    let l = lesions.find(x => x.internalId === id);
    if(l) {
        l.zoneText = document.getElementById(`zone-txt-${id}`).value;
        l.size = document.getElementById(`size-${id}`).value;
        const likertInput = document.getElementById(`likert-${id}`).value;
        l.likertScore = likertInput ? parseInt(likertInput) : null;
    }
}

function updateLesionData(id) {
    updateLesionDataTemp(id);
    sortAndRenameLesions();
}

function sortAndRenameLesions() {
    lesions.sort((a, b) => {
        if (b.ord !== a.ord) return b.ord - a.ord;
        // Si ordre PI-RADS identique, trier par Likert (décroissant : risque plus élevé en premier)
        // Traiter null comme 0 (non renseigné)
        const aLikert = a.likertScore !== null ? a.likertScore : 0;
        const bLikert = b.likertScore !== null ? b.likertScore : 0;
        if (bLikert !== aLikert) return bLikert - aLikert;
        // Puis par taille (décroissante)
        return (parseFloat(b.size)||0) - (parseFloat(a.size)||0); 
    });

    const container = document.getElementById('lesion-rows-container');
    lesions.forEach((l, index) => {
        const newLabel = "L" + (index + 1);
        l.label = newLabel;
        
        if (l.fabricObjs || l.fabricObj) { 
            // update all fabric objects when multiple circles exist
            if (Array.isArray(l.fabricObjs)) {
                l.fabricObjs.forEach(obj => {
                    try { obj.item(1).set('text', newLabel); obj.addWithUpdate(); } catch(e){}
                });
            } else {
                l.fabricObj.item(1).set('text', newLabel);
                l.fabricObj.addWithUpdate(); 
            }
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
                if(b.getAttribute('data-key') === l.piradsKey) {
                    b.classList.add('selected');
                }
            });

            if (container.children[index] !== row) {
                container.appendChild(row);
            }
        }
    });
    
    canvas.requestRenderAll();
    updateReport();
}

function removeLesion(internalId) {
    const lesionObj = lesions.find(l => l.internalId === internalId);
    if(lesionObj) {
        if (Array.isArray(lesionObj.fabricObjs)) {
            lesionObj.fabricObjs.forEach(obj => { try { canvas.remove(obj); } catch(e){} });
        } else if (lesionObj.fabricObj) {
            canvas.remove(lesionObj.fabricObj);
        }
    }
    const row = document.getElementById(`lesion-row-${internalId}`);
    if(row) row.remove();
    lesions = lesions.filter(l => l.internalId !== internalId);
    sortAndRenameLesions();
}

function addDuplicateCircle(internalId) {
    const l = lesions.find(x => x.internalId === internalId);
    if(!l) return;
    const base = (Array.isArray(l.fabricObjs) && l.fabricObjs[0]) || l.fabricObj;
    if(!base) return;

    // Try to copy visual properties from base
    let rx = 25, ry = 25;
    try { rx = base.item(0).get('rx') || rx; ry = base.item(0).get('ry') || ry; } catch(e){}
    let fill = '#FFD966';
    try { fill = base.item(0).get('fill') || fill; } catch(e){}
    let labelText = l.label || '?';
    try { labelText = base.item(1).get('text') || labelText; } catch(e){}

    const left = (base.left || TARGET_WIDTH/2) + 30;
    const top = (base.top || CANVAS_HEIGHT/2) + 30;

    const circle = new fabric.Ellipse({ rx: rx, ry: ry, fill: fill, stroke: 'black', strokeWidth: 1, originX: 'center', originY: 'center', opacity: 0.85 });
    const text = new fabric.Text(labelText, { fontSize: 16, fontFamily: 'Arial', fontWeight: 'bold', originX: 'center', originY: 'center', fill: 'black', padding: 5 });
    const group = new fabric.Group([circle, text], { left: left, top: top, hasControls: true, hasBorders: true, lockRotation: false, cornerSize: 8, transparentCorners: false });
    group.internalId = internalId;

    group.on('scaling', function() {
        this.item(1).set({ scaleX: 1/this.scaleX, scaleY: 1/this.scaleY });
    });

    canvas.add(group);
    canvas.setActiveObject(group);

    // Ensure fabricObjs is an array
    if (!Array.isArray(l.fabricObjs)) {
        l.fabricObjs = l.fabricObjs ? [l.fabricObjs] : [];
    }
    l.fabricObjs.push(group);
    canvas.requestRenderAll();
}

// --- LOGIQUE TRADUCTION LIKERT ---
function getLikertLabel(likertScore) {
    const likertMap = {
        1: "très peu suspecte",
        2: "peu suspecte",
        3: "équivoque",
        4: "suspecte",
        5: "très suspecte"
    };
    return likertMap[likertScore] || "équivoque";
}

// --- LOGIQUE REMANIEMENTS ZONES ---
function getZoneText(zoneValue, zoneName) {
    const templates = {
        pas: `La zone ${zoneName} ne présente pas de remaniement notable.`,
        quelques: `La zone ${zoneName} présente quelques remaniements ne gênant pas l'interprétation du signal.`,
        multiples: `La zone ${zoneName} présente de multiples remaniements gênant l'interprétation du signal.`,
        hypointense: `La zone ${zoneName} présente des remaniements hypointenses T2 diffus gênant l'interprétation du signal.`
    };
    return templates[zoneValue] || templates.quelques;
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
        
        // Afficher la mention Likert seulement si renseigné ET différent du PI-RADS
        let riskDesc = "";
        
        if (l.likertScore !== null && l.likertScore !== l.val) {
            // Likert renseigné et diffère du PI-RADS : inclure la mention override
            let likertLabel = getLikertLabel(l.likertScore);
            riskDesc = `${likertLabel} (classée Likert ${l.likertScore} malgré une sémiologie PI-RADS ${l.piradsKey})`;
        } else {
            // Likert non renseigné ou = PI-RADS : utiliser simplement le libellé
            riskDesc = `PI-RADS ${l.piradsKey}`;
        }
        
        return `${loc} ${riskDesc}${sz} (« ${l.label} » sur le schéma ${orig})`;
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
    
    let psaStr = psa > 0 ? `Taux de PSA récemment dosé à ${psa.toString().replace('.', ',')} ng/ml.` : `Taux de PSA non disponible.`;
    currentReportData.indication = psaStr;

    const perf = (document.getElementById('perfusion') && document.getElementById('perfusion').value) || 'multiparametrique';
    const techniqueTxt = perf === 'biparametrique'
        ? 'Analyse PI-RADS biparamétrique (T2 et diffusion).'
        : 'Analyse PI-RADS multiparamétrique (T2, diffusion, et perfusion).';

    currentReportData.technique = techniqueTxt;
    let txt = psaStr + "\n" + techniqueTxt + "\n\n";
    
    let resTxt = `Le volume de la glande est estimé à ${vol.toFixed(0)} cc`;
    
    if(psa > 0 && vol > 0) {
        let densStr = (psa / vol).toFixed(2).replace('.', ',');
        resTxt += ` (densité de PSA = ${densStr} ng/mL/cc)`;
    }
    resTxt += ".\n";

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

    const zpValue = document.getElementById('zone-zp').value;
    const ztValue = document.getElementById('zone-zt').value;

    resTxt += getZoneText(zpValue, "périphérique") + " ";
    if (pzLesions.length > 0) resTxt += formatLesionList(pzLesions);
    else resTxt += "Pas de lésion significative décelable.";
    resTxt += "\n";

    resTxt += getZoneText(ztValue, "de transition") + " ";
    if (tzLesions.length > 0) resTxt += formatLesionList(tzLesions);
    else resTxt += "Pas de lésion significative décelable.";
    resTxt += "\n";

    if (asLesions.length > 0) {
        resTxt += "Le stroma fibromusculaire antérieur présente ";
        // Utiliser formatLesionList() pour appliquer la logique Likert
        let sfmaFormatted = formatLesionList(asLesions);
        if (sfmaFormatted.startsWith('On y individualise')) {
            resTxt += sfmaFormatted.substring('On y individualise '.length);
        } else {
            resTxt += sfmaFormatted;
        }
    } else {
        resTxt += "Le stroma fibromusculaire antérieur est fin et très hypointense en T2.";
    }

    resTxt += "\n\nPar ailleurs :\nVésicules séminales symétriques d'aspect normal.\nPas d'épaississement significatif du détrusor.\nPas de dilatation des cavités pyélocalicielles.\nPas d'adénopathie pelvienne significative.";
    
    currentReportData.resultat = resTxt;
    txt += resTxt + "\n\n";

    // --- CONCLUSION ---
    let conclusionTxt = "";

    // Si aucune lésion renseignée => conclusion simple
    if (lesions.length === 0) {
        conclusionTxt = `Volume prostatique estimé à ${vol.toFixed(0)} cc. Pas de lésion suspecte.`;
        txt += conclusionTxt;
    } else {
        // Produire une ligne par lésion, triée par label (L1, L2...)
        const sortedByLabel = lesions.slice().sort((a, b) => {
            const ai = parseInt((a.label || '').replace(/^L/i, '')) || 0;
            const bi = parseInt((b.label || '').replace(/^L/i, '')) || 0;
            return ai - bi;
        });

        const lines = sortedByLabel.map(l => {
            const label = l.label || '';
            const loc = translateZoneInput(l.zoneText) || 'non précisée';
            const sz = (l.size) ? ` de ${l.size} mm` : '';
            
            // Déterminer la zone (périphérique, transition, SFMA)
            const zRaw = (l.zoneText || '').trim().toLowerCase();
            const isSFMA = ['13a', '14a', '15a'].includes(zRaw);
            const zonePrefix = isSFMA ? 'dans le stroma fibromusculaire antérieur ' :
                              (l.zoneType === 'ZP') ? 'dans la zone périphérique ' : 
                              (l.zoneType === 'ZT') ? 'dans la zone de transition ' : 
                              'dans la zone périphérique ';

            let riskText = 'équivoque';
            if (l.val === 4) riskText = 'suspecte';
            if (l.val === 5) riskText = 'très suspecte';

            let riskDesc = '';
            if (l.likertScore !== null && l.likertScore !== l.val) {
                // Likert renseigné et différent : format override
                riskDesc = `${getLikertLabel(l.likertScore)} (classée Likert ${l.likertScore} malgré une sémiologie PI-RADS ${l.piradsKey})`;
            } else {
                // Pas de Likert ou Likert = PI-RADS : utiliser le texte de risque
                riskDesc = `${riskText} (PI-RADS ${l.piradsKey})`;
            }

            return `Lésion ${riskDesc} ${zonePrefix}${loc}${sz} (« ${label} »).`;
        });

        conclusionTxt = lines.join('\n');
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

    const formatHTML = (text) => text ? text.replace(/\n/g, '<br>') : "";

    const includeImage = lesions.length > 0;
    let imgTag = "";
    let imgData = null;
    if (includeImage) {
        imgData = canvas.toDataURL({ format: 'png', multiplier: 3, quality: 1 });
        imgTag = `
            <p style="text-align: center; margin: 10px 0;">
                <img src="${imgData}" style="max-width: 520px; height: auto;" alt="Schéma Prostatique">
            </p>`;
    }

    // Diviser le résultat pour insérer le schéma avant "Par ailleurs :"
    let resultatHtml = formatHTML(currentReportData.resultat);
    let resultatFinal = resultatHtml;
    
    if (includeImage && resultatHtml.includes("Par ailleurs :")) {
        const parts = resultatHtml.split("Par ailleurs :");
        resultatFinal = parts[0] + imgTag + "Par ailleurs :" + parts[1];
    } else if (includeImage) {
        resultatFinal = resultatHtml + imgTag;
    }

    let htmlContent = `
        <div style="font-family: Calibri, sans-serif; font-size: 11pt; color: #000;">
            <p><strong style="text-decoration: underline;">INDICATION</strong><br>${formatHTML(currentReportData.indication)}</p>
            <p><strong style="text-decoration: underline;">TECHNIQUE</strong><br>${formatHTML(currentReportData.technique)}</p>
            <p><strong style="text-decoration: underline;">RÉSULTAT</strong><br>${resultatFinal}</p>
            <p><strong style="text-decoration: underline;">CONCLUSION</strong><br><strong>${formatHTML(currentReportData.conclusion)}</strong></p>
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

async function copySchema() {
    // 1. Désélectionner les objets pour ne pas voir les cadres de sélection
    canvas.discardActiveObject().renderAll();

    // 2. Générer l'image
    const imgData = canvas.toDataURL({ 
        format: 'png', 
        multiplier: 3, 
        quality: 1 
    });

    // 3. Créer le contenu HTML avec la contrainte de largeur
    const htmlContent = `
        <img src="${imgData}" style="max-width: 500px; height: auto; display: block;" alt="Schéma Thyroïde">
    `;

    try {
        // 4. Préparer les formats pour le presse-papier
        const blobHtml = new Blob([htmlContent], { type: 'text/html' });
        // Fallback texte simple si le HTML n'est pas supporté
        const blobText = new Blob(["[Schéma Prostate]"], { type: 'text/plain' });

        const data = [new ClipboardItem({ 
            'text/html': blobHtml, 
            'text/plain': blobText 
        })];

        // 5. Écrire dans le presse-papier
        await navigator.clipboard.write(data);
        
        // Feedback visuel sur le bouton "Copier schéma"
        try {
            const btn = document.getElementById('btn-copy-schema');
            if (btn) {
                const original = btn.innerHTML;
                btn.innerHTML = '<i class="fas fa-check"></i> Copié';
                btn.style.borderColor = 'green'; btn.style.color = 'green';
                setTimeout(() => { btn.innerHTML = original; btn.style.borderColor = ''; btn.style.color = ''; }, 1800);
            }
        } catch (e) { /* silent */ }
        
    } catch (err) {
        console.error(err);
        alert("Erreur de copie (HTTPS requis).");
    }
}

// Télécharger le schéma en PNG
function downloadSchema() {
    canvas.discardActiveObject().renderAll();
    const imgData = canvas.toDataURL({ format: 'png', multiplier: 3, quality: 1 });
    const a = document.createElement('a');
    a.href = imgData;
    a.download = 'schema-prostate.png';
    document.body.appendChild(a);
    a.click();
    a.remove();
    // Feedback visuel sur le bouton "Télécharger"
    try {
        const btn = document.getElementById('btn-download-schema');
        if (btn) {
            const original = btn.innerHTML;
            btn.innerHTML = '<i class="fas fa-check"></i> Téléchargé';
            btn.style.borderColor = 'green'; btn.style.color = 'green';
            setTimeout(() => { btn.innerHTML = original; btn.style.borderColor = ''; btn.style.color = ''; }, 1800);
        }
    } catch (e) { /* silent */ }
}

// Copier uniquement le compte-rendu texte (sans schéma)
async function copyReportOnly() {
    const text = (currentReportData && currentReportData.text) ? currentReportData.text : document.getElementById('report-text').value || '';
    try {
        await navigator.clipboard.writeText(text);
        const btn = document.getElementById('btn-copy-cr');
        const original = btn.innerHTML;
        btn.innerHTML = '<i class="fas fa-check"></i> Copié';
        btn.style.borderColor = 'green'; btn.style.color = 'green';
        setTimeout(() => { btn.innerHTML = original; btn.style.borderColor = ''; btn.style.color = ''; }, 1800);
    } catch (err) {
        console.error(err);
        alert('Erreur de copie (HTTPS requis).');
    }
}

function fullReset() {
    document.querySelectorAll('input').forEach(i => i.value = '');
    [...lesions].forEach(l => removeLesion(l.internalId));
    uniqueIdCounter = 0;
    toggleVolInputs();
    // Remettre les menus déroulants ZP / ZT sur la valeur par défaut "Quelques remaniements"
    if (document.getElementById('zone-zp')) document.getElementById('zone-zp').value = 'quelques';
    if (document.getElementById('zone-zt')) document.getElementById('zone-zt').value = 'quelques';
    updateReport();
}
</script>

| ZP = DWI | [PI-RADS](https://radiologyassistant.nl/abdomen/prostate/prostate-cancer-pi-rads-v2-1){:target="_blank"}  |  ZT = T2 |
| :----------: | :-------: | :----------: |
| linéaire/angulaire | <span style="background-color: #c4e538; color: black; padding: 2px 6px; border-radius:4px; font-weight: bold;">2</span> | ∅capsule/incomplète `+1 DWI marquée` | 
| focale modérée `+1 DCE` | <span style="background-color: #FFD966; color: black; padding: 2px 6px; border-radius:4px; font-weight: bold;">3</span> | hétérogène mal limité `+1 DWI ≥ 15 mm` |
| marquée | <span style="background-color: #FFA500; color: black; padding: 2px 6px; border-radius:4px; font-weight: bold;">4</span> | signal intermédiaire homogène |
| ≥ 15 mm ou EEP | <span style="background-color: #FF0000; color: black; padding: 2px 6px; border-radius:4px; font-weight: bold;">5</span> | ≥ 15 mm ou EEP |

<figure markdown="span">
    [![](assets/algoprostate.jpg){width=600"}](https://www.urofrance.org/recommandation/recommandations-francaises-du-comite-de-cancerologie-de-lafu-actualisation-2024-2026-cancer-de-la-prostate-diagnostic-et-prise-en-charge/#){:target="_blank"}
    <br>
    [**Récidive** après prostatectomie/radiothérapie/HIFU](assets/PI-RR.pdf){:target="_blank"} 
</figure>