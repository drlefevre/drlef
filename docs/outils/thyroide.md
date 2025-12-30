# [Echographie thyroïdienne](https://cireol.net/wp-content/uploads/2017/05/2017-CIREOL-EUTIRADS.pdf){:target="_blank"}

<script src="https://cdnjs.cloudflare.com/ajax/libs/fabric.js/5.3.1/fabric.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">

<div class="box md-typeset" id="thyroid-box">
  
  <div class="pairs">
    
    <div class="pair">
        <select id="echostructure" onchange="updateReport()">
            <option value="hyper" selected>Hyperéchogène</option>
            <option value="hypo">Hypoéchogène</option>
        </select>
        <select id="aspect" onchange="updateReport()">
            <option value="homo" selected>Homogène</option>
            <option value="heter">Hétérogène</option>
        </select>
    </div>

    <div class="pair">
        <select id="contours" onchange="updateReport()">
            <option value="reguliers" selected>Contours réguliers</option>
            <option value="lobules">Contours lobulés</option>
        </select>
        <select id="vascularisation" onchange="updateReport()">
            <option value="none" selected>Pas d'hypervascularisation</option>
            <option value="moderate">Hypervascularisation < 1 m/s</option>
            <option value="intense">Hypervascularisation > 1 m/s</option>
        </select>
    </div>

    <div class="pair">
        <input id="ld" type="number" inputmode="decimal" placeholder="Lobe droit (cc)" oninput="updateReport()" />
        <input id="lg" type="number" inputmode="decimal" placeholder="Lobe gauche (cc)" oninput="updateReport()" />
    </div>

  </div>

  <div class="canvas-container-wrapper">
      <div class="canvas-container">
          <canvas id="thyroidCanvas"></canvas>
          <div class="canvas-controls">
              <button type="button" class="action-btn" onclick="addNoduleVisual()">
                <i class="fas fa-plus"></i> Nodule
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

  <div id="nodule-rows-container"></div>

  <div class="final-actions">
      <button type="button" class="btn-main-copy" id="btn-copy-full" onclick="copyFullReport()">
         CR ± Schéma
      </button>
      
      <button type="button" class="btn-reset" onclick="fullReset()">
         Effacer
      </button>
  </div>
</div>

<style>
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

/* Masquer les flèches (spinners) dans les inputs number */
input[type=number]::-webkit-inner-spin-button, 
input[type=number]::-webkit-outer-spin-button { 
  -webkit-appearance: none; margin: 0; 
}
input[type=number] { -moz-appearance: textfield; }

/* Canvas */
.canvas-container-wrapper { display: flex; justify-content: center; margin-top: 1rem; width: 100%; }
/* Cherchez ce bloc dans votre <style> et modifiez-le */
.canvas-container {
    position: relative; 
    background: white;
    border: 1px solid var(--md-default-fg-color--lighter);
    border-radius: .5rem; 
    overflow: hidden;
    box-shadow: 0 2px 4px -1px rgba(0,0,0,0.1); 
    
    width: 100%;           /* Largeur par défaut (mobile) */
    max-width: 450px;      /* <--- AJOUTER CECI (400px, 450px ou 500px selon votre goût) */
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

/* Lignes Nodules */
#nodule-rows-container { margin-top: 15px; }
.nodule-row {
    background: var(--md-code-bg-color); padding: 4px 6px;
    border-radius: 4px; margin-bottom: 6px;
    border-left: 5px solid #ccc; /* Couleur changera selon EU-TIRADS */
    display: flex; align-items: center; gap: 6px; flex-wrap: nowrap; 
}
.nodule-name { font-weight: bold; font-size: 0.85rem; min-width: 25px; flex-shrink: 0; }

/* BOUTONS EU-TIRADS */
.tirads-btn {
    width: 24px; height: 24px;
    display: flex; justify-content: center; align-items: center;
    border-radius: 4px; cursor: pointer;
    font-weight: bold; font-family: sans-serif; font-size: .8rem; line-height: 1; 
    opacity: 0.35; transition: 0.2s; color: black; flex-shrink: 0; user-select: none;
}
.tirads-btn:hover { opacity: 0.7; }
.tirads-btn.selected { 
    opacity: 1; transform: scale(1.1); 
    box-shadow: 0 1px 3px rgba(0,0,0,0.3); z-index: 1;
}

.tirads-selector { display: flex; gap: 3px; margin-right: 10px; }

/* Inputs Dimensions Nodule */
.nodule-inputs-wrapper { display: flex; gap: 4px; flex-grow: 1; }
.nodule-input {
    width: 100% !important; background: white !important; border: 1px solid #ddd !important;
    height: 28px !important; padding: 0 4px !important; margin: 0 !important;
    font-size: 0.8rem !important; text-align: center;
}

.c-green  { background: #c4e538; } /* 2 */
.c-yellow { background: #FFD966; } /* 3 */
.c-orange { background: #FFA500; } /* 4 */
.c-red    { background: #FF0000; } /* 5 */

.trash-btn {
    border: none; background: transparent; cursor: pointer; 
    color: #555; opacity: 0.5; padding: 0; 
    display: flex; justify-content: center; align-items: center; 
    height: 24px; width: 24px; flex-shrink: 0;
}
.trash-btn:hover { opacity: 1; color: #d63031; }

.final-actions { margin-top: 1.5rem; display: flex; flex-direction: column; align-items: center; gap: 12px; }
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
const TIRADS_CONF = [
    { k: '2', v: 2, c: '#c4e538', cls: 'c-green' },
    { k: '3', v: 3, c: '#FFD966', cls: 'c-yellow' },
    { k: '4', v: 4, c: '#FFA500', cls: 'c-orange' },
    { k: '5', v: 5, c: '#FF0000', cls: 'c-red' }
];

let TARGET_WIDTH = 0;
let CANVAS_HEIGHT = 400; 
let canvas;
let uniqueIdCounter = 0; 
let nodules = []; 
let currentReportData = {}; 

window.addEventListener('load', function() {
    initCanvas();
    updateReport(); 
});

function initCanvas() {
    const container = document.querySelector('.canvas-container');
    TARGET_WIDTH = container.offsetWidth || 800;
    canvas = new fabric.Canvas('thyroidCanvas');
    
    const imagePath = '../assets/thyr.jpg'; 
    
    fabric.Image.fromURL(imagePath, function(img) {
        if(!img) {
             // Fallback si image non trouvée (pour test)
             console.error("Image thyr.jpg introuvable");
             canvas.setWidth(TARGET_WIDTH);
             canvas.setHeight(300);
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

// --- LOGIQUE NODULES ---

function addNoduleVisual() {
    uniqueIdCounter++;
    const internalId = uniqueIdCounter;
    
    const circle = new fabric.Ellipse({
        rx: 30, ry: 30, fill: '#FFD966', stroke: 'black', strokeWidth: 1,
        originX: 'center', originY: 'center', opacity: 0.85 
    });
    const text = new fabric.Text("N?", {
        fontSize: 20, fontFamily: 'Calibri', fontWeight: 'bold',
        originX: 'center', originY: 'center', fill: 'black'
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
    
    // Valeurs par défaut : TIRADS 3
    nodules.push({ 
        internalId: internalId, fabricObj: group, 
        tiradsKey: '3', val: 3,
        d1: '', d2: '', d3: ''
    });
    
    addNoduleRow(internalId);
    sortAndRenameNodules(); 
}

function addNoduleRow(internalId) {
    const container = document.getElementById('nodule-rows-container');
    const row = document.createElement('div');
    row.className = 'nodule-row';
    row.id = `nodule-row-${internalId}`;
    
    // MODIFICATION ICI : on remplace onclick par onmousedown
    let btns = TIRADS_CONF.map(p => {
        let sel = (p.k === '3') ? 'selected' : ''; 
        return `<div class="tirads-btn ${p.cls} ${sel}" data-key="${p.k}" onmousedown="setNoduleScore(${internalId}, '${p.k}')">${p.k}</div>`;
    }).join('');

    row.innerHTML = `
        <div class="nodule-name">N?</div>
        
        <div class="tirads-selector">${btns}</div>
        
        <div class="nodule-inputs-wrapper">
            <input type="number" class="nodule-input" id="d1-${internalId}" placeholder="mm" oninput="updateNoduleDataTemp(${internalId})" onblur="updateNoduleData(${internalId})">
            <input type="number" class="nodule-input" id="d2-${internalId}" placeholder="mm" oninput="updateNoduleDataTemp(${internalId})" onblur="updateNoduleData(${internalId})">
            <input type="number" class="nodule-input" id="d3-${internalId}" placeholder="mm" oninput="updateNoduleDataTemp(${internalId})" onblur="updateNoduleData(${internalId})">
        </div>
        
        <button class="trash-btn" onclick="removeNodule(${internalId})"><i class="fas fa-trash"></i></button>
    `;
    container.appendChild(row);
}

function setNoduleScore(id, key) {
    let n = nodules.find(x => x.internalId === id);
    let conf = TIRADS_CONF.find(x => x.k === key);
    if (!n || !conf) return;

    n.tiradsKey = key;
    n.val = conf.v;

    if(n.fabricObj) {
        n.fabricObj.item(0).set('fill', conf.c);
        canvas.requestRenderAll();
    }
    
    sortAndRenameNodules(); 
}

// Fonction appelée pendant la frappe (oninput)
// Met à jour les données et le texte, mais NE TRIE PAS pour éviter que les lignes sautent pendant qu'on tape
function updateNoduleDataTemp(id) {
    let n = nodules.find(x => x.internalId === id);
    if(n) {
        n.d1 = document.getElementById(`d1-${id}`).value;
        n.d2 = document.getElementById(`d2-${id}`).value;
        n.d3 = document.getElementById(`d3-${id}`).value;
    }
    updateReport();
}

// Fonction appelée quand on quitte le champ (onblur)
// Là on peut trier, car l'utilisateur a fini ce champ.
function updateNoduleData(id) {
    updateNoduleDataTemp(id); 
    sortAndRenameNodules(); // Le tri est maintenant sûr et ne cassera pas le focus suivant
}

function getMaxDim(n) {
    return Math.max(parseFloat(n.d1)||0, parseFloat(n.d2)||0, parseFloat(n.d3)||0);
}

function getVolume(d1, d2, d3) {
    let v1 = parseFloat(d1) || 0;
    let v2 = parseFloat(d2) || 0;
    let v3 = parseFloat(d3) || 0;
    if(v1===0 || v2===0 || v3===0) return 0;
    // Formule ellipsoide
    return (v1 * v2 * v3 * 0.52 / 1000);
}

function sortAndRenameNodules() {
    // 1. Tri des données
    nodules.sort((a, b) => {
        if (b.val !== a.val) return b.val - a.val;
        return getMaxDim(b) - getMaxDim(a);
    });

    const container = document.getElementById('nodule-rows-container');

    // 2. Mise à jour visuelle
    nodules.forEach((n, index) => {
        const newLabel = "N" + (index + 1);
        n.label = newLabel;
        
        // MAJ Canvas
        if (n.fabricObj) { 
            n.fabricObj.item(1).set('text', newLabel); 
            n.fabricObj.addWithUpdate(); 
        }
        
        // MAJ DOM
        const row = document.getElementById(`nodule-row-${n.internalId}`);
        
        if(row) { 
            row.querySelector('.nodule-name').innerText = newLabel; 
            
            // Bordure couleur
            let conf = TIRADS_CONF.find(c => c.k === n.tiradsKey);
            if(conf) row.style.borderLeftColor = conf.c;
            
            // Boutons actifs
            const btns = row.querySelectorAll('.tirads-btn');
            btns.forEach(b => {
                b.classList.remove('selected');
                if(b.getAttribute('data-key') === n.tiradsKey) {
                    b.classList.add('selected');
                }
            });
            
            // --- OPTIMISATION ---
            // On ne déplace la ligne que si elle n'est pas déjà au bon endroit.
            // Cela évite de "tuer" les événements de clic en cours.
            if (container.children[index] !== row) {
                container.appendChild(row);
            }
        }
    });
    
    canvas.requestRenderAll();
    updateReport();
}

function removeNodule(internalId) {
    const nObj = nodules.find(l => l.internalId === internalId);
    if(nObj) canvas.remove(nObj.fabricObj);
    const row = document.getElementById(`nodule-row-${internalId}`);
    if(row) row.remove();
    nodules = nodules.filter(l => l.internalId !== internalId);
    sortAndRenameNodules();
}

// --- GENERATION RAPPORT ---

function updateReport() {
    // 1. Volumes Lobes
    const volD = document.getElementById('ld').value;
    const volG = document.getElementById('lg').value;

    let txt = `Volumes des lobes droit et gauche estimés à ${volD} et ${volG} cc.\n`;

    // 2. Echostructure et contours
    const echoVal = document.getElementById('echostructure').value;
    let echoTxt = "Echostructure hyperéchogène homogène.";
    if(echoVal === 'hyper') echoTxt = "Echostructure hyperéchogène";
    if(echoVal === 'hypo') echoTxt = "Echostructure hypoéchogène";

    const aspectVal = document.getElementById('aspect').value;
    if(aspectVal === 'homo') echoTxt += " homogène";
    if(aspectVal === 'heter') echoTxt += " hetérogène"; 

    const contoursVal = document.getElementById('contours').value;
    if(contoursVal === 'reguliers') echoTxt += " avec contours réguliers.";
    if(contoursVal === 'lobules') echoTxt += " avec contours lobulés."; 

    txt += echoTxt + "\n";

    // 3. Vascularisation
    const vascVal = document.getElementById('vascularisation').value;
    let vascTxt = "Pas d'hypervascularisation au Doppler.";
    if(vascVal === 'moderate') vascTxt = "Hypervascularisation modérée.";
    if(vascVal === 'intense') vascTxt = "Hypervascularisation intense.";
    txt += vascTxt + "\n";

    txt += "Pas d'anomalie du tractus thyréoglosse.\n";

    // 4. Nodules
    if(nodules.length === 0) {
        txt += "Absence d'image nodulaire dans la thyroïde.\n";
    } else {
        nodules.forEach(n => {
            // Nodule EU-TIRADS 4 de 10 x 5 x 5 mm soit 0,1 cc nommé N1 sur le schéma
            let volN = getVolume(n.d1, n.d2, n.d3).toFixed(1).replace('.', ',');
            let d1 = n.d1 || '?';
            let d2 = n.d2 || '?';
            let d3 = n.d3 || '?';
            
            txt += `Nodule « ${n.label} » EU-TIRADS ${n.tiradsKey} de ${d1} x ${d2} x ${d3} mm soit ${volN} cc`;

            let maxD = getMaxDim(n);
            let indication = false;
            
            // Règles cytoponction
            if (n.val === 3 && maxD > 20) indication = true;
            if (n.val === 4 && maxD > 15) indication = true;
            if (n.val === 5 && maxD > 10) indication = true;

            if(indication) {
                txt += ", avec indication théorique à une cytoponction.\n";
            } else {
                txt += ".\n";
            }
        });
    }

    txt += "Pas d'adénopathie dans les secteurs II, III, IV et VI.\n\n";

    currentReportData.text = txt;
    document.getElementById('report-text').value = txt;
}

// --- COPIE PRESSE-PAPIER (HTML + IMG) ---
async function copyFullReport() {
    canvas.discardActiveObject().renderAll();
    const formatHTML = (text) => text ? text.replace(/\n/g, '<br>') : "";

    // Initialisation de la balise image à vide
    let imgTag = "";

    // On vérifie s'il y a des nodules dans le tableau global 'nodules'
    if (nodules.length > 0) {
        let imgData = canvas.toDataURL({ format: 'png', multiplier: 3, quality: 1 });
        imgTag = `
            <p style="text-align: center; margin: 10px 0;">
                <img src="${imgData}" style="max-width: 300px; height: auto;" alt="Schéma Thyroïde">
            </p>`;
    }

    let rawText = currentReportData.text;
    
    // Si imgTag est vide, l'image ne s'affichera pas
    let htmlContent = `
        <div style="font-family: Calibri, sans-serif; font-size: 11pt; color: #000;">
            <p>${formatHTML(rawText)}</p>
            ${imgTag}
        </div>
    `;

    try {
        const blobHtml = new Blob([htmlContent], { type: 'text/html' });
        const blobText = new Blob([rawText], { type: 'text/plain' });
        const data = [new ClipboardItem({ 'text/html': blobHtml, 'text/plain': blobText })];
        await navigator.clipboard.write(data);
        showCopyFeedback(true);
    } catch (err) {
        console.error(err);
        alert("Erreur de copie (Contexte sécurisé HTTPS requis pour ClipboardItem).");
    }
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
        <img src="${imgData}" style="max-width: 300px; height: auto; display: block;" alt="Schéma Thyroïde">
    `;

    try {
        // 4. Préparer les formats pour le presse-papier
        const blobHtml = new Blob([htmlContent], { type: 'text/html' });
        // Fallback texte simple si le HTML n'est pas supporté
        const blobText = new Blob(["[Schéma Thyroïde]"], { type: 'text/plain' });

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
    const imgData = canvas.toDataURL({ format: 'png', multiplier: 3, quality: 1});
    const a = document.createElement('a');
    a.href = imgData;
    a.download = 'schema-thyroide.png';
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
    [...nodules].forEach(n => removeNodule(n.internalId));
    uniqueIdCounter = 0;
    // Reset selects
    document.getElementById('echostructure').selectedIndex = 0;
    document.getElementById('vascularisation').selectedIndex = 0;
    updateReport();
}
</script>

<figure markdown="span">
    <b>4-10 cc/lobe</b> (< 9 cc ♀ et < 8 cc ado), Doppler < 40 cm/s, 20% [lobe pyramidal](https://radiopaedia.org/articles/pyramidal-lobe-of-thyroid){:target="_blank"}  
    nodule EU-TIRADS 3 < 20 mm ou 4 < 15 mm => **surveillance à 1a**, puis 2-3a, puis 5a  
    <br>
</figure>

| [EU-TIRADS](https://cireol.net/wp-content/uploads/2017/05/2017-CIREOL-EUTIRADS.pdf){:target="_blank"} | Critères | Cytoponction | Malignité |
| :---: | :---: | :---: | :---: |
| <span style="background-color:#c4e538; color:black; padding:2px 8px; border-radius:4px; font-weight:bold;">2</span> 6% | anéchogène/spongiforme | compressif | 0% |
| <span style="background-color:#FFD966; color:black; padding:2px 8px; border-radius:4px; font-weight:bold;">3</span> 60% | iso/hyperéchogène | > 20 mm | 3% |
| <span style="background-color:#FFA500; color:black; padding:2px 8px; border-radius:4px; font-weight:bold;">4</span> 30% | modérément hypo | > 15 mm | 15% |
| <span style="background-color:#FF0000; color:black; padding:2px 8px; border-radius:4px; font-weight:bold;">5</span> 4% | très hypo, microCa, irrégulier, h>l | > 10 mm | 50% |

<figure markdown="span">
    **contours irréguliers** = au moins 3 lobulations/spicules, **microCa** = au moins 5    
    **↗ taille significative** +2 mm dans 2 diam. / 50% en volume => [cytoponction](https://lamediatheque.radiologie.fr/mediatheque/media.aspx?mediaId=2896&channel=3277){:target="_blank"}
</figure>

=== "Basedow"
    - goitre hypoéchogène **homogène**, Ac anti-récepteur de la TSH (TRAK)
    - hypervascularisation intense **> 1 m/s** ("thyroid inferno" > 50% parenchyme)
    - récidive : plus hétérogène et moins vascularisé
    <figure markdown="span">
        ![](assets/Basedow.jpg){width="500"}
    </figure>
=== "Hashimoto"
    - goitre hypoéchogène **micronodulaire**, Ac anti-TPO +/- anti-Tg
    - hypervascularisation modérée **< 1 m/s**
    - suivi/an, travées fibreuses hyperécho, /!\ lymphome/cancer
    <figure markdown="span">
        ![](assets/Hashimoto.jpg){width="500"}
    </figure>
=== "De Quervain"
    - contexte viral, douloureux
    - plages hypoéchogènes mal limitées peu vascularisées
    - contrôle à M3 si pseudonodulaire
    <figure markdown="span">
        ![](assets/Quervain.jpg){width="500"}
    </figure>
=== "Hyperpara"
    - 80% [adénome](https://radiopaedia.org/articles/parathyroid-adenoma){:target="_blank"} parathyroïdien > 15% hyperplasie > 5% carcinome
    - nodule hypoéchogène homogène > 1 cm + hyperhémie
    - à confronter à scintigraphie MIBI / TEP choline
    <figure markdown="span">
        ![](assets/hyperpara.jpg){width="500"}
    </figure>