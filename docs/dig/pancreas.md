=== "PA"
    |  [CTSI](https://www.radeos.org/maladie/fiche-pancreatite-aigue_1136.html){:target="_blank"} J3 |   |  nécrose pancréatique |  | 
    | :----------: | :-------: | :----------: | :-------: |
    | `élargissement` | 1 (B) | `≤ 30%` | 2 |
    | `infiltration` | 2 (C) | `30-50%` | 4 |
    | `une/plrs collections` | 3/4 (D/E) | `≥ 50%` | 6 |

    ``` mermaid
    flowchart TD
        A(nécrose) -->|oui 15%| B(pancréatite nécrosante);
        B -->|nécrose péripancréatique isolée 20%| C(collection nécrotique aiguë);
        B -->|nécrose mixte 75%| C;
        C -->|4 semaines| D(nécrose organisée pancréatique);

        A -->|non 85%| F(pancréatite œdémato-interstitielle);
        F --> G(collection liquidienne aiguë péripancréatique);
        G -->|4 semaines| H(pseudo-kyste);
    ```

    !!! danger "[complications](https://radiopaedia.org/articles/acute-pancreatitis){:target="_blank"}"
        - a. splénique et gastro/pancréatico-duodénales = **faux-anévrysme** / irrégularité
        - thrombose veine splénique/VMS/TP => infarctus splénique, HTP segmentaire
        - rupture du conduit pancréatique principal (40% en cas de nécrose isthmique)
        - perforation colique, épaississement pariétal et sténose digestive

=== "PC"
    <figure markdown="span">
        ![](assets/PC.jpg){width="380"}
    </figure>

    !!! info "[causes](https://radiopaedia.org/articles/chronic-pancreatitis-2){:target="_blank"}"
        - OH et tabac +++, hyperCa<sup>2+</sup>, IRC, pancréas *divisum* 
        - [auto-immune](https://radiopaedia.org/articles/autoimmune-pancreatitis){:target="_blank"} (maladie à IgG4 avec ± foyers de néphrite, périaortite et cholangite)

    <figure markdown="span">
        ![](assets/dkpa.jpg){width="300"}
        DKPA = [pancréatite paraduodénale](https://radiopaedia.org/articles/paraduodenal-pancreatitis){:target="_blank"} = pancréatite du sillon  
        H50 OH-tabac, épicentre au niveau de la papille mineure,  
        paroi D2 épaissi avec PDC marquée et kystes
    </figure>


=== "ADK"
    <figure markdown="span">
        TDM = 2 verres d'eau juste avant, IV-, **artériel tardif à 40s** à 4 ml/s, veineux à 70s
    </figure>
    
    !!! tip "[adénocarcinome pancréatique](https://radiopaedia.org/articles/pancreatic-ductal-adenocarcinoma-4){:target="_blank"}"
        - lésion céphalo-isthmique vs corporéo-caudale (en dh bord gauche VMS)
        - extension vasculaire : aorte, tronc cœliaque, AH, AMS, VMS, TP, VCI, vx spléniques
        - **cavernome portal** = CI résection car risque hémorragique
        - **ligament arqué** 10% /!\ ischémie hépatique si DPC
        - **artère hépatique** droite (naissance de l'AMS = 10%)

    !!! warning "résécabilité (classification [NCCN](https://www.snfge.org/sites/www.snfge.org/files/tncd/2024-05/tncd_chap-09-cancer-pancre%CC%81as_2024-05-17_1.pdf){:target="_blank"}) = TDM < 30j avant chirurgie"
        - 20% résécable = aucun contact artériel, contact VMS/TP < 180° sans sténose
        - borderline = contact AMS/TC/AHC/VCI, thrombose VMS/TP reconstructible
        - localement avancé = **AMS/TC > 180°** ou **thrombose VMS/TP étendue**

    !!! danger "méta = TDM thoracique + IRM hépatique"
        - 80% **foie** (DD abcès en IRM = tb perfusionnels, PDC annulaire précoce)
        - 15% péritoine / gg à distance
        - 5% poumons

    !!! tip "causes **carcinose** = pancréas, estomac, CCR, ovaires, sein (lobulaire)"


=== "TNE"
    <figure markdown="span">
        ![](assets/TNE.jpg){width="300"}
        [80% non fonctionnelles](https://radiopaedia.org/articles/pancreatic-neuroendocrine-tumours-2){:target="_blank"}  
        grande taille avec kyste/nécrose/sang  
        </br>
        ![](assets/metashyper.jpg){width="300"}
        métas hépatiques hypervascularisées  
        DD = sein, rein, mélanome, thyroïde  
        </br>
        ![](assets/insulinome.jpg){width="300"}
        [insulinome](https://radiopaedia.org/articles/insulinoma){:target="_blank"} = bénin dans 90%, NEM1  
        surveillance si < 2 cm avec preuve histo  
        </br>
        ![](assets/gastrinome.jpg){width="300"}
        triangle du [gastrinome](https://radiopaedia.org/articles/gastrinoma){:target="_blank"} /!\ 90% malin  
        25% dans le pancréas < paroi duodénale et ganglions
    </figure>  


=== "TIPMP"
    ```
    Séquences axiale diffusion, axiales et coronale T2, bili-IRM 2D et 3D, et axiale T1 sans injection de produit de contraste.   

    Pas de signe d'atypie, notamment pas d'anomalie de signal en diffusion, de portion tissulaire, ni de dilatation du canal pancréatique principal.    
    ```

    <figure markdown="span">
        ![](assets/tipmp.jpg){width="300"}
        [TIPMP](https://radiopaedia.org/articles/intraductal-papillary-mucinous-neoplasm){:target="_blank"} des canaux secondaires = 50% dans l'**uncus** = 5% malignité  
        </br>
        ![](assets/tipmpcpp.jpg){width="300"}
        atteinte du CPP si dilatation sans sténose **> 6 mm** = 60% malignité  
    </figure> 

    !!! warning "[indications chirurgicales](https://www.fmcgastro.org/texte-postu/postu-2019-paris/prise-en-charge-des-tipmp-recommandations-europeennes/){:target="_blank"} (sinon surveillance M6, 1 an, puis /an)"
        - absolues = **ictère**, **nodule mural intrakystique ≥ 5 mm**, **CPP > 10 mm**
        - relatives = PA, kyste > 4 cm ou > 5mm/an, CPP 5-9 mm, nodule mural < 5 mm

    !!! info "[DD TKA](https://www.fimatho.fr/images/site-cracmo/ressources-pros/tka-mini-revue.pdf){:target="_blank"} F45"
        <figure markdown="span">
            kystes multiples accolés + calcifications + pas de communication avec CPP
        </figure> 


=== "kystes"
    <figure markdown="span">
        [![](assets/kystespanc.jpg){width="550"}](https://radiopaedia.org/articles/cystic-lesions-of-the-pancreas-differential){:target="_blank"}
    </figure> 

    ``` mermaid
    flowchart TD
        D(((paroi fine))) -->|uncus, communication CPP, multiples| L(TIPMP);
        D -->|microkystes, 30% calcif centrale, 60a| N(cystadénome séreux);
        D -->|forme molle, graisse| M(malformation lymphatique);
    ```

    ``` mermaid
    flowchart TD
        style F stroke-width:4px
        F((épaisse)) -->|face post. corps/queue, ♀ 50a, cloisons| I(tumeur kystique mucineuse);
        F -->|♀ 30a, sang, portion tissulaire| J(tumeur pseudo-papillaire solide et kystique); 
        F -->|hypervascularisée| K(TNE);
    ```

    <figure markdown="span">
        </br>
        [VHL](https://radiopaedia.org/articles/von-hippel-lindau-disease-5){:target="_blank"} = kystes vrais, cystadénomes séreux et TNE
    </figure> 