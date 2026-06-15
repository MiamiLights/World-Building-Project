
### Obiettivi:
1. Realizzazione di una mappa 3D di un'area prestabilita e raccogliere informazioni su tale area tramite impiego di reti neurali (computer vision ?). 
2. Decidere quali algoritmi impiegare e stilare una lista di PRO e CONTRO dell'impiego di un algoritmo piuttosto che un altro. 


### Librerie:
1. https://it.mathworks.com/help/lidar/getstarted.html?s_tid=CRUX_lftnav
2. https://github.com/laspy/laspy/tree/master
3. https://github.com/mcneel/rhino3dm

### Informazioni generali
- Sensori LiDAR: i sensori LiDAR emettono raggi laser che rimbalzano sulle superfici degli oggetti e misurano il tempo che tali raggi impiegano a ritornare, permettendo al sensore di rilevare la posizione di superfici solide. La distribuzione di energia che ritorna al sensore crea delle onde: 
  
  ![[Pasted image 20260615095616.png]]
  
  
Esistono due tipi di LiDAR: 
1. Sensori LiDAR discreti: registrano picchi individuali (discreti) nella curva di energia. Questi picchi vengono registrati. 
2. Sensori LiDAR continui: registrano una distribuzione dell'energia di ritorno. Sono più complessi rispetto a quelli discreti ma catturano più informazioni. 

### Formati file LiDAR
Che si tratti di punti discreti o distribuzioni, spesso i dati LiDAR sono disponibili sotto forma di punti discreti. Un'insieme di punti discreti viene detto "Nuvola" (LiDAR point cloud). 

Ogni punto LiDAR possiede delle caratteristiche: coordinate tridimensionali (X,Y,Z), intensità e classificazione (quest'ultima caratteristica si utilizza per distinguere il tipo di oggetto su cui il laser è rimbalzato; ad esempio, nel caso di un albero il punto viene classificato come "vegetazione"). 

I dati vengono spesso conservati sotto forma di file .las.

#### Struttura generale file .las
1. Header block: contiene la firma (LASF signature), metadata che identificano il progetto, informazioni varie sui record (offset, fattori di scala).
2. VLRs (variable length record): contengono informazioni aggiuntive come SRS (spatial reference system che serve per processare le informazioni geografiche) e metadati personalizzati con informazioni su dimensioni aggiuntive associate ai singoli punti (es. intensità, tempo, colore, ecc.).  
3. Point records: registrazioni effettive dei punti registrati dal sensore con tutte le informazioni tridimensionali associate. 



#### Fusione LiDAR e Camera

Rete neurale siamese


#### Link potenzialmente utili
https://www.nature.com/articles/s41598-023-35170-z
https://www.diva-portal.org/smash/get/diva2:1885830/FULLTEXT01.pdf
https://research.google/blog/lidar-camera-deep-fusion-for-multi-modal-3d-detection/
