
# 🌊 Dinamiche di LULC e di eutrofizzazione dei laghi di Amatitlan ed Atitlan
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

# 📌 Introduzione e obiettivo
Il Mesoamerica è una delle aree più critiche al mondo per studiare il Land Use/Land Cover Change (LULC), specialmente per le dinamiche di deforestazione e di espansione agricola nelle zone vicino ai laghi. In Guatemala, l'accelerazione della deforestazione, l'espansione agricola incontrollata e la pressione urbana stanno alterando radicalmente i bacini idrografici lacustri.

L'obiettivo di questo studio è analizzare il cambiamento di  land use change e di bloom algale in aree critiche del Guatemala che in questi ultimi anni hanno avuto drastici cambiamenti dovuti all'antropizzazione delle aree. Questo verificherà in poi se presente una relazione tra LULC e fenomeni di eutrofizzazione acquatica, tutto questo  in un orizzonte temporale di 10 anni ($2015 \rightarrow 2020 \rightarrow 2025$), con campionamenti a cadenza quinquennale.

# 🔬 Approccio Comparativo: Lago Profondo vs Lago Poco Profondo

La metodologia viene sviluppata in parallelo su due sistemi lacustri con caratteristiche morfometriche e risposte ecologiche opposte:

Lago di Atitlán: Un lago profondo di origine vulcanica (oligotrofico, ma in transizione verso condizioni mesotrofiche a causa di pressioni diffuse).

Lago di Amatitlán: Un lago poco profondo (iper-eutrofico, soggetto a un forte carico di nutrienti puntiformi e diffusi provenienti dall'area metropolitana di Città del Guatemala).

🛰️ 2. Dataset e Finestra Temporale

Il tipo di satellite usato è Landsat 8, acquisite tramite Google Earth Engine (GEE) (Modello Collection 2, Level 2 - Surface Reflectance).

[Landsat 8 su Google Earth Engine](https://code.earthengine.google.com/?scriptPath=Examples%3ADatasets%2FLANDSAT%2FLANDSAT_LC08_C02_T1&hl=it)

Le immagini sono state prese dall’ 1 agosto al 31 dicembre degli anni indicati

Motivazioen ecologica : coincide con i picchi storici di fioritura algale e cianobatterica (Rejmánková et al.), garantendo al contempo le condizioni di stabilità atmosferica necessarie per l'acquisizione di immagini satellitari ottiche prive di copertura nuvolosa. 

# N.B
"L'analisi metodologica viene sviluppata qui in Github in dettaglio sul Lago di Atitlán e successivamente applicata in ottica comparativa al Lago di Amatitlán, offrendo così un duplice scenario di risposta ecologica (lago profondo vs lago poco profondo) agli impatti del LULC e di fioritura algale."

# 💻 Analisi codice e sviluppo del progetto in R

caricamento librerie e dei dati
``` r
library(terra)
library(imageRy)
lago15 <- rast("C:/Users/erosm/Downloads/amatitlan_2015.tif")
lago20 <- rast("C:/Users/erosm/Downloads/amatitlan_2020.tif")
lago25 <- rast("C:/Users/erosm/Downloads/amatitlan_2025.tif")
```

verifichiamo un'immagine preliminare con le bande True colors (colori reali):

``` r
im.plotRGB(lago15, r=3, g=2, b=1)
```
![LAGO1](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/LAGO1.jpeg)

Analizziamo la distribuzione spettrale delle frequenze per verificare la consistenza radiometrica dei sensori tra il 2015 e il 2025:
``` r
hist(values(lago15[[1]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Red 2015", col="red")
hist(values(lago15[[2]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Green 2015", col="green")
hist(values(lago15[[3]]),freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Blue 2015", col="blue")

hist(values(lago25[[1]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Red 2025", col="red")
hist(values(lago25[[2]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Green 2025", col="green")
hist(values(lago25[[3]]), freq = FALSE, xlim = c(0, 30000),ylim = c(0, 0.00045), main="Istogramma Blue 2025", col="blue")

```
![ISTO](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/ISTO.jpeg)


Calcolo degli indici NDVI e DVI usati per vedere il **land-use change** (Banda 3 = Red, Banda 4 = NIR)
``` r
ndvi15 <- (lago15[[4]] - lago15[[3]]) / (lago15[[4]] + lago15[[3]])
ndvi20 <- (lago20[[4]] - lago20[[3]]) / (lago20[[4]] + lago20[[3]])
ndvi25 <- (lago25[[4]] - lago25[[3]]) / (lago25[[4]] + lago25[[3]])

dvi15 <- lago15[[4]] - lago15[[3]]
dvi20 <- lago20[[4]] - lago20[[3]]
dvi25 <- lago25[[4]] - lago25[[3]]
``` 
PLOT LAND USE CHANGE (NDVI)
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


Isolamento del Lago (mascheramento NDWI) e calcolo del Surface Algae bloom index (**SABI**)

ISOLARE IL LAGO CON L'NDWI (Normalized Difference Water Index)
L'NDWI sfrutta il Verde (riflesso dall'acqua) e il NIR (assorbito dall'acqua).
I valori > 0 indicano l'acqua pura. I valori < 0 indicano la terraferma.

``` r
ndwi15 <- (lago15[[2]] - lago15[[4]]) / (lago15[[2]] + lago15[[4]])
ndwi20 <- (lago20[[2]] - lago20[[4]]) / (lago20[[2]] + lago20[[4]])
ndwi25 <- (lago25[[2]] - lago25[[4]]) / (lago25[[2]] + lago25[[4]])
``` 
Creiamo le maschere: diciamo a R "Trova tutti i pixel dove l'NDWI è > 0 , cioè acqua pura"

``` r
maschera_acqua15 <- ndwi15 > 0
maschera_acqua20 <- ndwi20 > 0
maschera_acqua25 <- ndwi25 > 0
``` 

CALCOLARE IL SABI (Surface Algal Bloom Index)
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

![BLOOM ATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/BLOOM%20ATITLAN.jpeg)

![BLOOM AMATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/BLOOM%20AMATITLAN.jpeg)

# 📊 DISCUSSIONE DEI RISULTATI

Per l'NDVI: Evidenziare se attorno ai laghi la foresta (verde scuro) ha ceduto il passo a zone agricole/urbane (giallo/marrone).

Per il SABI: Evidenziare come l'aumento delle aree rosse/gialle nei laghi (specialmente nel 2025) coincida temporalmente con la perdita di vegetazione circostante osservata nell'NDVI, confermando l'apporto di nutrienti da dilavamento agricolo o scarichi urbani (Alawadi et al.)

# 🏔️ LAGO DI ATITLAN
![CONFRONTO ATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/CONFRONTO%20ATITLAN.jpeg)

In questi 10 anni i livelli di fioritura algale si sono espansi dalle sponde a tutto il lago. Analizzando le zone di sponda si può notare che le fioriture algali si sono espanse in modo concentrico negli anni, muovendosi verso il centro dello specchio d'acqua (zone naturalmente meno antropizzate). Analizzando i plot del LULC si denota subito un incremento di zone antropizzate con progressiva trasformazione di zone forestali seppur con basso indice NDVI a favore di zone di insediamento o altro utilizzo. Questo può causare un dilavamento costante di sedimenti intorno al lago che innesca processi di instabilità ecologica in un lago storicamente oligotrofico (caratteristica derivata dalla sua profondità)


# 🌾 LAGO DI AMATITLAN

![CONFRONTO AMATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/CONFRONTO%20AMATITLAN.jpeg)

Nel 2015 il lago mostrava una fioritura algale confinata e acuta nella porzione terminale est, probabilmente legata a uno scarico puntiforme. Nel 2025, in concomitanza con l'aumento dell'impatto antropico e agricolo visibile dall'NDVI su tutto il bacino , il fenomeno è cambiato: non abbiamo più l'evento isolato nella coda, ma l'intero specchio d'acqua ha subito un incremento sistematico e omogeneo dei valori di SABI. Il lago è diventato ecologicamente più instabile e diffusamente produttivo a causa del costante dilavamento di nutrienti dal suolo circostante.

📚 RIFERIMENTI BIBLIOGRAFICI

Rejmánková, E., et al. Spatiotemporal dynamics of cyanobacterial blooms in Lake Atitlán, Guatemala. (Ecological Applications).

SABI Index Definition: Alawadi, F. (2010). Detection of Algal Blooms in Shallow Water Using Landsat.
