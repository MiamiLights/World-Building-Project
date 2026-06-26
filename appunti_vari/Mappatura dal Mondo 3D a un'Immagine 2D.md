## Introduzione

Come descrivere la mappatura (o proiezione) di un punto dal mondo tridimensionale (3D) su un'immagine bidimensionale (2D) catturata da una fotocamera.

Vedremo nel dettaglio come funziona questa trasformazione, che permette di prendere un punto con coordinate $(X, Y, Z)$ in un determinato sistema di riferimento del mondo e mapparlo su un'immagine, ottenendo come risultato una coordinata pixel $(x, y)$.

### Proiezione Centrale

Per questo modello assumiamo che la fotocamera segua il principio della **proiezione centrale**. Si tratta di uno dei modelli più semplici e tipici utilizzati per descrivere il funzionamento delle telecamere, noto anche come modello della **pinhole camera** (camera oscura o a foro stenopeico).

Il principio della proiezione centrale si basa sulla presenza di un unico punto, chiamato **centro di proiezione**. Fondamentalmente, tutti i raggi di luce passano attraverso questo punto. In questo contesto, possiamo descrivere la mappatura di un punto 3D del mondo $(X, Y, Z)$ in una posizione pixel 2D attraverso questa semplice formula:

$$x = PX$$

- $x$ rappresenta le coordinate del pixel.
    
- $P$ è la **matrice di proiezione** (la trasformazione di cui parleremo).
    
- $X$ è il punto nel mondo 3D.
    

### Sistemi di Coordinate

In questo processo di mappatura sono coinvolti ben **quattro sistemi di coordinate**:

1. **Sistema di coordinate del mondo (World Coordinate System):** È il sistema in cui si trova il punto 3D con coordinate $(X, Y, Z)$, dotato di una propria origine.
    
2. **Sistema di coordinate della telecamera (Camera Coordinate System):** È un sistema di coordinate che ha la sua origine $(0, 0, 0)$ esattamente nel centro di proiezione della telecamera. Esprime ogni elemento dello spazio rispetto a questo centro.
    
3. **Sistema di coordinate del piano immagine (Image Plane Coordinate System):** È il piano 2D su cui viene proiettato il mondo 3D.
    
4. **Sistema di riferimento del sensore (Sensor Frame):** Descrive l'effettiva posizione o il numero dei pixel sulla matrice del sensore.
    

### Posizione della Telecamera (Parametri Estrinseci)

Il primo passo consiste nel descrivere dove si trova la telecamera all'interno del sistema di coordinate del mondo. Per farlo, dobbiamo definire:

- La posizione del centro di proiezione, espressa con la variabile $X_0$ (che indica le coordinate $X, Y, Z$ del centro).
    
- Una **matrice di rotazione** (composta da tre parametri di rotazione, come ad esempio _imbardata, beccheggio e rollio_ — _yaw, pitch, roll_), che descrive la direzione verso cui è orientata la telecamera nel mondo 3D.
    

### Proiezione e Parametri Intrinseci

La proiezione dal mondo 3D al piano immagine 2D comporta il passaggio da tre coordinate a due sole coordinate. C'è quindi una **perdita di una dimensione**, intrinseca alla proiezione centrale.

Oltre a questo, ogni telecamera possiede dei parametri interni, chiamati **intrinseci**, che vengono tipicamente codificati in una cosiddetta **matrice di calibrazione ($K$)**. Questi parametri descrivono, ad esempio:

- Come il chip (sensore) è posizionato e allineato rispetto al piano immagine.
    
- La distanza tra il piano immagine e il centro di proiezione (lunghezza focale).
    

In sintesi, questa trasformazione complessiva ha **11 gradi di libertà** (ed è alla base della tecnica chiamata _DLT - Direct Linear Transformation_):

- **6 parametri estrinseci** per la posizione e l'orientamento della telecamera.
    
- **5 parametri intrinseci** (a volte ridotti a 4) per la geometria interna.
    

A questi si aggiunge una serie di parametri non lineari derivanti dalla **distorsione della lente**. A seconda dell'obiettivo utilizzato, sono necessari parametri extra per correggere le distorsioni specifiche che la lente introduce.

### La Matrice di Proiezione e l'Inversione

Mettendo insieme la trasformazione dal sistema del mondo a quello della telecamera, la proiezione e i parametri intrinseci, otteniamo la matrice finale:

> La matrice $P$ è una matrice $3 \times 4$ formata dai parametri interni $K$ e dai parametri estrinseci $R$(rotazione) e $X_0$ (centro di proiezione).

È fondamentale notare che la trasformazione dal mondo alle coordinate pixel ($x = PX$) è una mappatura che **funziona solo in una direzione**. Non può essere invertita direttamente in modo semplice. Poiché passiamo da 3D a 2D, perdiamo l'informazione sulla profondità e non possiamo recuperare l'esatta posizione spaziale del punto partendo da un solo pixel.

Tuttavia, possiamo identificare la **linea retta** nello spazio su cui giace quel punto. Ogni pixel corrisponde di fatto a una direzione nello spazio: tutti i punti che si trovano su quella linea retta verranno mappati sulla stessa identica coordinata pixel.

Non potendo invertire completamente la mappatura, possiamo farlo parzialmente, ottenendo uno spazio di soluzione a una dimensione (1D). Questa retta parte da $X_0$ (il centro di proiezione) e, attraverso l'inversa della matrice di calibrazione e di rotazione, definisce un vettore di direzione nello spazio.

### Ricostruzione 3D (Stereovisione)

Questa proprietà geometrica permette di recuperare le coordinate 3D dei punti se si combinano **più immagini della stessa scena prese da posizioni diverse**.

Se abbiamo un'osservazione dello stesso punto da due posizioni differenti, avremo due rette nello spazio. Il punto in cui queste due rette si intersecano corrisponderà all'esatta posizione del punto 3D nello spazio.

Spero che questa spiegazione sia stata utile. Grazie per l'attenzione!