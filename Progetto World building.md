
### Obiettivi:
1. Realizzazione di una mappa 3D di un'area prestabilita e raccogliere informazioni su tale area tramite impiego di reti neurali (computer vision ?). 
2. Decidere quali algoritmi impiegare e stilare una lista di PRO e CONTRO dell'impiego di un algoritmo piuttosto che un altro. 


### Librerie:
1. https://it.mathworks.com/help/lidar/getstarted.html?s_tid=CRUX_lftnav
2. https://github.com/laspy/laspy/tree/master
3. https://github.com/mcneel/rhino3dm
4. https://www.open3d.org/

### Informazioni generali
- Sensori LiDAR: i sensori LiDAR emettono raggi laser che rimbalzano sulle superfici degli oggetti e misurano il tempo che tali raggi impiegano a ritornare, permettendo al sensore di rilevare la posizione di superfici solide. La distribuzione di energia che ritorna al sensore crea delle onde: 
  
  ![[Pasted image 20260615095616.png]]
  
  
Esistono due tipi di LiDAR: 
1. Sensori LiDAR discreti: registrano picchi individuali (discreti) nella curva di energia. Questi picchi vengono registrati. 
2. Sensori LiDAR continui: registrano una distribuzione dell'energia di ritorno. Sono più complessi rispetto a quelli discreti ma catturano più informazioni. 

Il **numero di ritorno**(return number) di un punto indica la posizione d'ordine di quel preciso punto nella sequenza dei riflessi. Un raggio emesso dal sensore può rimbalzare su una o più superfici prima di ritornare al sensore. Mettiamo caso che un fascio di luce rimbalzi prima su un albero e successivamente sul terreno. Ad ogni rimbalzo il sensore rileva un picco di energia e registra il punto corrispondente. Il punto generato dal rimbalzo sull'albero avrà come return number 1, mentre il punto generato dal secondo rimbalzo sul terreno avrà numero di ritorno 2. Quindi il numero di ritorno corrisponde alla posizione di un dato punto nella sequenza dei rimbalzi. 

Il **Numero totale di ritorni** (number of returns) rappresenta il conteggio complessivo di tutti i riflessi generati dal quel singolo impulso laser. Risponde alla domanda: "In quanti pezzi totali si è diviso questo raggio laser durante il viaggio?".  E' un numero fisso e identico per tutti i punti nati dallo stesso impulso.Numero totale di ritorni

Esempio:

Un singolo raggio laser colpisce una foglia, poi un ramo e infine il terreno. Questo impulso genera 3 punti. 
1. Il punto sulla foglia alta: 
	1.  return number 1 (è stato il primo impatto)
	2.  number of returns 3 (l' impulso totale ha prodotto 3 record)
2. Il punto sul ramo:
	1. return number 2
	2. number of returns 3
3. il punto sul terreno:
	1. return number 3
	2. number of returns 3

Perché questa distinzione è importante? 

Se un punto ha Return Number = 1 e Number of Returns = 1, significa che il laser ha colpito un oggetto solido (es. un tetto o l'asfalto) ed è tornato indietro subito senza dividersi. Se invece ha Return Number = 1 e Number of Returns = 4, sappiamo istantaneamente che quel punto è la cima di una vegetazione molto fitta attraverso cui il laser è riuscito a penetrare a fondo.
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

Se si vogliono usare una telecamera e un sensore lidar insieme bisogna fare dei passi preliminari per permettere una sincronizzazione tra i due. Fisicamente i due componenti potrebbero essere posizionati in maniera diversa. Per ovviare a questa differenza si utilizza una matrice di trasformazione, ossia una matrice 4x4 che in algebra si usa per cambiare la posizione, orientamento o dimensione di un oggetto tridimensionale. Converte le coordinate di un punto tridimensionale (x, y, z) in nuove coordinate (x', y', z') attraverso moltiplicazione standard matriciale. 

![[Pasted image 20260617142429.png]]


La parte 3x3 in alto a sinistra indicata con R è la matrice di rotazione. Contiene i numeri che dicono di quanto il LiDAR è ruotato rispetto alla camera. 

La colonna a destra indicata con t è il vettore di traslazione. Dice semplicemente di quanti centimetri un sensore è spostato nello spazio rispetto a all'altro. 

L'ultima riga serve solo a far quadrare i conti. 

Una volta calcolata questa matrice, si può prendere qualsiasi punto (X, Y, Z) del LiDAR e "proiettarlo" matematicamente sulle coordinate (u, v) dell'immagine 2D. 

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


