
# LAGO DI AMATITLAN e ATITLAN, analisi eutrofizzazione e land use change intorno ai lago 2015/2020/2025
<table>
  <tr>
    <td>
      <img src="https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/Screenshot%202026-07-04%20194538.png" alt="Prima Immagine" width="100%"/>
    </td>
    <td>
      <img src="https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/Screenshot%202026-07-04%20194618.png" alt="Seconda Immagine" width="100%"/>
    </td>
  </tr>
</table>

# Introduzione
Il Mesoamerica è una delle aree più critiche al mondo per studiare il Land Use/Land Cover Change (LULC), specialmente per le dinamiche di deforestazione e di espansione agricola nelle zone vicino ai laghi. In Guatemala sono presenti laghi di natura e grandezze diverse che negli ultimi anni hanno subito un forte impatto antropico. la finalità dello studio è identificare se c’è stato un cambio di  land use change e se possa avere una relazione con le fioriture algali.

L' orizzonte temporale preso in considerazione è di 10 anni (dal 2015 al 2025 con step ogni 5 anni).

Il tipo di satellite usato è Landsat 8 (google earth engine) , impostato su immagini RAW modello 1 

[Landsat 8 su Google Earth Engine](https://code.earthengine.google.com/?scriptPath=Examples%3ADatasets%2FLANDSAT%2FLANDSAT_LC08_C02_T1&hl=it)

Le immagini sono state prese dall’ 1/8 al 31/12 
Tale finestra temporale coincide con i picchi storici di fioritura algale e cianobatterica (Rejmánková et al.), garantendo al contempo le condizioni di stabilità atmosferica necessarie per l'acquisizione di immagini satellitari ottiche prive di copertura nuvolosa. 

# N.B
"L'analisi metodologica viene sviluppata in dettaglio sul Lago di Atitlán e successivamente applicata in ottica comparativa al Lago di Amatitlán, offrendo così un duplice scenario di risposta ecologica (lago profondo vs lago poco profondo) agli impatti del LULC."

# Analisi codice e sviluppo del progetto
Per questo progetto sono stati usati i vari pacchetti:
``` r
library(terra)
library(imageRy)
``` 
caricamento dati mantenendo un range di 5 anni per permettere un'analisi più puntuale
``` r
lago15 <- rast("C:/Users/erosm/Downloads/amatitlan_2015.tif")
lago20 <- rast("C:/Users/erosm/Downloads/amatitlan_2020.tif")
lago25 <- rast("C:/Users/erosm/Downloads/amatitlan_2025.tif")
```

verifichiamo un'immagine con le bande colori reali:

``` r
im.plotRGB(lago15, r=3, g=2, b=1)
```
![LAGO1](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/LAGO1.jpeg)

visualizziamo le bande di colori con istogrammi per verificare la differenzaa tra 205 e 2025
``` r
hist(values(lago15[[1]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Red 2015", col="red")
hist(values(lago15[[2]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Green 2015", col="green")
hist(values(lago15[[3]]),freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Blue 2015", col="blue")

hist(values(lago25[[1]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Red 2025", col="red")
hist(values(lago25[[2]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Green 2025", col="green")
hist(values(lago25[[3]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Blue 2025", col="blue")

```
![ISTO](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/ISTO.jpeg)


Calcolo degli indici NDVI usati per vedere il landu-use change (Banda 3 = Red, Banda 4 = NIR)
``` r
ndvi15 <- (lago15[[4]] - lago15[[3]]) / (lago15[[4]] + lago15[[3]])
ndvi20 <- (lago20[[4]] - lago20[[3]]) / (lago20[[4]] + lago20[[3]])
ndvi25 <- (lago25[[4]] - lago25[[3]]) / (lago25[[4]] + lago25[[3]])
``` 
Calcolo il DVI per far risaltare il lago
``` r
dvi15 <- lago15[[4]] - lago15[[3]]
dvi20 <- lago20[[4]] - lago20[[3]]
dvi25 <- lago25[[4]] - lago25[[3]]
``` 

3. PLOT 1: LAND USE E DEFORESTAZIONE (NDVI)
``` r
par(mfrow = c(2, 2))
``` 
Ho creato una palette che mi sembrasse più idonea alla visualizzazione Palette colori: Marrone (suolo nudo/urbano) -> Giallo -> Verde scuro (Foresta)
``` r
cl_ndvi <- colorRampPalette(c("saddlebrown", "yellow", "forestgreen"))(100)
``` 
Plot per verificare il land-use change
``` r
plot(ndvi15, col = cl_ndvi, range =c(0,1), main = "Stato Vegetazione 2015 (NDVI)")
plot(ndvi20, col = cl_ndvi, range =c(0,1), main = "Stato Vegetazione 2020 (NDVI)")
plot(ndvi25, col = cl_ndvi, range =c(0,1), main = "Stato Vegetazione 2025 (NDVI)")
``` 

![STATO VEGETAZ ATITLAN DEF](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/STATO%20VEGETAZ%20ATITLAN%20DEF.jpeg)

![vegetazione amatitlan](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/vegetazione%20amatitlan.jpeg)


4. PLOT 2: EUTROFIZZAZIONE E ALGHE NEL LAGO (DVI)

ANALISI AVANZATA EUTROFIZZAZIONE: SABI E MASCHERAMENTO (2015 vs 2025)

STEP 1: ISOLARE IL LAGO CON L'NDWI (Normalized Difference Water Index)
L'NDWI sfrutta il Verde (riflesso dall'acqua) e il NIR (assorbito dall'acqua).
I valori > 0 indicano l'acqua pura. I valori < 0 indicano la terraferma.

``` r
ndwi15 <- (lago15[[2]] - lago15[[4]]) / (lago15[[2]] + lago15[[4]])
ndwi20 <- (lago20[[2]] - lago20[[4]]) / (lago20[[2]] + lago20[[4]])
ndwi25 <- (lago25[[2]] - lago25[[4]]) / (lago25[[2]] + lago25[[4]])
``` 
Creiamo le maschere: diciamo a R "Trova tutti i pixel dove l'NDWI è > 0"

``` r
maschera_acqua15 <- ndwi15 > 0
maschera_acqua20 <- ndwi20 > 0
maschera_acqua25 <- ndwi25 > 0
``` 

STEP 2: CALCOLARE IL SABI (Surface Algal Bloom Index)
Il SABI è sensibile alla clorofilla e ai cianobatteri in superficie.
Formula: (NIR - Red) / (Blue + Green)

``` r
sabi15 <- (lago15[[4]] - lago15[[3]]) / (lago15[[1]] + lago15[[2]])
sabi20 <- (lago20[[4]] - lago20[[3]]) / (lago20[[1]] + lago20[[2]])
sabi25 <- (lago25[[4]] - lago25[[3]]) / (lago25[[1]] + lago25[[2]])
``` 

STEP 3: APPLICARE LA MASCHERA (Il trucco per la scala dei colori)
"Tagliamo via" la terraferma dal calcolo delle alghe, mantenendo solo l'acqua
maskvalue=FALSE significa "Nascondi tutto quello che NON è acqua"

``` r
sabi15_solo_lago <- mask(sabi15, maschera_acqua15, maskvalue=FALSE)
sabi20_solo_lago <- mask(sabi20, maschera_acqua20, maskvalue=FALSE)
sabi25_solo_lago <- mask(sabi25, maschera_acqua25, maskvalue=FALSE)
``` 

Creiamo una palette: 
``` r
Blu profondo (acqua pulita) -> Azzurro -> Giallo/Arancio -> Rosso (fioritura severa)
cl_alghe <- colorRampPalette(c("darkblue", "deepskyblue", "yellow", "red"))(100)
```

Ora che la terraferma non c'è più, la scala cromatica si adatterà 
automaticamente in modo chirurgico solo alle dinamiche dell'acqua!

``` r
plot(sabi15_solo_lago, col = cl_alghe, range= c(-0.03,0.05), main = "Bloom Algali 2015 (SABI)")
plot(sabi20_solo_lago, col = cl_alghe, range= c(-0.03,0.05), main = "Bloom Algali 2020 (SABI)")
plot(sabi25_solo_lago, col = cl_alghe, range= c(-0.03,0.05), main = "Bloom Algali 2025 (SABI)")
```

![BLOOM ATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/BLOOM%ATITLAN.jpeg)

![BLOOM AMATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/BLOOM%AMATITLAN.jpeg)

# DISCUSSIONE E RISULTATI

Per l'NDVI: Evidenziare se attorno ai laghi la foresta (verde scuro) ha ceduto il passo a zone agricole/urbane (giallo/marrone).

Per il SABI: Evidenziare come l'aumento delle aree rosse/gialle nei laghi (specialmente nel 2025) coincida temporalmente con la perdita di vegetazione circostante osservata nell'NDVI, confermando l'apporto di nutrienti da dilavamento agricolo o scarichi urbani.

#LAGO DI ATITLAN
![CONFRONTO ATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/CONFRONTO%20ATITLAN.jpeg)

In questi 10 anni i livelli di fioritura algale si sono espansi dalle sponde a tutto il lago. Analizzando le zone limitrofe delle sponde si può notare un'atropizzazione sempre più densa. Questi dati si riflettono sicuramente sulle espansioni algali nel lago.


#LAGO DI AMATITLAN

![CONFRONTO AMATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/CONFRONTO%20AMATITLAN.jpeg)

Nel 2015 il lago mostrava una fioritura algale confinata e acuta nella porzione terminale est, probabilmente legata a uno scarico puntiforme. Nel 2025, in concomitanza con l'aumento dell'impatto antropico e agricolo visibile dall'NDVI su tutto il bacino , il fenomeno è cambiato: non abbiamo più l'evento isolato nella coda, ma l'intero specchio d'acqua ha subito un incremento sistematico e omogeneo dei valori di SABI. Il lago è diventato ecologicamente più instabile e diffusamente produttivo a causa del costante dilavamento di nutrienti dal suolo circostante.
