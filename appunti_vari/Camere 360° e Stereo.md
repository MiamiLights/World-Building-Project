### Camere Stereoscopiche:
Simulano la visione umana, sono composte da due sensori posti ad una distanza fissa, chiamata **baseline**.

Entrambi i sensori scattano una foto nello stesso istante. Un software analizza le differenze di posizione degli stessi oggetti nelle due immagini (**disparità**). Più un oggetto è vicino, maggiore sarà la sua disparità tra le due immagini. Attraverso la [[Accenni Matematici#Triangolazione|triangolazione]], il sistema calcola la profondità **Z**. 

Output Foto (Statico): Genera istantaneamente una [[#mappa di profondità]] 3D associata allo scatto.
Video (Tempo reale): Genera un flusso costante di dati RGB-D (Colore + Profondità). Permette il tracciamento degli oggetti in movimento e l'applicazione di filtri temporali per eliminare il rumore di calcolo tra un frame e l'altro.

Vantaggi: precisione per quanto riguarda oggetti vicini alla camera. Mappa di profondità.
Svantaggi: l'errore di profondità cresce man mano che l'oggetto si allontana. Se un muro è completamente bianco e uniforme, il software non riesce a trovare punti di corrispondenza tra le due foto, fallendo nel calcolo della disparità.

Algoritmi ricostruzione 3D con Camere Stereo:
	• Stereo SLAM: ORB-SLAM3 / RTAB-Map tracciano il movimento della telecamera nello spazio sfruttando la baseline per avere subito la scala reale in metri, generando una nuvola di punti.
	• Fusione Volumetrica: algoritmi come TSDF uniscono i singoli frame di profondità in una griglia di voxel per eliminare il rumore di fondo. 
	• Generazione della mesh: Poisson Surface Reconstruction trasforma poi i punti in una mesh (superficie poligonale)

###### Mappa di profondità: 
..una foto normale è una griglia di pixel in cui ogni pixel contiene informazioni sul colore (rosso, verde, blu). Una mappa di profondità è un'immagine speciale delle stesse dimensioni, in cui ogni pixel non contiene un colore, ma un valore numerico che indica la distanza geometrica dalla fotocamera.
Per convenzione visiva, spesso le tonalità più chiare/scure indicano la transizione da oggetti vicini a lontani.
Disparità alta -> oggetto vicino, lontano altrimenti

## Camere a 360 gradi
Utilizzano due o più lenti grandangolari, per catturare l'intero ambiente circostante in un'unica immagine sferica (proiezione equirettangolare).
I singoli scatti vengono uniti tramite un processo software chiamato stitching (cucitura) per creare un'unica immagine sferica.

La ricostruzione 3D da immagini o video a 360 gradi è più complessa rispetto alle camere stereo, perchè non abbiamo una "profondità nativa". Per ricostruire l'ambiente, ci sono due grandi famiglie di algoritmi: quelli basati su geometria pura (visione artificiale classica) e quelli basati su IA (deep learning).

Approccio **geometrico** : 
Si muove la camera nello spazio e si analizza il parallasse (fenomeno per cui un oggetto sembra spostarsi di posizione in funzione del punto di osservazione) generato dal movimento:
• Feature Extraction Sferica: il punto di partenza è trovare punti di riferimento stabili nelle immagini equirettangolari usando algoritmi di Feature Extraction e Matching: Spherical-ORB, SphiSFM, ecc.
• SfM (Structure from Motion): trova i punti comune tra le varie foto e risolve il problema Bundle Adjustment: calcola contemporaneamente la posizione esatta in cui si è scattata ogni foto e la posizione 3D nello spazio di tutti i punti della stanza.
• Visual Slam : video in tempo reale Visual SLAM (Simultaneous Localization and Mapping) il video viene elaborato calcolando fotogramma per fotogramma dove si trova la camera nello spazio e creando una nuvola di punti dell'ambiente circostante l'algoritmo. Da vedere: OpenVSLAM/ORB-SLAM3.

Approccio Deep Learning:
stima della profondità analizzano l'immagine equirettangolare e generano una mappa di profondità: UniFuse, BiFuse, OmniData, PanoFormer.
Algoritmi di Layout Estimation: per mappare una stanza in 3D capiscono dove sono pavimento, soffitto e muri. HorizonNet / LED2-Net trovano gli angoli della stanza e generano un modello 3D semplificato della stanza.

3D Gaussian Splatting e NeRF:
3DGS: prende fotogrammi del video a 360 e calcola la posizione della camera (SLAM/SfM) e poi riempie lo spazio tridimensionale con milioni di particelle 3D semitrasparenti e colorate. Permette di ricostruire una stanza intera in pochi minuti di elaborazione.
NeRF: prendono pochi scatti o un video a 360 e usano una rete neurale per rappresentare l'intero ambiente 3D come una funzione di luce e densità.

Camera a 360° riduce i tempi di scansione.
Vantaggi: copertura totale dell'ambiente, facile per scansionare stanze.
Svantaggi: forti distorsioni della lente ai poli. Se un oggetto si trova troppo vicino alla linea di giunzione tra le due lenti grandangolari, nel modello 3D finale potrebbe apparire tagliato o sdoppiato.



Problema dell'altezza: distorsione, perdita di dettagli. Bisogna allontanarsi per non distorcere troppo. Bisogna scansionare a quote differenti?

# SLAM
SLAM (Simultaneous Localization and Mapping) è, in geometria computazionale, il problema della costruzione o aggiornamento di una mappa di un ambiente sconosciuto. 
Permette ad un dispositivo di muoversi in un ambiente sconosciuto, capire dove si trova e contemporaneamente costruire una mappa dell'ambiente.
Lo SLAM si divide in:
1) LiDAR SLAM: utilizza sensori LiDAR che emettono impulsi di luce e misurano il tempo di ritorno per calcolare le distanze degli ostacoli.
2) Visual SLAM (VSLAM): vengono sostituiti i laser con una o più telecamere e vengono elaborati i fotogrammi per navigare nello spazio. 

VSLAM:
Classificazione per Hardware:
1) VSLAM Monoculare: usa una sola telecamera standard, non abbiamo percezione della profondità. Ambiguità di scala: non capiamo se una stanza è reale o un modellino.
2) VSLAM Stereoscopico: usa telecamere separate da distanza fissa (baseline), calcola la profondità istantanea e risolve il problema della scala metrica.
3) VSLAM RGB-D: Usa una telecamera standard accoppiata a un sensore di profondità, fornisce direttamente colore e profondità per pixel.
4) Visual-Inertial SLAM: unisce le telecamere a una IMU (accelerometro+giroscopio). L'IMU traccia il movimento per frazioni di secondo, evitando che il sistema si perda.

Classificazione per Algoritmo
1) Metodi Indiretti, basati sulle caratteristiche (Feature-based), il software non analizza tutta l'immagine, ma estrae solo alcuni punti geometrici salienti (chiamati features o angoli). Segue questi punti matematici da un frame all'altro e triangolarizza la loro posizione 3D. Es: ORB-SLAM, OpenVSLAM. Sono molto robusi ai campi di illuminazione, leggeri computazionalmente, eccellenti per la localizzazione a lungo termine grazie alla facilità di fare Loop Closure (riconoscimento dei luoghi già visitati). Generano mappe "sparse", un dispositivo vede gli ostacoli principali ma non vede la superficie continua di un muro liscio. 
2) Metodi Diretti (Dense / Semi-Dense): non estraggono punti di interesse. Analizzano direttamente l'intensità della luminosità e il colore di tutti i pixel tra fotogrammi consecutivi. Es: LSD-SLAM, DSO. Possono generare mappe dense e superfici continue anche su aree dove non ci sono angoli netti o dettagli evidenti. 
3) SLAM semantico e IA:
	1) Semantic Slam: viene unite la geometrica alle reti neurali di Object Detection (es. YOLO). Il dispositivo mobile non vede più solo una nuvola di punti ma capisce cosa ti trova davanti.
	2) Neural SLAM (NeRF e Gaussian Splatting) 




# COLMAP
Software open-source, permette di ricostruire modelli 3D a partire da una sequenza di fotografie 2D. Strumento standard per generare i dati di partenza necessari per addestrare gli algoritmi come NeRF o 3DGS.
Il processo di ricostruzione si divide principalmente in due macro-fasi:
1) Structure from Motion: questa fase serve a capire da dove sono state scattate le foto e a creare una prima nuvola di punti sparsa:
	1) Feature Extraction e Matching: il software analizza le immagini alla ricerca di punti di interesse geometrici (usando algoritmo SIFT) e cerca gli stessi punti nelle altre foto.
	2) Dopo aver verificato che le corrispondenze siano geometricamente corrette, calcola la posizione tridimensionale della fotocamera per ogni scatti e la posizione nello spazio dei punti chiave individuati, generando una nuvola di punti sparsa.
2) Multi-View Stereo: una volta nota la posizione delle telecamere, si passa alla fase densa:
	1) Utilizzando l'algoritmo PatchMatch, il software calcola la profondità per ogni singolo pixel di ogni immagine. Il risultato sarà una nuvola di punti densa e una mesh 3D.

es: colmap automatic_reconstructor \ 
	--workspace_path ./colmap_output \ 
		--image_path ./immagini_biscotto \ 
			--quality medium
il file database.db che verrà creato conterrà tutti i dati geometrici grezzi estratti dalle immagini prima che inizi la ricostruzione 3D. Vengono memorizzati l'elenco delle immagini caricate, keypoint (feautre importanti come angoli o dettagli estratti da ogni foto tramite SIFT), e i match, ovvero i collegamenti tra punti simili.
La directory sparse/0 conterrà il risultato della ricostruzione sparsa (SfM), camera.bin contiene i parametri intrinseci delle fotocamere (lunghezza focale, distorsione lente), image.bin contiene la posizione e l'orientamento nello spazio 3D di ogni singola fotografia (la posa della fotocamera), point3D.bin contiene le coordinate 3D della nuvola di punti.

