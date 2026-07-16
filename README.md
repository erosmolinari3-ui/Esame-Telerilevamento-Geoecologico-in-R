
# 🏞️ Dinamiche di LULC e di eutrofizzazione nei laghi di Amatitlan ed Atitlan
### Esame di Telerilevamento Geo-Ecologico in R - 2026
#### Molinari Eros

# 📑 Introduzione

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

 > aree di studio prese in considerazione. A sx lago di Atitlan, a dx lago di Amatitlan

Il Mesoamerica è una delle aree più critiche al mondo per studiare il Land Use/Land Cover Change (LULC), specialmente per le dinamiche di deforestazione e di espansione agricola. In Guatemala, l'accelerazione di questi fenomeni e la pressione urbana stanno alterando radicalmente i bacini idrografici lacustri.

# 🎯 Obiettivo
L'obiettivo di questo studio è analizzare il cambiamento di land use change e di bloom algale in zone lagunari, aree critiche del paese che in questi ultimi anni hanno subito varie trasformazioni dovute all'antropizzazione. Verrà verificato in seguito, se presente, una relazione tra LULC e fenomeni di eutrofizzazione acquatica, tutto questo  in un orizzonte temporale di 10 anni ($2015 \rightarrow 2020 \rightarrow 2025$), con campionamenti a cadenza quinquennale.

# 🔬 Approccio Comparativo: Lago Profondo vs Lago Poco Profondo

La metodologia viene sviluppata in parallelo su due sistemi lacustri con caratteristiche morfometriche e risposte ecologiche opposte:

Lago di Atitlán: Un lago profondo di origine vulcanica (oligotrofico, ma in transizione verso condizioni mesotrofiche a causa di pressioni diffuse tra cui turismo ed antropizzazione).

Lago di Amatitlán: Un lago pur sempre di origine vulcanica, ma poco profondo (iper-eutrofico, soggetto a un forte carico di nutrienti puntiformi e diffusi provenienti dall'area metropolitana di Città del Guatemala).

# 🛰️ Dataset e Finestra Temporale

Il tipo di satellite usato è Landsat 8 in Google Earth Engine (GEE) (Modello Collection 2, Level 2 - Surface Reflectance).

[Landsat 8 su Google Earth Engine](https://code.earthengine.google.com/?scriptPath=Examples%3ADatasets%2FLANDSAT%2FLANDSAT_LC08_C02_T1&hl=it)

>[!NOTE]
> Il codice JavaScript utilizzato è quello fornito durante il corso ed è disponibile nel file Codice.js

Le immagini sono state prese dall’ 1 agosto al 31 dicembre degli anni indicati

Motivazione ecologica : coincide con i picchi storici di fioritura algale e cianobatterica nelle aree di studio (Rejmánková et al.), garantendo al contempo le condizioni di stabilità atmosferica necessarie per l'acquisizione di immagini satellitari ottiche prive di copertura nuvolosa. 

# 💻 Analisi codice e sviluppo del progetto in R

>[!NOTE]
>Per motivi di chiarezza e di impatto grafico l'analisi metodologica viene sviluppata qui in Github nel dettaglio sul Lago di Atitlán e successivamente applicata in ottica comparativa al Lago di Amatitlán, offrendo così un duplice scenario di risposta ecologica (lago profondo vs lago poco profondo) agli impatti del LULC e di fioritura algale.

caricamento librerie e dei dati
``` r
library(terra)      # analisi delle immagini satellitari (raster)
library(imageRy)    # visualizzazione delle immagini satellitari  
library(viridis)    # editing delle palette di colori
library(RColorBrewer)  # editing delle palette di colori per scale di colori per daltonismo
library(ggplot2) #per creare grafici di confronto multivariabili
```

Importazione dei dati tramite `setwd()`:
``` r
setwd("~C:\\Users\\erosm\\Downloads\\")
getwd()
list.files()
```
Dati importati via `rast()`:
``` r
lago15 <- rast("C:/Users/erosm/Downloads/amatitlan_2015.tif")
lago20 <- rast("C:/Users/erosm/Downloads/amatitlan_2020.tif")
lago25 <- rast("C:/Users/erosm/Downloads/amatitlan_2025.tif")
```

verifichiamo un'immagine preliminare con le bande True colors (colori reali):

``` r
im.plotRGB(lago15, r=3, g=2, b=1)
```
![prepost](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/pre%20post.jpeg)

> lago di Atitlan con bande True Colors, a sx 2015 a dx 2025

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

> Istogrammi di consistenza radiometrica, confronto tra 2015 e 2025

Metto in plot le singole bande per verificare vegetazione sana (NIR) e vegetazione visibile (RBG)
``` r
im.multiframe(2,4) # Visualizzare un pannello grafico con 2 righe e 4 colonne
plot(lago15[[1]], col = magma(100), main = "Pre - Red") 
plot(lago15[[2]], col = magma(100), main = "Pre - Green")
plot(lago15[[3]], col = magma(100), main = "Pre - Blue")
plot(lago15[[4]], col = magma(100), main = "Pre - NIR")

plot(lago25[[1]], col = magma(100), main = "Post - Red")
plot(lago25[[2]], col = magma(100), main = "Post - Green")
plot(lago25[[3]], col = magma(100), main = "Post - Blue")
plot(lago25[[4]], col = magma(100), main = "Post - NIR")
``` 
![BANDE](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/plot%20singole.jpeg)
> Si notano cambiamenti principalmente intorno alle zone di cosa come le bande RGB vengano assorbite, determinando un graduale cambiamento a zone antropizzate o di suolo nudo

###Calcolo degli indici NDVI e DVI usati per vedere il **land-use change** (Banda 3 = Red, Banda 4 = NIR)
``` r
ndvi15 <- (lago15[[4]] - lago15[[3]]) / (lago15[[4]] + lago15[[3]])
ndvi20 <- (lago20[[4]] - lago20[[3]]) / (lago20[[4]] + lago20[[3]])
ndvi25 <- (lago25[[4]] - lago25[[3]]) / (lago25[[4]] + lago25[[3]])
diff_ndvi<-(ndvi15-ndvi25)

dvi15 <- lago15[[4]] - lago15[[3]]
dvi20 <- lago20[[4]] - lago20[[3]]
dvi25 <- lago25[[4]] - lago25[[3]]
``` 
PLOT LAND USE/LAND COVER CHANGE (NDVI)

``` r
plot(ndvi15, col=viridis(100), range =c(0,1), main = "Stato Vegetazione Atitlan 2015 (NDVI)")
plot(ndvi20, col=viridis(100), range =c(0,1), main = "Stato Vegetazione Atitlan 2020 (NDVI)")
plot(ndvi25, col=viridis(100), range =c(0,1), main = "Stato Vegetazione Atitlan 2025 (NDVI)")
plot(ndvi, col=viridis(100), range =c(0,1), main = "Differenza 2015-2025 Atitlan (NDVI)")

``` 

![STATO VEGETAZ ATITLAN DEF](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/atindvi.jpeg)

> Stato vegetazionale intorno al lago di Atitlan

![vegetazione amatitlan](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/amandvi.jpeg)

> Stato vegetazionale intorno al lago di Amatitlan

### Calcolo Surface Algae Bloom Index (SABI)

Surface Algae Bloom Index (SABI) serve a evidenziare e mappare le fioriture algali galleggianti (cianobatteri) e la vegetazione costiera. Sfrutta le bande vicine all'infrarosso e allo spettro del visibile per evidenziare le fioriture algali (Alawadi et al.). 

formula:

$$
SABI = \frac{NIR - RED}{GREEN + BLUE}
$$

Inizialmente ho isolato il Lago (mascheramento NDWI) e calcolato il Surface Algae bloom index (**SABI**)

L'NDWI (Normalized Difference Water Index) sfrutta il Verde (riflesso dall'acqua) e il NIR (assorbito dall'acqua).
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
>[!NOTE]
> Vedendo i risultati del decadimento spaziale dell'eutrofizzazione decido di porre i valori 0 come NA per non falsare le divisioni
``` r
# Trasforma tutti i valori 0 (FALSE, la terra) in NA
maschera_acqua25[maschera_acqua25 == 0] <- NA
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
diff_sabi<-(sabi15_solo_lago-sabi25_solo_lago)
``` 

Creiamo una palette idonea che permetta di verificare anche il più minimo cambiamento nei 10 anni: 

``` r
palette <- colorRampPalette(brewer.pal(11, "PuOr"))(100)
```

``` r
plot(sabi15_solo_lago, col = viridis(10), range = c(-0.06, 0.06), main = "Bloom Algali 2015 Atitlan (SABI)")
plot(sabi20_solo_lago, col = viridis(10), range = c(-0.06, 0.06), main = "Bloom Algali 2020 Atitlan (SABI)")
plot(sabi25_solo_lago, col = viridis(10), range = c(-0.06, 0.06), main = "Bloom Algali 2025 Atitlan (SABI)")
plot(diff_sabi, col =palette, range = c(-0.06, 0.06), main = "Differenza 2015-2025 Atitlan (SABI)")

```

![BLOOM ATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/blooma.jpeg)

![BLOOM AMATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/bloomatit.jpeg)


##Decadimento Spaziale dell'Eutrofizzazione
Per validare e quantificare la relazione ecologica tra l'uso del suolo circostante e lo stato trofico delle acque lacustri, abbiamo integrato l'analisi visiva con una modellizzazione della distanza spaziale dal fattore di disturbo (la linea di costa).
``` r
library(ggplot2)

# Estrazione del confine vettoriale del lago (shoreline) ed esclusione automatica della terraferma (NA)
confini_lago <- as.polygons(maschera_acqua25)
linea_costa  <- as.lines(confini_lago)

# Calcolo delle distanze continue (distanza di tutti i pixel della mappa dalla linea di costa)
dist_raster <- distance(sabi25_solo_lago, linea_costa)
dist_lago <- mask(dist_raster, maschera_acqua25)

# Estrazione dei valori per l'analisi statistica in R con ggplot2 eliminando i valori NA
df_spaziale <- data.frame(
  Distanza = as.numeric(values(dist_lago)),
  SABI_2015 = as.numeric(values(sabi15_solo_lago)),
  SABI_2025 = as.numeric(values(sabi25_solo_lago))
)
df_spaziale <- df_spaziale[complete.cases(df_spaziale), ]

# Campionamento statistico e reshaping in formato lungo per ggplot
df_sub <- df_spaziale[sample(1:nrow(df_spaziale), 5000), ]
df_long <- data.frame(
  Distanza = rep(df_sub$Distanza, 2),
  SABI = c(df_sub$SABI_2015, df_sub$SABI_2025),
  Anno = rep(c("2015", "2025"), each = nrow(df_sub))
)
  ```

## Plotting del modello di regressione locale LOESS (Locally Estimated Scatterplot Smoothing) per adattare un modello di decadimento non lineare

``` r
ggplot(df_long, aes(x = Distanza, y = SABI, color = Anno)) +
  geom_point(alpha = 0.15, size = 1) +
  geom_smooth(method = "loess", span = 0.5, size = 1.5, se = TRUE) +
  scale_color_manual(values = c("2015" = "deepskyblue3", "2025" = "firebrick2")) +
  labs(
    title = "Decadimento Spaziale dell'Eutrofizzazione (SABI)",
    subtitle = "Confronto multitemporale 2015 vs 2025",
    x = "Distanza dalla linea di costa (metri)",
    y = "Indice SABI"
  ) +
  theme_minimal()
  ```
![decadimento eutro atitlan](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/decadimento%20spaziale%20eutrofizzazione%20amatitlan.jpeg)

Lago di Atitlán: Il decadimento spaziale è perlopiù assente. La curva LOESS del 2025 mostra che, dopo il picco iniziale sulla costa, i valori di SABI rimangono costantemente superiori alla baseline storica del 2015 senza però, essere alti. Questo ci presenta la situazione di un lago con grande capacità tampone (diluizione volumetrica) che protegge il centro del lago per la sua profondità, che però, poco a poco, sta sviluppando una sensibile modificazione data da agenti esterni. 

![decadimento eutro amatitlan](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/dec%20amatitlan.jpeg)

Lago di Amatitlan: Il modello LOESS mostra un nitido decadimento spaziale inshore-offshore. Nel 2025 (curva rossa) si osserva un picco severo di eutrofizzazione confinato nei primi 100/200 metri dalla riva, che decade rapidamente e linearizza verso lo zero man mano che ci si sposta verso le acque aperte (pelagiche) rimanendo però sempre più alto dello 0. A 500 metri circa rimane costante il livello nel 2025 evidenziando una netta separazione tra ciò che avveniva nel 2015, derivato dalla capacità del corpo d'acqua di diffondere le sostanze in acque più prossimali. L'intero volume d'acqua, a causa della scarsa profondità e della sempre più forte pressione antropica, è saturo di nutrienti, determinando una fioritura sistemica diffusa.

# 📊 DISCUSSIONE DEI RISULTATI

### 🏔️ LAGO DI ATITLAN
![CONFRONTO ATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/CONFRONTO%20ATITLAN.jpeg)

In questi 10 anni i livelli di fioritura algale si sono espansi dalle sponde a tutto il lago. Analizzando le zone di sponda si può notare che le fioriture algali si sono espanse in modo concentrico negli anni, muovendosi verso il centro dello specchio d'acqua (zone naturalmente meno antropizzate). Analizzando i plot del LULC si denota subito un incremento di zone antropizzate con progressiva trasformazione di zone forestali seppur con basso indice NDVI a favore di zone di insediamento o altro utilizzo. Questo può causare un dilavamento costante di sedimenti intorno al lago che innesca processi di instabilità ecologica in un lago storicamente oligotrofico (caratteristica derivata dalla sua profondità)


### 🌾 LAGO DI AMATITLAN

![CONFRONTO AMATITLAN](https://cdn.jsdelivr.net/gh/erosmolinari3-ui/immagini-esame@main/CONFRONTO%20AMATITLAN.jpeg)

Nel 2015 il lago mostrava una fioritura algale confinata e acuta nella porzione terminale est, probabilmente legata a uno scarico puntiforme. Nel 2025, in concomitanza con l'aumento dell'impatto antropico e agricolo visibile dall'NDVI su tutto il bacino , il fenomeno è cambiato: non abbiamo più l'evento isolato nella coda, ma l'intero specchio d'acqua ha subito un incremento sistematico e omogeneo dei valori di SABI. Il lago è diventato ecologicamente più instabile e diffusamente produttivo a causa del costante dilavamento di nutrienti dal suolo circostante.

📚 RIFERIMENTI BIBLIOGRAFICI

Rejmánková, E., et al. Spatiotemporal dynamics of cyanobacterial blooms in Lake Atitlán, Guatemala. (Ecological Applications).

SABI Index Definition: Alawadi, F. (2010). Detection of Algal Blooms in Shallow Water Using Landsat.
