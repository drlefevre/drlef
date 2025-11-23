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
                <i class="fas fa-copy"></i> Copier
              </button>
          </div>
      </div>
  </div>

  <div id="lesion-rows-container"></div>

  <div class="results" style="margin-top: 1rem;">
    <div class="result wide">
      <textarea id="report-text" readonly></textarea>
      
      <div class="copy-row">
        <button type="button" class="copy" id="btn-copy">Copier</button>
        <span class="copied" id="msg-copied" aria-live="polite"></span>
      </div>
    </div>
  </div>

  <div class="actions">
    <button type="button" class="clear" onclick="fullReset()">Effacer</button>
  </div>
</div>

<style>
/* Style Global Box */
.box {
  max-width: 820px;
  margin: 1rem 0 2rem;
  padding: 1rem 1rem .5rem;
  border: 1px solid var(--md-default-fg-color--lightest);
  border-radius: .75rem;
  background: var(--md-default-bg-color);
}
.pairs { display:grid; grid-template-columns: 1fr; gap:.45rem; }
.pair { display:grid; grid-template-columns: repeat(2, 1fr); gap:.6rem; }
.result.wide { grid-column: 1 / -1; }

/* Inputs */
.box input, .box select {
  width: 100%; padding: .4rem .6rem;
  border: 1px solid var(--md-default-fg-color--lighter); 
  border-radius: .4rem; background: var(--md-code-bg-color);
  font-size: .8rem; color: var(--md-default-fg-color);
  margin: 0;


}
.box input:focus, .box select:focus { border-color: var(--md-default-fg-color--light); }
input[type=number] { -moz-appearance: textfield; }
input[type=number]::-webkit-inner-spin-button, input[type=number]::-webkit-outer-spin-button { -webkit-appearance: none; margin: 0; }

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
    background: var(--md-code-bg-color); padding: 0 8px;
    border-radius: 4px; margin-bottom: 6px;
    border-left: 5px solid #ccc;
    display: flex; align-items: center; gap: 8px;
    transition: all 0.2s; height: 38px;
}
.lesion-name { font-weight: bold; font-size: 0.9rem; min-width: 25px; }

/* --- STYLE BOUTONS PI-RADS SIMPLIFIÉ --- */
.pirads-selector { display: flex; gap: 4px; align-items: center; }

.pi-btn {
    width: 28px; height: 28px;
    display: flex; justify-content: center; align-items: center;
    border-radius: 4px; cursor: pointer;
    font-weight: bold; font-family: sans-serif; font-size: 13px;
    opacity: 0.4; transition: 0.2s;
    user-select: none; -webkit-user-select: none; color: white;
    box-sizing: border-box; padding: 0; margin: 0;
}
.pi-btn:hover { opacity: 0.8; }
.pi-btn.selected { opacity: 1; transform: scale(1.1); box-shadow: 0 1px 3px rgba(0,0,0,0.3); }

/* Classe pour réduire la police des boutons "+1" */
.pi-sm { font-size: 10px; letter-spacing: -1px; }

/* Couleurs */
.c-green  { background: #4cd137; }
.c-yellow { background: #c4e538; color: #333; }
.c-gold   { background: #fbc531; color: #333; }
.c-orange { background: #e17055; }
.c-red    { background: #e84118; }


/* Input Taille */
.size-input.box { 
    flex-grow: 1; background: white !important;
    border: 1px solid #ddd !important; text-align: center;
    min-width: 50px; height: 30px !important; padding: 0 5px !important;
    vertical-align: middle;
}

.trash-btn {
    border: none; background: transparent; cursor: pointer; 
    color: #000000ff; opacity: 0.6; padding: 0 5px; font-size: 0.9rem;
    height: 30px; display: flex; align-items: center;
}
.trash-btn:hover { opacity: 1; }

#report-text {
    width: 100%; min-height: 80px; border: none; background: transparent;
    resize: none; font-family: inherit; color: var(--md-default-fg-color);
    overflow: hidden; line-height: 1.4; padding: 0; font-size: 0.85rem;
    outline: none !important;
    box-shadow: none !important;
}
#report-text:focus { outline: none !important; box-shadow: none !important; }

.copy-row { display:flex; justify-content: flex-end; align-items:center; gap:.6rem; margin-top:.35rem; }
.copy { border:1px solid var(--md-default-fg-color--lighter); background:transparent; border-radius:.4rem; padding:.25rem .6rem; cursor:pointer; color: var(--md-default-fg-color); font-size: 0.8rem;}
.copy:hover { background: var(--md-default-fg-color--lightest); }
.copied { font-size:.8rem; opacity:.8; color: green; }
.actions { margin-top: .8rem; display: flex; justify-content: flex-end; }
.clear { background: transparent; border: none; color: var(--md-default-fg-color); opacity: 0.6; cursor: pointer; font-size: 0.8rem; }
.clear:hover { background: var(--md-default-fg-color--lightest); }
</style>

<script>
// --- CONFIGURATION SIMPLIFIÉE ---
// k: identifiant interne, lbl: texte affiché, v: valeur score, c: couleur hexa, cls: classes CSS
const PIRADS_CONF = [
    { k: '1',   lbl: '1',  v: 1, c: '#4cd137', cls: 'c-green' },
    { k: '2',   lbl: '2',  v: 2, c: '#c4e538', cls: 'c-yellow' },
    { k: '2+1', lbl: '+1', v: 3, c: '#fbc531', cls: 'c-gold pi-sm' }, // Texte +1, Couleur du 3
    { k: '3',   lbl: '3',  v: 3, c: '#fbc531', cls: 'c-gold' },
    { k: '3+1', lbl: '+1', v: 4, c: '#e17055', cls: 'c-orange pi-sm' }, // Texte +1, Couleur du 4
    { k: '4',   lbl: '4',  v: 4, c: '#e17055', cls: 'c-orange' },
    { k: '5',   lbl: '5',  v: 5, c: '#e84118', cls: 'c-red' }
];

const TARGET_WIDTH = 320; 
let CANVAS_HEIGHT = 400; 

// --- STATE ---
let canvas;
let uniqueIdCounter = 0; 
let lesions = []; 

// --- INIT ---
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
    
    canvas.on('object:moving', function(e) {
        if(e.target.internalId) detectZoneDetailed(e.target);
    });
    canvas.on('object:scaling', function(e) {
        const obj = e.target;
        if(obj.type === 'group' && obj.internalId) {
            obj.item(1).set({ scaleX: 1/obj.scaleX, scaleY: 1/obj.scaleY });
        }
    });
}

function setupBackground(img) {
    const ratio = img.height / img.width;
    CANVAS_HEIGHT = TARGET_WIDTH * ratio;
    canvas.setDimensions({ width: TARGET_WIDTH, height: CANVAS_HEIGHT });
    img.set({ scaleX: TARGET_WIDTH/img.width, scaleY: CANVAS_HEIGHT/img.height, originX: 'left', originY: 'top' });
    canvas.setBackgroundImage(img, canvas.renderAll.bind(canvas));
}

// --- ALGORITHME DE ZONAGE V8 (OFFSET -22px) ---
function detectZoneDetailed(fabricObj) {
    const x = fabricObj.left;
    const y = fabricObj.top;
    const id = fabricObj.internalId;

    const H = CANVAS_HEIGHT;
    const W = TARGET_WIDTH;
    const CENTER_X = (W / 2) - 22; // Correctif de centrage (Offset -22px)

    // Hauteurs
    const Y_LIMIT_BASE_MID = H * 0.33;
    const Y_LIMIT_MID_APEX = H * 0.67;

    // Lignes Rouges (a/p)
    const Y_DASH_BASE = H * 0.17; 
    const Y_DASH_MID = H * 0.50;  
    const Y_DASH_APEX = H * 0.83; 

    // Limites Latérales Grille
    const MEDIAL_LIMIT_X = W * 0.30; 

    // Limites TACHE VERTE (ZT)
    const GREEN_RAD_X_BASE = W * 0.28; 
    const GREEN_RAD_X_MID = W * 0.30;
    const GREEN_RAD_X_APEX = W * 0.22;

    const GREEN_BOTTOM_BASE = H * 0.25; 
    const GREEN_BOTTOM_MID = H * 0.58;  
    const GREEN_BOTTOM_APEX = H * 0.88;

    // Variables de calcul
    let heightStr = ""; 
    let sideStr = (x < CENTER_X) ? "droite" : "gauche";
    let dist = Math.abs(x - CENTER_X);
    
    let yDash = 0;
    let greenRadX = 0;
    let greenBottom = 0;
    let baseZoneNum = 0;
    
    // Rayon du SFMA dynamique
    let stromaRadX = 0; 

    if (y < Y_LIMIT_BASE_MID) {
        heightStr = "basale";
        yDash = Y_DASH_BASE;
        greenRadX = GREEN_RAD_X_BASE;
        greenBottom = GREEN_BOTTOM_BASE;
        baseZoneNum = 1;
        stromaRadX = W * 0.13; 
    } else if (y < Y_LIMIT_MID_APEX) {
        heightStr = "médio-lobaire";
        yDash = Y_DASH_MID;
        greenRadX = GREEN_RAD_X_MID;
        greenBottom = GREEN_BOTTOM_MID;
        baseZoneNum = 3;
        stromaRadX = W * 0.09; 
    } else {
        heightStr = "apicale";
        yDash = Y_DASH_APEX;
        greenRadX = GREEN_RAD_X_APEX;
        greenBottom = GREEN_BOTTOM_APEX;
        baseZoneNum = 5;
        stromaRadX = W * 0.06; 
    }

    // 1. Grille : Secteur a/p
    let isAnt = (y < yDash);
    let sectorChar = isAnt ? "a" : "p";

    // 2. Grille : Médian/Latéral
    let isMedialGrid = (dist < MEDIAL_LIMIT_X);
    let finalZoneNum = isMedialGrid ? baseZoneNum : (baseZoneNum + 1);
    
    if (sideStr === "gauche") finalZoneNum += 6;
    let zoneCode = finalZoneNum + sectorChar;

    // 3. Type de Tissu
    let regionType = "PZ"; 

    // Vérification SFMA avec le nouveau rayon dynamique
    if (isAnt && dist < stromaRadX) {
        regionType = "AS";
        if (heightStr === "basale") zoneCode = "13a";
        else if (heightStr === "médio-lobaire") zoneCode = "14a";
        else zoneCode = "15a";
    } else {
        // Test ZT (Vert)
        let isInGreen = (dist < greenRadX) && (y < greenBottom);
        if (isInGreen) {
            regionType = "TZ"; 
        } else {
            regionType = "PZ"; 
        }
    }

    const lesion = lesions.find(l => l.internalId === id);
    if(lesion) {
        lesion.heightStr = heightStr;
        lesion.sideStr = sideStr;
        lesion.sectorStr = (sectorChar === 'a') ? "antérieure" : "postérieure";
        lesion.zoneCode = zoneCode;
        lesion.regionType = regionType;
    }
    updateReport();
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

function autoResizeTextarea() {
    const tx = document.getElementById('report-text');
    tx.style.height = 'auto'; 
    tx.style.height = (tx.scrollHeight + 5) + 'px';
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
    canvas.add(group);
    canvas.setActiveObject(group);
    lesions.push({ 
        internalId: internalId, fabricObj: group, 
        piradsKey: '3', val: 3, size: '',
        heightStr: "médio-lobaire", sideStr: "droite", sectorStr: "antérieure", zoneCode:"??", regionType: "TZ" 
    });
    addLesionRow(internalId);
    detectZoneDetailed(group); 
    sortAndRenameLesions();
}

// --- GESTION DES BOUTONS SIMPLIFIÉE ---
function addLesionRow(internalId) {
    const container = document.getElementById('lesion-rows-container');
    const row = document.createElement('div');
    row.className = 'lesion-row';
    row.id = `lesion-row-${internalId}`;
    
    // Génération simple des boutons
    let btns = PIRADS_CONF.map(p => {
        // Le 3 est sélectionné par défaut
        let sel = (p.k === '3') ? 'selected' : ''; 
        return `<div class="pi-btn ${p.cls} ${sel}" onclick="setLesionScore(${internalId}, '${p.k}', this)">${p.lbl}</div>`;
    }).join('');

    row.innerHTML = `
        <div class="lesion-name">L?</div>
        <div class="pirads-selector">${btns}</div>
        <input type="number" class="size-input box" id="size-${internalId}" placeholder="mm" oninput="updateLesionSize(${internalId})">
        <button class="trash-btn" onclick="removeLesion(${internalId})"><i class="fas fa-trash"></i></button>
    `;
    container.appendChild(row);
}

function setLesionScore(id, key, elem) {
    // 1. Trouver la lésion et la config
    let l = lesions.find(x => x.internalId === id);
    let conf = PIRADS_CONF.find(x => x.k === key);
    if (!l || !conf) return;

    // 2. Mettre à jour les données
    l.piradsKey = key;
    l.val = conf.v;

    // 3. Mettre à jour l'interface (classe selected)
    let parent = elem.parentElement;
    for(let child of parent.children) child.classList.remove('selected');
    elem.classList.add('selected');

    // 4. Mettre à jour le Canvas et la Bordure
    if(l.fabricObj) {
        l.fabricObj.item(0).set('fill', conf.c);
        canvas.requestRenderAll();
    }
    let row = document.getElementById(`lesion-row-${id}`);
    if(row) row.style.borderLeftColor = conf.c;
    
    sortAndRenameLesions();
}

function updateLesionSize(internalId) {
    const lesion = lesions.find(l => l.internalId === internalId);
    if (lesion) { lesion.size = document.getElementById(`size-${internalId}`).value; updateReport(); }
}

function sortAndRenameLesions() {
    lesions.sort((a, b) => {
        if (b.val !== a.val) return b.val - a.val;
        return (parseFloat(b.size)||0) - (parseFloat(a.size)||0);
    });
    const container = document.getElementById('lesion-rows-container');
    lesions.forEach((l, index) => {
        const newLabel = "L" + (index + 1);
        l.label = newLabel;
        if (l.fabricObj) { l.fabricObj.item(1).set('text', newLabel); l.fabricObj.addWithUpdate(); }
        const row = document.getElementById(`lesion-row-${l.internalId}`);
        if(row) { row.querySelector('.lesion-name').innerText = newLabel; container.appendChild(row); }
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
    
    const psa = parseFloat(document.getElementById('psa').value) || 0;
    let txt = "";
    if (psa > 0) txt += `Taux de PSA récemment dosé à ${psa} ng/ml.\n`;
    txt += "Analyse PI-RADS multiparamétrique (T2, diffusion, perfusion).\n\n";
    txt += `Le volume de la glande est estimé à ${vol.toFixed(0)} cc.\n`;
    
    if(psa > 0 && vol > 0) {
        let densStr = (psa / vol).toFixed(2);
        txt += `Densité de PSA évaluée à ${densStr} ng/mL/cc.\n`;
    }

    txt += "Pas d'épaississement significatif du détrusor.\n"; 
    txt += "Pas de dilatation des cavités pyélocalicielles.\n\n";

    const pzLesionsText = [];
    const tzLesionsText = [];
    const asLesionsText = [];
    const significantLesions = []; 

    lesions.forEach((l) => {
        // --- Construction du texte descriptif (Corps du CR) ---
        let locString = "";
        
        if(l.regionType === 'AS') {
            // Pour le SFMA : pas de côté, juste la hauteur
            locString = `du SFMA ${l.heightStr}`; 
        } else {
            // Pour ZT et ZP : Hauteur + Secteur + Côté
            // Ex: "apicale postérieure gauche"
            locString = `${l.heightStr} ${l.sectorStr} ${l.sideStr}`;
        }

        let piradsShort = `PI-RADS ${l.piradsKey}`;
        let desc = `On y individualise une lésion ${locString} ${piradsShort}`;
        if(l.size && l.size > 0) desc += ` de ${l.size} mm`;
        desc += ` (« ${l.label} » dans z${l.zoneCode}).`;

        if(l.regionType === 'PZ') pzLesionsText.push(desc);
        else if(l.regionType === 'TZ') tzLesionsText.push(desc);
        else asLesionsText.push(desc);

        // --- Construction de la conclusion (Dernière ligne) ---
        if (l.val >= 3) {
            let riskText = "équivoque"; 
            if (l.val === 4) riskText = "suspecte"; 
            if (l.val === 5) riskText = "très suspecte"; 
            
            let prep = "dans la";
            let regionFull = "";
            let locDetails = ""; 

            if (l.regionType === 'PZ') {
                regionFull = "zone périphérique";
                // Ordre : Hauteur -> Secteur -> Côté
                locDetails = `${l.heightStr} ${l.sectorStr} ${l.sideStr}`;
            } else if (l.regionType === 'TZ') {
                regionFull = "zone de transition";
                // Ordre : Hauteur -> Secteur -> Côté
                locDetails = `${l.heightStr} ${l.sectorStr} ${l.sideStr}`;
            } else { 
                // Cas SFMA (AS)
                regionFull = "stroma fibromusculaire antérieur";
                prep = "dans le";
                // Pas de secteur (toujours ant), pas de côté pour le SFMA
                locDetails = `${l.heightStr}`;
            }
            
            let conclusionLine = `Lésion ${riskText} (PI-RADS ${l.piradsKey}) ${prep} ${regionFull} ${locDetails}.`;
            significantLesions.push(conclusionLine);
        }
    });

    txt += "La zone périphérique présente quelques remaniements ne gênant pas l'interprétation du signal. ";
    if(pzLesionsText.length > 0) txt += pzLesionsText.join(" ");
    else txt += "Pas de lésion significative décelable.";
    txt += "\n";

    txt += "La zone de transition présente quelques remaniements ne gênant pas l'interprétation du signal. ";
    if(tzLesionsText.length > 0) txt += tzLesionsText.join(" ");
    else txt += "Pas de lésion significative décelable.";
    txt += "\n";

    if(asLesionsText.length > 0) {
        txt += "Le stroma fibromusculaire antérieur est le siège d'une anomalie. " + asLesionsText.join(" ");
    } else {
        txt += "Le stroma fibromusculaire antérieur est fin et très hypointense en T2.";
    }
    txt += "\n\n";

    txt += "Vésicules séminales symétriques d'aspect normal.\nPas d'adénopathie pelvienne significative.\nPas de lésion osseuse suspecte.";

    if(significantLesions.length > 0) {
        txt += "\n\n" + significantLesions.join("\n");
    }

    const area = document.getElementById('report-text');
    area.value = txt;
    autoResizeTextarea();
}

function copySchema() {
    // 1. Enlever la sélection (cadres bleus)
    canvas.discardActiveObject().renderAll();
    
    // 2. Générer l'image HD
    const dataURL = canvas.toDataURL({ format: 'png', quality: 1, multiplier: 1 });

    try {
        // 3. Conversion technique de l'image pour le presse-papier
        const byteString = atob(dataURL.split(',')[1]);
        const mimeString = dataURL.split(',')[0].split(':')[1].split(';')[0];
        const ab = new ArrayBuffer(byteString.length);
        const ia = new Uint8Array(ab);
        for (let i = 0; i < byteString.length; i++) {
            ia[i] = byteString.charCodeAt(i);
        }
        const blob = new Blob([ab], {type: mimeString});
        const item = new ClipboardItem({ [mimeString]: blob });

        // 4. Écriture dans le presse-papier
        navigator.clipboard.write([item]).then(() => {
            // Feedback visuel sur le bouton
            const btn = document.querySelector('button[onclick="copySchema()"]');
            if(btn) {
                const originalHtml = btn.innerHTML;
                btn.innerHTML = '<i class="fas fa-check"></i> Copié !';
                setTimeout(() => btn.innerHTML = originalHtml, 1500);
            }
        }).catch(err => {
            console.error(err);
            alert("Impossible de copier : votre navigateur bloque l'accès au presse-papier (nécessite HTTPS).");
        });

    } catch (e) {
        console.error(e);
        alert("Erreur lors de la création de l'image.");
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

document.getElementById('btn-copy').addEventListener('click', function() {
    const text = document.getElementById('report-text').value;
    navigator.clipboard.writeText(text).then(() => {
        const msg = document.getElementById('msg-copied');
        msg.textContent = 'Copié ✓';
        setTimeout(() => msg.textContent='', 1500);
    });
});
</script>