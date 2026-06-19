
### Obiettivi:
Il progetto mira a sviluppare una pipeline software in grado di generare mappe 3D di un'area geografica prestabilita. Il sistema non si limiterà a ricostruire la geometria tridimensionale ma userà modelli di Deep Learning e Computer Vision per classificare gli elementi presenti nell'ambiente (vegetazione, edifici, ecc). 


### Informazioni generali
- Sensori LiDAR: i sensori LiDAR emettono raggi laser che rimbalzano sulle superfici degli oggetti e misurano il tempo che tali raggi impiegano a ritornare, permettendo al sensore di rilevare la posizione di superfici solide tramite i picchi di energia rilevati. 
  
  ![[Pasted image 20260615095616.png]]

  
## Analisi Comparativa dei Sensori (Pro e Contro)
Per l'acquisizione di dati spaziali e visivi sono stati presi in esame diversi approcci hardware. La scelta finale dipende dal bilanciamento tra budget, accuratezza e densità informativa.

### Approccio Pure LiDAR (3D o Solid-State)

   Pro: 
   - Precisione millimetrica: misurazione diretta e assoluta della distanza, indipendentemente dalle condizioni di illuminazione. 
   Contro: 
   - Costo hardware : i sensori LiDAR 3D hanno costi proibitivi. I sensori Solid-State sono più economici ma limitati nel campo visivo.
   - Assenza di informazioni cromatiche e dettagli (scritte stradali, cartelli)

## Approccio Pure Camera (Monoculare, Stereo o 360)

Pro: 
- Economicità
- Ricchezza semantica: le immagini 2D contengono la massima densità di informazioni di colore, texture e contesto per il riconoscimento degli oggetti. 
Contro: 
- Cecità geometrica: La ricostruzione 3D (tramite stima della profondità) è un'approssimazione matematica. Soffre fortemente i cambi di luce, le ombre e le superfici riflettenti o monocromatiche. 

## Approccio Sensor Fusion (LiDAR + CAMERA) - Approccio Consigliato

Pro: 
- Unisce la certezza geometrica del LiDAR con la ricchezza semantica della telecamera. 

Contro: 
- Richiede una complessa calibrazione spaziotemporale e un carico computazionale elevato. 

## Pipeline Algoritmica 

Il flusso di elaborazione dati è suddiviso in tre macro-fasi gestite tramite librerie Python specializzate.

### Fase 1: Raccolta Dati e Sincronizzazione Hardware

La qualità della mappa 3D finale dipende dalla precisione in fase di acquisizione e dalla corretta calibrazione dei componenti. Per garantire la coerenza geometrica ed evitare disallineamenti spaziali i componenti devono essere fissati su un supporto rigido progettato per azzerare i micro-movimenti. 

#### 1. Sincronizzazione temporale hardware
Per evitare disallineamenti spaziali i flussi dati devono essere sincronizzati al millisecondo:

Soluzioni possibili:
**Sincronizzazione hardware basata su GPS/GNSS** che funge da "orologio maestro" e invia a intervalli di tempo un impulso elettrico per azzerare i contatori interni dei sensori. Dopo l'intervallo il GPS invia ai componenti anche delle stringhe NMEA con i riferimenti temporali corretti. I dati prodotti dai componenti contengono questa stringa di dati temporali che verrà analizzata a tempo di elaborazione. Verranno presi infatti foto e punti con lo stesso timestamp dando quindi la certezza che stiamo riferendoci allo stesso contesto.
   
   Pro:
   - Precisione Assoluta allineata agli orologi atomici dei satelliti
   - Fornisce dettagli posizionali utili per associare la geometria 3D alle coordinate geografiche globali.

   Contro
   - Inutilizzabile in ambienti chiusi o con poco segnale GPS

**Approccio master-slave con microcontrollore** come Master.  Il microcontrollore, collegato ai sensori tramite cavi, invia degli impulsi elettrici che fungono da starter. I componenti eseguono le loro operazioni in maniera sincrona mentre il microcontrollore comunica al PC l'invio di un impulso caratterizzato da un tag. Quando il PC riceve l'immagine e la nuvola di punti questi vengono associati all'impulso comunicato precedentemente dal microcontrollore.  
   
   Pro: 
   - Immunità ambientale
   - Controllo totale su ogni singolo impulso elettrico, potendo creare logiche di controllo personalizzate.

   Contro: 
   - Complessità elettronica e di cablaggio
#### 2. Protocollo di Calibrazione dei Sensori

Prima di avviare la sessione di rilievo effettiva, viene eseguita la procedura di calibrazione geometrica, divisa in due step sequenziali:

#### A. Calibrazione Intrinseca della Telecamera (Modellazione Ottica)

Serve a determinare come la lente specifica proietta il mondo 3D sul sensore 2D e a correggerne i difetti geometrici.

**Metodo:** Si riprende da diverse angolazioni un target noto (scacchiera o pannello a cerchi asimmetrici) usando OpenCV o il MATLAB Camera Calibrator.

**Output:** Si ottiene la matrice intrinseca K e i coefficienti di distorsione della lente (radiale e tangenziale). Ogni immagine raccolta nella pipeline viene immediatamente "rettificata" (distorta al contrario) per renderla geometricamente perfetta.

#### B. Calibrazione Estrinseca LiDAR-Camera (Allineamento Spaziale)

Serve a calcolare la posizione e l'orientamento relativo tra il centro ottico della telecamera e il centro del sensore LiDAR

**Metodo:** Si posiziona un pannello speciale (es. una scacchiera rigida con forti contrasti sia visivi che di riflettanza LiDAR) visibile contemporaneamente a entrambi i sensori.

**Algoritmo:** (DA DEFINIRE) Il software identifica gli angoli della scacchiera nell'immagine 2D e i corrispondenti spigoli tridimensionali nella nuvola di punti LiDAR. Attraverso la risoluzione del problema PnP (_Perspective-n-Point_), calcola la matrice di trasformazione omogenea T (composta da Matrice di Rotazione e Vettore di Traslazione).

#### 4. Output della Fase 1

Il risultato finale della Fase 1 è un dataset pronto per l'elaborazione, composto da:

1. **File .las** strutturati con metadati completi (coordinate, intensità, return number) e timestamp sincroni.
2. **Immagini .png** rettificate (prive di distorsione ottica).
3. **File di calibrazione (calib.json o .mat)** contenente le matrici K e T, che consentiranno alla successiva fase di Sensor Fusion di proiettare istantaneamente i punti 3D sui pixel 2D tramite la formula:
### Fase 2: Ingestione Dati, Filtraggio e Gestione della Memoria

I dati grezzi appena raccolti non possono essere dati in pasto direttamente ai modelli di Deep Learning: i file LiDAR sono troppo pesanti e contengono rumore (es. polvere nell'aria o riflessi), mentre le immagini devono essere corrette geometricamente.

La pipeline di questa fase prevede tre passaggi software:

#### 1. Stream Batching e Parsing Efficiente (`laspy`)

I file .las generati dal LiDAR possono contenere decine di milioni di punti, pesando svariati Gigabyte. Se provassimo a caricarli interamente in memoria con un comando standard, la RAM si saturerebbe immediatamente.

**Soluzione**: si utilizza la libreria laspy in modalità di lettura a blocchi (chunking). L'algoritmo legge solo un pacchetto di punti alla volta, esegue le operazioni di filtraggio e libera la memoria prima di passare al blocco successivo.
#### 2. Filtraggio del Rumore e Undersampling (`Open3D`)

Una volta importati i blocchi di punti, usiamo Open3D per ottimizzare la nuvola:

**Voxel Downsampling:** Lo spazio 3D viene suddiviso in una griglia di cubi virtuali chiamati Voxel. Se all'interno di un singolo cubo sono caduti 50 punti LiDAR, l'algoritmo li fonde calcolandone il baricentro e ne restituisce uno solo. Questo riduce il peso del file anche dell'80% mantenendo intatta la struttura geometrica dell'ambiente.
    
**Rimozione degli Outlier:** Il LiDAR può registrare falsi punti dovuti a particelle di polvere, pioggia o riflessi specchiari. L'algoritmo analizza la distanza media tra i punti: se un punto si trova isolato e troppo lontano dai suoi vicini, viene eliminato automaticamente come rumore di scansione.

### 3. Rettifica delle Immagini (`OpenCV`)

In parallelo al LiDAR, le immagini raccolte dalle telecamere vengono elaborate per eliminare la distorsione ottica della lente (come la curvatura dei bordi):

Utilizzando la matrice intrinseca $K$ e i coefficienti di distorsione calcolati nella Fase 0, la funzione cv2.undistort() di OpenCV "stira" l'immagine pixel per pixel.
Il risultato è un frame rettificato in cui le linee rette del mondo reale (es. i bordi dei palazzi) appaiono perfettamente dritte anche nell'immagine digitale.


## Fase 3: Fusione Geometrico-Semantica (Sensor Fusion)

In questa fase si uniscono la geometria 3D del LiDAR e i dettagli visivi della telecamera.
Grazie a questa fase, ogni punto della nuvola 3D non sarà più un semplice punto grigio nello spazio con coordinate (X, Y, Z), ma riceverà le informazioni sul colore (R, G, B) e sulla classe semantica provenienti dall'immagine.

A questo punto ci sono due opzioni:

##### **1. Early Fusion**: 
Ogni punto 3D $P_{lidar} = (X, Y, Z)$ viene proiettato sui pixel dell'immagine $(u, v)$ tramite la formula:

$$P_{camera} = K \times T \times P_{lidar}$$
**Risultato:** Il punto 3D copia il colore (RGB) del pixel corrispondente, creando una **nuvola di punti fotorealistica**.

**Pro/Contro:** Semplice e leggera, ma soffre di errori di parallasse (occlusioni).

##### **2. Deep Fusion** (DA DEFINIRE) 
Reti neurali 2D (per le immagini) e 3D (per il LiDAR) estraggono le caratteristiche in parallelo e le uniscono in uno spazio comune standardizzato, la **Bird's Eye View (BEV)** (vista dall'alto).

**Risultato:** Massima accuratezza nel riconoscimento degli oggetti anche con scarsa luce o dati parziali.

**Pro/Contro:** Robusta ed efficiente, ma richiede GPU potenti e complessi dataset di addestramento.

## Fase 4: Segmentazione Semantica e Classificazione

Fase dedicata al Deep Learning per comprendere il contesto e catalogare l'ambiente:

**1. Ground Filtering (Isolamento Terreno):** Algoritmi geometrici (es. Cloth Simulation Filter in Open3D) separano la quota del terreno (asfalto, terra) dagli elementi verticali (edifici, alberi).

**2. Riconoscimento Oggetti 3D:** Reti come **PointNet++** analizzano la forma locale dei punti non-terreno per assegnare una classe di appartenenza codificata nel file `.las`:

- Classe 2: Terreno
- Classe 5: Vegetazione / Alberi
- Classe 6: Edifici / Facciate

## Fase 5: Ricostruzione 3D e Modellazione (World Building)

La nuvola di punti discreti viene convertita in un modello geometrico continuo e matematico.

**1. Ricostruzione delle Superfici (Mesh):** Algoritmi come il Poisson Surface Reconstruction (**Open3D**) uniscono i punti organici (terreno, alberi) creando una "pelle" continua di triangoli 3D.

**2. Estrazione delle Primitive:** Gli oggetti rigidi urbani (edifici squadrati, pali) vengono campionati ed estrapolati sotto forma di volumi geometrici puri (**3D Bounding Boxes**) con coordinate, orientamento e dimensioni.

**3. Esportazione CAD/BIM (rhino3dm):** Lo script Python genera direttamente un file .3dm (Rhinoceros), organizzando gli elementi geometrici ricostruiti su livelli separati (TERRENO, EDIFICI, VEGETAZIONE), pronti per il disegno e la modifica manuale.


--------
## Appunti su alcuni aspetti dei file .las

##### Logica dei Ritorni Laser (Return Number e Number of Returns)

I sensori a impulsi discreti registrano i picchi individuali nella curva di energia di ritorno. Un singolo impulso laser può dividersi incontrando superfici parzialmente penetrabili (es. vegetazione):

**Return Number (Numero di Ritorno):** Indica l'ordine sequenziale del riflesso registrato dal sensore per quel singolo raggio.

**Number of Returns (Numero Totale di Ritorni):** Rappresenta il conteggio complessivo di tutti i riflessi generati dallo stesso impulso laser (valore identico per tutti i punti nati dallo stesso raggio).
  
 **Esempio analitico:** Un raggio laser colpisce in sequenza una foglia alta, un ramo intermedio e il terreno, generando 3 record:
 1. Punto sulla foglia: Return Number = 1, Number of Returns = 3.
 2. Punto sul ramo: Return Number = 2, Number of Returns = 3.
 3. Punto sul terreno: Return Number = 3, Number of Returns = 3.     

**Rilevanza applicativa:** L'analisi combinata dei due valori permette di discriminare istantaneamente la tipologia di superficie incontrata:

- Se Return Number= 1 e Number of Returns = 1: l'impulso ha colpito una superficie solida impenetrabile (es. tetto, asfalto).
- Se Return Number = 1 e Number of Returns = 4: il punto identifica la sommità di una vegetazione molto fitta penetrata parzialmente dal fascio laser.
  
### Formati file LiDAR
Che si tratti di punti discreti o distribuzioni, spesso i dati LiDAR sono disponibili sotto forma di punti discreti. Un'insieme di punti discreti viene detto "Nuvola" (LiDAR point cloud). 

Ogni punto LiDAR possiede delle caratteristiche: coordinate tridimensionali (X,Y,Z), intensità e classificazione (quest'ultima caratteristica si utilizza per distinguere il tipo di oggetto su cui il laser è rimbalzato; ad esempio, nel caso di un albero il punto viene classificato come "vegetazione"). 

I dati vengono spesso conservati sotto forma di file .las.
#### Struttura generale file .las
1. Header block: contiene la firma (LASF signature), metadata che identificano il progetto, informazioni varie sui record (offset, fattori di scala).
2. VLRs (variable length record): contengono informazioni aggiuntive come SRS (spatial reference system che serve per processare le informazioni geografiche) e metadati personalizzati con informazioni su dimensioni aggiuntive associate ai singoli punti (es. intensità, tempo, colore, ecc.).  
3. Point records: registrazioni effettive dei punti registrati dal sensore con tutte le informazioni associate (posizione, intensità, dati geografici ecc.). 


### Problemi di memoria durante il data processing

Il file caricato tramite laspy.read() viene caricato completamente in memoria e nel caso di file con molti punti questi peseranno moltissimo in memoria. Soluzioni:
1. Batching dei punti
2. Undersampling dei punti utilizzati per la ricostruzione 3d.


### Problemi nell'impiego di LiDAR e Camera
La scelta del sensore giusto è dettata da requisiti e budget. Consideriamo 4 tipologie di sensori LiDAR:
1. LiDAR mono raggio
2. LiDAR 2D
3. LiDAR 3D
4. LiDAR a stato solido

Il lidar mono raggio è da scartare a priori per questa applicazione in quanto è in grado di rilevare esclusivamente la distanza di un oggetto ma non riesce a dare informazioni spaziali e geometriche. Il LiDAR 2D (il cui nome potrebbe ingannare) misura distanza e angolazione ruotando a 360°, tuttavia non misura l'altezza e, viene principalmente usato per mappare i muri di uno spazio chiuso, sia un magazzino o casa propria. Il LiDAR 3D misura invece anche l'altezza rendendo possibile la registrazione di oggetti tridimensionali, il problema principale di questo modello è il prezzo proibitivo. Il LiDAR A solid state invece non ruota e riesce a rilevare superfici solo all'interno di un cono di fronte ad esso ma riesce a rilevare anche l'altezza. 

La mappatura 3D è possibile impiegando 3D e solid state,  il riconoscimento di oggetti è possibile con entrambi ma nel caso del solid state è più complesso.


Nel caso delle Telecamere abbiamo 3 strade:
1. Telecamera singola (una normalissima GoPro o anche il sensore del telefono)
2. Telecamera stereoscopica (2 canali)
3. Telecamera a 360°

Tutte presentano lo stesso problema: la ricostruzione 3D è complessa perché la telecamera non ha una vera e propria concezione della profondità in quanto non può riconoscere la distanza precisa con gli oggetti come il LiDAR. 
Il vero vantaggio dell'impiego di una fotocamera sta nel riconoscimento degli oggetti. 

Conclusioni: 
LiDAR 3D ottima per fare sia ricostruzione 3D che riconoscimento.
LiDAR Solid State ottima per ricostruzione 3D ma nel caso del riconoscimento il modello utilizzato andrebbe riaddestrato. 

Telecamere varie: Ricostruzione 3D molto complicata ma riconoscimento semplice. 



#### Fusione LiDAR e Camera

##### Calibrazione camera

Prima di poter lavorare con il LiDAR è necessario "calibrare" la telecamera. Utilizzando una libreria Python per la computer vision chiamata openCV e tramite l'utilizzo di una scacchiera, si calcolano due componenti: 
- una matrice intrinseca **K** (la carta di identità ottica della telecamera) che indica al computer come la lente specifica veda il mondo e lo proietta sul sensore 2D. 
- il coefficiente di distorsione: 5 numeri che descrivono l'equazione della curva che la lente crea.

Ogni foto che la telecamera scatta andrà "sistemata" tramite l'utilizzo di questi due componenti avendo come prodotto un'immagine oggettiva senza la distorsione della lente. 
##### Calibrazione camera-LiDAR

Se si vogliono usare una telecamera e un sensore lidar insieme bisogna fare dei passi preliminari per permettere una sincronizzazione tra i due. Fisicamente i due componenti potrebbero essere posizionati in maniera diversa. Per ovviare a questa differenza si utilizza una matrice di trasformazione, ossia una matrice 4x4 che in algebra si usa per cambiare la posizione, orientamento o dimensione di un oggetto tridimensionale. Converte le coordinate di un punto tridimensionale (x, y, z) in nuove coordinate (x', y', z') attraverso moltiplicazione standard matriciale. 

![[Pasted image 20260617142429.png]]


La parte 3x3 in alto a sinistra indicata con R è la matrice di rotazione. Contiene i numeri che dicono di quanto il LiDAR è ruotato rispetto alla camera. 

La colonna a destra indicata con t è il vettore di traslazione. Dice semplicemente di quanti centimetri un sensore è spostato nello spazio rispetto a all'altro. 

L'ultima riga serve solo a far quadrare i conti. 

Ottenuta la matrice di trasformazione che possiamo definire "Matrice estrinseca" la moltiplichiamo per la matrice K intrinseca. 

$$M_{calibrazione} = K \times T$$
###### Come si calcola questa matrice?
Non si fa a occhio. Si usa un oggetto speciale chiamato target di calibrazione. 
Il target è di solito un pannello a scacchiera o un oggetto geometrico preciso che è visibile sia alla telecamera che al LiDAR.

Una volta calcolata la matrice il software esegue questa operazione per ogni singolo punto P del LiDAR $$ P_{camera} = M_{calibrazione} \times P_{lidar}$$
Il risultato è un nuovo punto P_camera che ha solo due coordinate (u, v) sull'immagine. 

Oltre alla posizione spaziale abbiamo un problema di sincronizzazione temporale dobbiamo far corrispondere a un certo insieme di punti una certa immagine. Per fare questo utilizziamo un sistema che assegni dei timestamp a ogni pacchetto dati. Quando il sistema riceve un frame, cerca nel suo database l'immagine e la nuvola di punti che hanno lo stesso timestamp. Se il ritardo tra i due è superiore a pochi ms, il sistema scarta i dati perché sarebbero troppo imprecisi.   

#### Link potenzialmente utili
https://www.nature.com/articles/s41598-023-35170-z
https://www.diva-portal.org/smash/get/diva2:1885830/FULLTEXT01.pdf
https://research.google/blog/lidar-camera-deep-fusion-for-multi-modal-3d-detection/
https://amslaurea.unibo.it/id/eprint/17919/1/TesiAccordino.pdf
https://www.sciencedirect.com/science/article/pii/S0926580522003193

### Algoritmi di classificazione
https://www.neuvition.com/it/technology-blog/classification-algorithms.html  (aggiornato al 2023)
#### Camera-LiDAR fusion
https://towardsdatascience.com/sensor-fusion-kitti-lidar-based-obstacle-detection-part-1-9c5f4bc8d497/

#### Camera-Lidar Calibration
https://github.com/Deephome/Awesome-LiDAR-Camera-Calibration#1-target-based-methods
### Dataset
https://www.cvlibs.net/datasets/kitti/eval_object.php?obj_benchmark=3d

### Introduction to Lidar
https://medium.com/@amnahhmohammed/getting-started-with-lidar-92ff984ec9f3
https://medium.com/@faizalmusthofa69/the-art-of-lidar-901e9ed6348b


## 5. Stack Software e Dipendenze

La libreria **laspy** viene utilizzata per la gestione efficiente dell'input e dell'output dei file nei formati las e laz. Permette il parsing completo dei metadati, inclusi l'Header block e i Variable Length Records, e abilita lo streaming dei dati a blocchi per preservare la memoria RAM.

La libreria **Open3D** costituisce il core geometrico del progetto. Si occupa della visualizzazione tridimensionale, del voxel downsampling per l'undersampling dei punti, della rimozione statistica degli outlier, del calcolo dei vettori normali e dell'applicazione degli algoritmi di mesh generation come la Poisson Surface Reconstruction.

La libreria **rhino3dm** gestisce la fase finale di esportazione del lavoro. Consente la generazione e la scrittura diretta di file nativi in formato 3dm, organizzando le geometrie e i modelli tridimensionali ricostruiti su layer strutturati all'interno dell'ambiente CAD di Rhinoceros.

La libreria **OpenCV** è lo strumento fondamentale per la manipolazione delle immagini e la computer vision 2D. All'interno della pipeline si occupa della calibrazione intrinseca della telecamera tramite l'analisi dei pattern a scacchiera, della successiva rettifica dei fotogrammi per eliminare le distorsioni geometriche della lente e della risoluzione del problema Perspective-n-Point per stimare la posa tra i sensori.


1. https://it.mathworks.com/help/lidar/getstarted.html?s_tid=CRUX_lftnav
2. https://github.com/laspy/laspy/tree/master
3. https://github.com/mcneel/rhino3dm
4. https://www.open3d.org/