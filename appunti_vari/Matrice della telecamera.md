https://www.youtube.com/watch?v=Hz8kz5aeQ44

# 📌 Appunti di Computer Vision: La Matrice della Telecamera (Camera Matrix)

La **Camera Matrix** è un concetto fondamentale della computer vision. Descrive matematicamente come un punto nello spazio tridimensionale reale (World Space) viene proiettato e mappato su un'immagine bidimensionale (piani dei pixel).

## 1. Il Modello Pinhole Camera (Foro Stenopeico)

Per analizzare le telecamere reali si usa una semplificazione teorica chiamata **Pinhole Camera**.

- **Definizione:** Una fotocamera ideale con un'apertura infinitesimale (descritta come un singolo punto). Una telecamera normale ha un'apertura (il diaframma) di qualche mm, nel modello ideale pinhole l'apertura viene pensata come un singolo punto geometrico nello spazio.
    
- **Vantaggio:** Ogni punto sul sensore d'immagine viene raggiunto dalla luce proveniente da un'unica direzione, evitando la sfocatura.
    
- **Telecamere Reali vs Pinhole:**
    
    - Le telecamere reali hanno aperture grandi e usano lenti per far convergere la luce (riducendo la sfocatura).
        
    - Le lenti introducono errori geometrici, come la **distorsione radiale** (correggibile in seguito) e la **distorsione tangenziale** (spesso trascurabile).
        

## 2. Sistemi di Coordinate e Convenzioni

La gestione dei vettori richiede la comprensione di due sistemi di coordinate distinti:

### Sistema di Coordinate del Mondo (World Coordinate System)

Dipende dall'applicazione. Una convenzione comune prevede:

- Assi **X** e **Y** che formano il piano del terreno.
    
- Asse **Z** rivolto verso l'alto.
    

### Sistema di Coordinate della Telecamera (Camera Coordinate System)

- Il sensore è allineato con gli assi **X** e **Y**.
    
- La telecamera punta lungo l'asse **Z positivo** (chiamato **Asse Ottico** o **Asse Principale**).
    
- _Nota:_ In computer graphics, a volte la telecamera guarda verso il canale Z negativo (cambiano solo i segni nelle formule).
    

> ⚠️ **Convenzione delle Unità di Misura nei seguenti appunti:**
> 
> - **Lettere MAIUSCOLE (X, Y, Z, F, W):** Variabili espresse in **millimetri (mm)**.
>     
> - **Lettere minuscole (u, v, w):** Variabili espresse in **pixel (px)**.
>     

## 3. La Matrice Intrinseca (Intrinsic Matrix)

La matrice intrinseca mappa le coordinate 3D della telecamera in coordinate pixel 2D.

### Geometria dei Triangoli Simili

Posizionando l'apertura della pinhole camera nell'origine $(0,0,0)$ del sistema della telecamera, il sensore si trova a una distanza $F$ (Lunghezza Focale in mm) dietro l'apertura.

Prendendo un punto nello spazio della telecamera con coordinate $(X, Z)$ e proiettandolo sul sensore alla coordinata $-u$, per la regola dei triangoli simili otteniamo:

$$\frac{-u}{-F} = \frac{X}{Z} \implies u = F \frac{X}{Z}$$

### Conversione in Pixel ($K$)

Poiché le immagini digitali lavorano in pixel, introduciamo un fattore di scala $K$:

$$K = \frac{w \text{ (larghezza in pixel)}}{W \text{ (larghezza in mm)}}$$

Moltiplicando la formula per $K$, otteniamo la lunghezza focale espressa in pixel ($f_x = F \cdot K$) e la coordinata finale in pixel.

### Offset del Sensore (Centro Principale)

Se il sensore non è perfettamente allineato con l'asse ottico ma è spostato di un offset $(u_0, v_0)$ espresso in pixel, la formula finale per le coordinate dell'immagine diventa:

$$u = f_x \frac{X}{Z} + u_0$$

$$v = f_y \frac{Y}{Z} + v_0$$

Riorganizzando le equazioni in forma matriciale lineare:

$$\begin{bmatrix} u \\ v \end{bmatrix} = \begin{bmatrix} f_x \frac{X}{Z} + u_0 \\ f_y \frac{Y}{Z} + v_0 \end{bmatrix}$$

## 4. Coordinate Omogenee (Homogeneous Coordinates)

Nello spazio Euclideo classico usiamo le coordinate cartesiane. Passare alle coordinate omogenee offre due grandi vantaggi in Computer Vision:

1. Permettono di rappresentare **punti all'infinito** (es. dove le linee parallele sembrano incontrarsi).
    
2. Consentono di combinare **Rotazione** e **Traslazione** in un'unica moltiplicazione matriciale.
    

|**Tipo di Coordinata**|**Spazio 2D**|**Spazio 3D**|
|---|---|---|
|**Cartesiana**|$(x, y)$|$(X, Y, Z)$|
|**Omogenea**|$(x, y, 1)$ oppure $(xs, ys, s)$|$(X, Y, Z, 1)$ oppure $(Xs, Ys, Zs, s)$|

- **Proprietà di Scala:** Moltiplicare una coordinata omogenea per uno scalare non nullo non cambia il punto geometrico. Per tornare alle cartesiane, basta dividere i primi elementi per l'ultimo elemento $s$.
    
- **Rappresentazione matematica delle trasformazioni:**
    
    $$\tilde{x} = \begin{bmatrix} R & t \\ 0 & 1 \end{bmatrix} x$$
    
    _(Dove $R$ è la matrice di rotazione e $t$ è il vettore di traslazione)._
    

## 5. La Matrice Estrinseca (Extrinsic Matrix)

Solitamente, gli oggetti nell'ambiente sono definiti nel _World Coordinate System_. Prima di applicare i parametri intrinseci della telecamera, dobbiamo convertire queste coordinate nel _Camera Coordinate System_.

- **Principio di Inversione:** Trasformare un sistema di coordinate in un modo equivale a trasformare i punti degli oggetti nel modo opposto (inverso).
    
- Se conosciamo la posizione e la rotazione della telecamera nel mondo, abbiamo la matrice **Camera-to-World**.
    
- A noi serve l'opposto: la matrice **World-to-Camera** (detta **Matrice Estrinseca**). Si ottiene semplicemente calcolando l'inversa della matrice Camera-to-World:
    

$$\text{Matrice Estrinseca} = \begin{bmatrix} R & | & t \end{bmatrix} = \text{Inversa di (Camera-to-World)}$$

Applicandola alle coordinate omogenee del mondo di un punto, otteniamo le sue coordinate rispetto alla telecamera.

## 6. La Camera Matrix Finale

La **Camera Matrix** complessiva si ottiene moltiplicando la Matrice Intrinseca ($K$) per la Matrice Estrinseca ($[R|t]$).

### Formula Finale

$$P = K \times [R | t]$$

Estesa in forma matriciale:

$$\begin{bmatrix} u \cdot s \\ v \cdot s \\ s \end{bmatrix} = \begin{bmatrix} f_x & 0 & u_0 \\ 0 & f_y & v_0 \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} r_{11} & r_{12} & r_{13} & t_x \\ r_{21} & r_{22} & r_{23} & t_y \\ r_{31} & r_{32} & r_{33} & t_z \end{bmatrix} \begin{bmatrix} X_w \\ Y_w \\ Z_w \\ 1 \end{bmatrix}$$

- **Input:** Coordinate 3D nel mondo $(X_w, Y_w, Z_w, 1)$.
    
- **Output:** Coordinate 2D pixel sull'immagine $(u, v)$ previa divisione per il fattore di scala omogeneo $s$.