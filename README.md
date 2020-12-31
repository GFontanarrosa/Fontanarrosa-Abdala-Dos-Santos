"Codes for Morphospace analysis leads to an evo-devo model of digit patterning, Journal Of Experimental Zoology Part B."
---
author: "Fontanarrosa, Gabriela;Abdala, Virginia; and Dos Santos, Daniel Andrés"
date: "19/12/2020"
output:
  pdf_document: default
  html_document: default
---


Codes
1.Script for complete enumeration of the theoretical morphospace

create data for the 53 unique phalangeal formula (PF) surveyed across the lepidosaurian dataset


```{r}
PF <- data.frame(DI =  c(2, 2, 2, 2, 2, 2, 0, 3, 3, 2, 2, 0, 2, 2, 3, 3, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 2, 0, 2, 0, 0, 2, 2, 0, 0, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2, 2, 2, 2), DII = c(3, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 1, 3, 4, 3, 3, 0, 2, 0, 2, 1, 2, 1, 3, 3, 0, 3, 3, 3, 2, 2, 2, 2, 3, 0, 0, 3, 3, 0, 2, 2, 2, 0, 2, 2, 2, 0, 0, 0, 2, 2, 2, 3), DIII = c(4, 3, 3, 4, 4, 4, 3, 3, 3, 3, 3, 3, 4, 4, 4, 4, 2, 2, 1, 2, 1, 3, 1, 4, 4, 1, 4, 4, 4, 2, 3, 3, 2, 0, 1, 2, 4, 4, 3, 3, 4, 4, 3, 3, 3, 3, 0, 0, 2, 3, 3, 2, 3), DIV = c(5, 3, 3, 4, 5, 6, 3, 3, 3, 3, 4, 0, 5, 5, 4, 5, 2, 2, 2, 2, 1, 3, 1, 4, 5, 1, 5, 4, 5, 3, 3, 2, 2, 0, 3, 3, 4, 4, 4, 4, 5, 5, 3, 4, 5, 4, 3, 2, 4, 3, 3, 2, 5), DV = c(3, 2, 2, 3, 4, 3, 2, 2, 3, 3, 3, 0, 2, 4, 3, 3, 0, 2, 0, 0, 0, 2, 1, 3, 3, 0, 3, 2, 0, 3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2, 2, 3, 0, 3, 3, 0, 0, 0, 0, 0, 3, 2, 3))
aux <- as.matrix(PF)
nrofla <- nrow(aux)

dedos <- split(as.character(aux), 1:nrow(aux)) 
dedlab <- paste("D", c("I","II","III","IV","V"), sep = "")
detalle <- lapply(dedos, function(x) paste(dedlab, "_", x, sep =""))
nodos <- unique(unlist(detalle))
redmtx <- matrix(0, length(nodos), length(nodos))
rownames(redmtx) <- colnames(redmtx) <- nodos
for(i in detalle) {
    cuales <- match(i, nodos)
    redmtx[cuales, cuales] <- redmtx[cuales, cuales] + 1
}

```


2.Script for theoretical morphospace partition into eu-, meta-, quasi-, and u-manus 
The next bulk of sentences is oriented to enumerate the theoretical morphospace for
hand configurations. A vector of classification for the different manus is finally obtained.

```{r}
allmanus <- as.matrix(expand.grid(paste("DI_", 0:3, sep =""), paste("DII_", 0:4, sep =""), 
                                  paste("DIII_", 0:4, sep =""), paste("DIV_", 0:6, sep = ""), 
                                  paste("DV_", 0:4, sep ="")))
```


```{r}
howmany0 <- c()
for(i in 1:nrow(allmanus)) {
    pfs <- allmanus[i,]
    howmany0 <- c(howmany0, sum(redmtx[pfs, pfs] == 0))
}    
manusclasses <- rep("quasimanus", length(howmany0))
manusclasses[howmany0 == 20] <- "umanus"
manusclasses[howmany0 == 0] <- "metamanus"
observed <- duplicated(rbind(matrix(unlist(detalle), ncol = 5, byrow = TRUE), allmanus))
observed <- observed[-c(1:length(detalle))]
manusclasses[observed] <- "eumanus"

```
                                  
Create a single data.frame with PFs and classification of manus


```{r}
finaloutput <- cbind(expand.grid(c(0:3), c(0:4), c(0:4), c(0:6), c(0:4)), factor(manusclasses))
colnames(finaloutput) <- c(dedlab, "classification")
```


3.Script for Flow Network of Phylogenetic Tracked PFs (PhyNet) Construction.




```{r}
library(ape)
library(phytools)
library(castor)
library(phylobase)
library(igraph)
```


```{r}
fornexusfile <-
c("#NEXUS[Fontanarrosa et al. 2019, Limb evolution and development hidden in phalangeal formula codification]", 
"BEGIN TREES;", "TRANSLATE", "1 Ablepharus,", "2 Acanthosaura_,", 
"3 Acontias,", "4 Acratosaura_mentalis,", "5 Afroedura_transvaalica,", 
"6 Agama_,", "7 Agama_hispida,", "8 Aigialosaurus_bucchichi_(=Opetiosaurus_bucchichi),", 
"9 Aigialosaurus_dalmaticus_?,", "10 Aleuroscalabotes_felinus,", 
"11 Alexandresaurus_camacan,", "12 Alopoglossus_angulatus_,", 
"13 Alopoglossus_atriventris,", "14 Ameiva_ameiva,", "15 Amphibolorus,", 
"16 Amphiglossus_astrolabi,", "17 Amphiglossus_reticulatus,", 
"18 Amphiglossus_splendidus,", "19 Anadia_bogotensis,", "20 Anepischetosia_maccoyi_(Nannoscincus_maccoyi),", 
"21 Anisolepis_longicauda,", "22 Anolis_antonii,", "23 Anolis_crysolepis,", 
"24 Anolis_gundlachi,", "25 Anolis_maculiventris,", "26 Anolis_mariarum,", 
"27 Anolis_tolimensis,", "28 Anolis_trachyderma,", "29 Anolis_ventrimaculatus,", 
"30 Anomalopus_mackayi,", "31 Anomalopus_sp.,", "32 Anotosaura_collaris,", 
"33 Anotosaura_vanzolinia,", "34 Aristelliger_cochranae,", "35 Aristelliger_expextatus,", 
"36 Aristelliger_georgeensis,", "37 Aristelliger_lar,", "38 Aristelliger_praesignis,", 
"39 Arthrosaura_kockii,", "40 Arthrosaura_reticulata,", "41 Assacus_elisae,", 
"42 Assacus_sp.,", "43 Ateuchosaurus_chinensis,", "44 Ateuchosaurus_pellopleurus,", 
"45 'Bachia_barbouri*',", "46 Bachia_bicolor,", "47 Bachia_blairi,", 
"48 Bachia_bresslaui,", "49 Bachia_dorbignyi,", "50 Bachia_flavescens,", 
"51 Bachia_flavescens_parkeri,", "52 Bachia_heteropa_trinitatis,", 
"53 Bachia_intermedia,", "54 Bachia_monodactylus_monodactylus,", 
"55 'Bachia_pyburni_(B._panoplia)_*1',", "56 'Bachia_pyburni_*2',", 
"57 Bachia_scolecoides,", "58 Bachia_sp._,", "59 Bachia_trisanale,", 
"60 'Bachyseps_anosyensis_(=Amphiglossus_anosyensis)_',", "61 Basciliscus_sp.,", 
"62 Bassiana,", "63 Bipes_biporus,", "64 'Bipes_canaliculatus*1',", 
"65 'Bipes_canaliculatus*2',", "66 Bipes_tridactylus,", "67 Brachymeles_bicolor,", 
"68 Brachymeles_boholensis,", "69 Brachymeles_boulengeri,", "70 Brachymeles_cebuensis,", 
"71 Brachymeles_elerae,", "72 Brachymeles_gracilis,", "73 Brachymeles_libayani_,", 
"74 Brachymeles_mutingkamay,", "75 Brachymeles_paeforum,", "76 Brachymeles_pathfinderi_,", 
"77 Brachymeles_schadenbergi,", "78 Brachymeles_talinis,", "79 Brachymeles_tridactylus,", 
"80 'Brachyseps_frontoparietalis_(=Amphiglossus_praeornatus)',", 
"81 'Brachyseps_gastrosticus_(=Amphiglossus_gastrosticus)',", 
"82 'Brachyseps_macrocercus_(=Amphiglossus_macrocercus)',", "83 'Brachyseps_mandady_(=Amphiglossus_mandady)',", 
"84 'Brachyseps_punctatus_(=Amphiglossus_punctatus)',", "85 'Brachyseps__spilostichus_(Amphiglossus_spilostichus)',", 
"86 Bradypodion_damaranum,", "87 Bradypodion_occidentale,", "88 'Brookesia_sp.*1',", 
"89 'Brookesia_sp.*2',", "90 Bunopus,", "91 'Caledoniscincus_austrocaledonicus_(=Leiolopisma_austrocaledonicum)',", 
"92 Calotes_,", "93 Calotes_versicolor,", "94 Calyptotis_lepidorostrum_,", 
"95 Calyptotis_ruficauda,", "96 Calyptotis_scutirostrum,", "97 Calyptotis_temporalis,", 
"98 Calyptotis_thorntonensis,", "99 Caparaoinia_itaiquara,", 
"100 'Carinascincus_sp._(=Niveoscincus)',", "101 Carlia_schmeltzii,", 
"102 'Carlia_sp.*',", "103 Carphodactylus_laevis,", "104 Carsosaurus_marchesetti_?,", 
"105 'Cercosaura_argula_(=Prionodactylus_argulus)',", "106 Cercosaura_eigenmanni_,", 
"107 Cercosaura_ocellata,", "108 Cercosaura_parkeri,", "109 Cercosaura_schreibersi,", 
"110 Chalarodon_,", "111 'Chalcides_(some_species)*1',", "112 'Chalcides_(some_species)*2',", 
"113 Chalcides_armitagei,", "114 Chalcides_bedriagai,", "115 'Chalcides_boulengeri_(=Sphenops_boulengeri)',", 
"116 Chalcides_chalcides_mertensis,", "117 Chalcides_chalcides_vittatus,", 
"118 Chalcides_colosii_,", "119 Chalcides_ghiari,", "120 Chalcides_lanzai,", 
"121 Chalcides_manueli,", "122 'Chalcides_mauritanicus*1',", 
"123 'Chalcides_mauritanicus*2',", "124 'Chalcides_mionecton_(some)*1',", 
"125 'Chalcides_mionecton_(some)_*2',", "126 'Chalcides_mionecton_(some_populations)_',", 
"127 Chalcides_montanus_,", "128 Chalcides_ocellatus,", "129 Chalcides_polylepis,", 
"130 Chalcides_pulchellus,", "131 Chalcides_ragazzii_,", "132 Chalcides_sexlineatus,", 
"133 'Chalcides_sphenopsiformis_(=Sphenops_sphenopsiformis)',", 
"134 Chalcides_striatus,", "135 Chalcides_thierryi,", "136 'Chalcides_viridanus_(Chalcidea_viridianus@Chalcides_simonyi)',", 
"137 'Chamaesaura_sp.*1',", "138 'Chamaesaura_sp.*2',", "139 'Chamaesaura_sp.*3',", 
"140 Chamalaeo_calyptratus,", "141 Chamalaeo_dilepis,", "142 Chamalaeo_namaquensis,", 
"143 'Chondrodactylus_angulifer*1',", "144 'Chondrodactylus_angulifer*2',", 
"145 'Chondrodactylus_bibronii_(Pachydactylus_bibronii)',", "146 'Christinus_marmoratus_(Phyllodactylus_marmoratus)',", 
"147 Clevosaurus_sp._?,", "148 Cnemaspis_africanus,", "149 Cnemaspis_annularis,", 
"150 Cnemaspis_chanthaburiensis_,", "151 Cnemaspis_kandiana,", 
"152 Cnemaspis_occidentalis,", "153 Cnemaspis_petrodroma,", "154 Cnemaspis_podihuna,", 
"155 'Cnemaspis_spinicolis*1',", "156 'Cnemaspis_spinicolis*2',", 
"157 Cnemaspis_tropidogaster,", "158 Cnemidophorus_cf._abalosi,", 
"159 Cnemidophorus_lacertoides,", "160 Cnemidophorus_lemniscatus,", 
"161 Cnemidophorus_longicaudus,", "162 Coeranoscincus_reticulatus,", 
"163 Coggeria_naufragus,", "164 Coleodactylus_septentrionalis,", 
"165 Coleonix_brevis,", "166 Coleonix_mitratus,", "167 Coleonyx_elegans,", 
"168 Coleonyx_fasciatus,", "169 'Coleonyx_reticulatus*1',", "170 'Coleonyx_reticulatus*2',", 
"171 Coleonyx_switaki,", "172 Coleonyx_variegatus,", "173 Colobodactylus_dalcyanus,", 
"174 'Colobodactylus_taunayi*1',", "175 'Colobodactylus_taunayi*2',", 
"176 Colobosaura_modesta,", "177 Colobosaura_sp.,", "178 Colobosauroides_cearensis,", 
"179 'Colopus_kochii_(Pachydactylus_kochii)',", "180 Colopus_walhbergii,", 
"181 Corucia,", "182 Cosymbotus,", "183 Crenadactylus_ocellatus,", 
"184 'Cryptoblepharus_boutonii_cognatus_(_Cryptoblepharus_cognatus)_',", 
"185 'Cryptoblepharus_voeltzkowi_(=Cryptoblepharus_boutonii_voeltzkowi)',", 
"186 'Ctenophorus_adelaidensis_(Rankinia_adelaidensis)',", "187 'Ctenophorus_chapmani_(Rankinia_chapmani)',", 
"188 Ctenophorus_clayi,", "189 Ctenophorus_femoralis,", "190 Ctenophorus_sp.,", 
"191 Ctenotus,", "192 Cyclodomorphus,", "193 Cyrtodactylus_agusanensis,", 
"194 Cyrtodactylus_caspius,", "195 Cyrtodactylus_condorensis,", 
"196 Cyrtodactylus_fedtschenkoi,", "197 Cyrtodactylus_intermedius,", 
"198 Cyrtodactylus_loriae,", "199 Cyrtodactylus_malayanus,", 
"200 Cyrtodactylus_marmoratus,", "201 Cyrtodactylus_montiumsalsorum,", 
"202 Cyrtodactylus_papuensis,", "203 Cyrtodactylus_pulchellus,", 
"204 Cyrtopodium_scabrum,", "205 'Dasia_(Apterygodon_)',", "206 Diplodactylus_stenodactylus,", 
"207 Diplodactylus_vittatus,", "208 Dixonius_siamensis,", "209 Draco,", 
"210 Dryadosaura_nordestina,", "211 Ebenavia_inunguis,", "212 Echinosaura_horrida,", 
"213 Egernia,", "214 Eichstaettisaurus_schroederi,", "215 Emoia_adspersa,", 
"216 Emoia_atrocostata,", "217 Emoia_concolor,", "218 Emoia_longicauda,", 
"219 Emoia_nigra,", "220 Emoia_pallidiceps,", "221 'Emoia_reimschiisseli_(=Emoia_caeruleocauda)',", 
"222 'Eremiascincus_antoniorum_(Glaphyromorphus_antoniorum)',", 
"223 'Eremiascincus_douglasi_(=Glaphyromorphus_douglasi)',", 
"224 'Eremiascincus_emigrans_(=Glaphyromorphus_emigrans)',", 
"225 'Eremiascincus_pardalis_(=Glaphyromorphus_pardalis)',", 
"226 'Eremiascincus_timorensis_(=Glaphyromorphus_timorensis)_',", 
"227 Eublepharis_macularius,", "228 Eugongylus_sp.,", "229 Eulamprus__sp.,", 
"230 Eumeces_sp.,", "231 Eumecia_sp.,", "232 Eurydactylodes_vieillardi,", 
"233 Eutropis_longicaudata,", "234 'Flexiseps_alluaudi_(Amphiglossus_alluaudi)',", 
"235 'Flexiseps_andranovahensis_(=Amphiglossus_andranovahensis)',", 
"236 'Flexiseps_ardouini__(Amphiglossus_ardouini)',", "237 'Flexiseps_crenni_(Amphiglossus_crenni)*1',", 
"238 'Flexiseps_crenni_(Amphiglossus_crenni)*2',", "239 'Flexiseps_decaryi_(=Amphiglossus_decaryi)',", 
"240 'Flexiseps_elongatus_(Amphiglossus_elongatus)',", "241 'Flexiseps_johannae_(Amphiglossus_johannae)',", 
"242 'Flexiseps_melanurus_(=Amphiglossus_melanurus)',", "243 'Flexiseps_ornaticeps_(=Amphiglossus_ornaticeps)',", 
"244 'Flexiseps_tanysoma_(Amphiglosus_tanysoma)',", "245 Gallotia_atlantica,", 
"246 Gallotia_caesaris,", "247 Gallotia_galloti,", "248 Gallotia_simonyi,", 
"249 Gallotia_stehlini,", "250 Gehyra_australis,", "251 Gehyra_baiola,", 
"252 Gehyra_oceanica,", "253 Gehyra_variegata,", "254 'Gekko_gecko_(Gecko_verticillatus)',", 
"255 Gekko_hokounensis,", "256 Gekko_porosus,", "257 Gekko_vittatus,", 
"258 Gerrhonotus,", "259 Gerrhosaurus,", "260 Geymodactylus_geckoides,", 
"261 Glaphyromorphus_cracens,", "262 Glaphyromorphus_crassicaudus,", 
"263 Glaphyromorphus_darwiniensis_,", "264 Glaphyromorphus_fuscicaudis,", 
"265 Glaphyromorphus_mjobergi,", "266 Glaphyromorphus_nigricaudis,", 
"267 Glaphyromorphus_pumilus,", "268 Glaphyromorphus_punctulatus,", 
"269 Gonatodes_albogularis,", "270 Gonatodes_albogularis_fuscus,", 
"271 Gonatodes_concinatus,", "272 Gonatodes_humeralis,", "273 Gonocephalus,", 
"274 Graciliscincus_sp.,", "275 Gymnophthalmus_sp.,", "276 Gymnophthalmus_underwoodi,", 
"277 'Hakaria_simonyi_(Hakaria_sokotrana)*1',", "278 'Hakaria_simonyi_(Hakaria_sokotrana)*2',", 
"279 'Harrinosoniascincus_zia_(Cautula_zia)',", "280 Helodema_horridum,", 
"281 Heloderma_suspectum,", "282 Hemidactylus_brasilianus,", 
"283 Hemidactylus_flaviviridis,", "284 Hemidactylus_frenatus,", 
"285 Hemidactylus_mabouia,", "286 Hemiergis_decresciensis,", 
"287 'Hemiergis_gracilipes_(Sphenomorphus_gracilipes=Glaphyromorphus_gracilipes)',", 
"288 Hemiergis_initialis,", "289 Hemiergis_miliwae,", "290 'Hemiergis_peronii*1',", 
"291 'Hemiergis_peronii*2',", "292 Hemiergis_quadrilineata,", 
"293 Hemiphyllodactylus,", "294 Hemitheconix,", "295 Heterodactylus_imbricatus,", 
"296 'Heterodactylus_lundi*1',", "297 'Heterodactylus_lundi*2',", 
"298 Heteronotia_binoei,", "299 Holodactylus_africanus,", "300 Homonota_darwini,", 
"301 Homonota_fasciata,", "302 Homonota_uruguayanensis,", "303 Homopholis_walbergii,", 
"304 Hoplodactylus_duvauceli,", "305 Hoplodactylus_maculatus,", 
"306 Hoplodactylus_pacificus,", "307 Hydrosaurus,", "308 Iguana_iguana,", 
"309 'Iphisa_elegans*1',", "310 'Iphisa_elegans*2',", "311 Janetaescincus_veseyfitzgeraldi,", 
"312 Japalura_variegata,", "313 Kentropix_viridistriga,", "314 Lacerta,", 
"315 Lacertaspis_rohdei,", "316 Lacertoides_sp._,", "317 Lamprolepis_sp.,", 
"318 Lampropholis_elongata,", "319 Lampropholis_sp.,", "320 Lankascincus_sp.,", 
"321 Lanthanothus,", "322 Lanthanothus_borneensis,", "323 Leiolepis_,", 
"324 Leiosaurus_paronae,", "325 Lepidoblepharis_heyerorum,", 
"326 Lepidodactylus_christiani,", "327 'Lepidodactylus_lugubris_(Lepidodactylus_woodfordi)',", 
"328 Lepiodactylus_christiani,", "329 Leptosiaphos_blochmanni,", 
"330 Leptosiaphos_graueri,", "331 'Leptosiaphos_hackarsi_(=Panaspis_hackarsi)',", 
"332 Leptosiaphos_luberoensis,", "333 Leptosiaphos_vigintiserium,", 
"334 Lerista_aericeps_,", "335 Lerista_arenicola,", "336 Lerista_borealis,", 
"337 'Lerista_bouganvilii*1',", "338 'Lerista_bouganvilii*2',", 
"339 'Lerista_christinae*1',", "340 'Lerista_christinae*2',", 
"341 'Lerista_desertorum*_1',", "342 'Lerista_desertorum*_2',", 
"343 'Lerista_distinguenda*1',", "344 'Lerista_distinguenda*2',", 
"345 'Lerista_dorsalis*1',", "346 'Lerista_dorsalis*2',", "347 'Lerista_dorsalis*3',", 
"348 Lerista_elegans,", "349 Lerista_fragilis,", "350 'Lerista_gerrardii*1',", 
"351 'Lerista_gerrardii*2',", "352 Lerista_haroldi,", "353 'Lerista_kalumburu*1',", 
"354 'Lerista_kalumburu*2',", "355 Lerista_lineata,", "356 Lerista_macropisthopus,", 
"357 Lerista_microtis,", "358 'Lerista_muelleri*(some)1',", "359 'Lerista_muelleri*(some)2',", 
"360 Lerista_neander,", "361 Lerista_orientalis,", "362 Lerista_planiventralis,", 
"363 Lerista_punctatovittata,", "364 Lerista_separanda,", "365 Lerista_stictopleura,", 
"366 Lerista_storri,", "367 Lerista_taeniata,", "368 Lerista_talpina,", 
"369 Lerista_terdigitata,", "370 Lerista_vittata,", "371 Lerista_walkeri,", 
"372 'Lerista_wilkinsi_(=L._wilkinsoni)',", "373 Lerista_xanthura,", 
"374 Lerista_zonulata,", "375 Liolaemus_abdalai,", "376 Liolaemus_albiceps,", 
"377 Liolaemus_argentinus,", "378 Liolaemus_atacamensis,", "379 Liolaemus_austromendocinus,", 
"380 Liolaemus_azarai,", "381 Liolaemus_bibronii,", "382 Liolaemus_bitaeniatus,", 
"383 Liolaemus_capillitas,", "384 Liolaemus_chacoensis,", "385 Liolaemus_chaltin,", 
"386 Liolaemus_chiloensis,", "387 Liolaemus_crepuscularis,", 
"388 Liolaemus_cuyanus,", "389 Liolaemus_dorbigny,", "390 Liolaemus_groseorum,", 
"391 Liolaemus_incayali,", "392 Liolaemus_irregularis,", "393 Liolaemus_kingii,", 
"394 Liolaemus_kolengh,", "395 Liolaemus_koslowskyi,", "396 Liolaemus_kriegi,", 
"397 Liolaemus_lavillai,", "398 Liolaemus_magellanicus,", "399 Liolaemus_multicolor,", 
"400 Liolaemus_neuquensis,", "401 Liolaemus_nigromaculatus,", 
"402 Liolaemus_nigroviridis,", "403 Liolaemus_orientalis,", "404 Liolaemus_ornatus,", 
"405 Liolaemus_pagaburoi,", "406 Liolaemus_pictus,", "407 Liolaemus_platei,", 
"408 Lioscincus_sp.,", "409 'Lipinia_inconspicua_(=Scincella_inconspicua_=Sphenomorphus_inconspicuum)',", 
"410 Lipinia_sp.,", "411 'Loxopholis_guianense_(=Leposoma_guianensis)',", 
"412 'Loxopholis_hoogmoedi__(=Arthrosaura_hoogmoedi)',", "413 'Loxopholis_osvaldoi_(=Leposoma_osvaldoi)',", 
"414 'Loxopholis_percarinatum__(=Leposoma_percarinatum)_',", 
"415 'Loxopholis_rugiceps_(=Leposoma_rugiceps)',", "416 Lucasium_alboguttatus,", 
"417 'Lucasium_dameum_(Lucasius_damaeus)',", "418 Lucasium_maini,", 
"419 Lucasium_squarrosus,", "420 Lucasium_stenodactylus,", "421 'Lygisaurus_sp.*1',", 
"422 'Lygisaurus_sp.*2',", "423 'Lygisaurus_sp.*3',", "424 Lygodactylus_capensis,", 
"425 Lygosoma_bowringii_,", "426 Lygosoma_punctata,", "427 Lygosoma_quadrupes,", 
"428 Lyriocephalus,", "429 Mabuya_mabouya,", "430 'Madascincus_ankodabensis_(Amphiglossus_ankodabensis)',", 
"431 'Madascincus_igneocaudatus__(=Amphiglossus_igneocaudatus)',", 
"432 'Madascincus_melanopleura_(Amphiglossus_melanopleurus)',", 
"433 'Madascincus_mourondavae_(Amphiglossus_mouroundavae)',", 
"434 'Madascincus_nanus_(Amphiglossus_nanus)',", "435 'Madascincus_stumpffi_(Amphiglossus_stumpffi)',", 
"436 Marmorosphax_tricolor,", "437 Menetia_sp.,", "438 'Menetia_sp._(Menitia_)',", 
"439 Micrablepharus_atticolus,", "440 Micrablepharus_maximiliani,", 
"441 'Mochlus_brevicaudis_(=Lygosoma_brevicaudis)',", "442 Mochlus_sundevalli,", 
"443 'Moloch_sp._*1',", "444 'Moloch__sp._*_2',", "445 Moloch_horridus,", 
"446 Morethia_sp.,", "447 Nactus_pelagicus,", "448 Nannoscincus_gracilis,", 
"449 Nannoscincus_greeri,", "450 Nannoscincus_hanchisteus,", 
"451 Nannoscincus_humectus,", "452 Nannoscincus_slevini,", "453 'Narudasia_festiva*1',", 
"454 'Narudasia_festiva*2',", "455 Naultinussp.,", "456 Neoseps_sp.,", 
"457 Nephrururs_deleani,", "458 Nephrurus_asper,", "459 Nephrurus_laevissimus,", 
"460 Nephrurus_levis,", "461 Nephrurus_stellatus,", "462 Nephrurus_vertebralis,", 
"463 Nephrurus_wheeleri,", "464 Neusticurus_bicarinatus_,", "465 Neusticurus_rudis,", 
"466 Notoscincus_sp.,", "467 Oedura_leseuri,", "468 Oedura_marmorata,", 
"469 'Oligosoma_(Clyclodina)',", "470 Ophiomorus_blanfordi,", 
"471 Ophiomorus_brevipes,", "472 Ophiomorus_chernovi,", "473 Ophiomorus_nuchalis,", 
"474 Ophiomorus_persicus,", "475 Ophiomorus_raithmai,", "476 'Ophiomorus_streeti*1',", 
"477 'Ophiomorus_streeti*2',", "478 Ophiomorus_trydactylus,", 
"479 Otocryptis,", "480 Pachydactylus_austeni,", "481 Pachydactylus_mariquensis,", 
"482 Pachydactylus_punctatus,", "483 'Pachydactylus_rangei_(Palmatogecko_rangei)',", 
"484 'Pachydactylus_vanzyli_(Kaokogecko_vanzyli)',", "485 Pamelaescincus_gardineri,", 
"486 Panaspis_breviceps,", "487 Panaspis_togoensis,", "488 'Panaspis_wahlbergi_(=Afroablepharus_wahlbergi)',", 
"489 Papuascincus_sp.,", "490 'Parvoscincus_lawtoni_(Sphenomorphus_lawtoni)',", 
"491 Parvoscincus_sp.,", "492 Phelsuma_dubia,", "493 Phoboscincus_sp.,", 
"494 Pholidobolus_montium,", "495 'Pholidobolus_vertebralis_(=Prionodactylus_vertebralis)',", 
"496 Phrynocephalus,", "497 Phrynosoma_asio,", "498 Phrynosoma_cornutum,", 
"499 Phrynosoma_douglassi,", "500 Phrynosoma_modestum,", "501 Phyllodactylus_angelensis,", 
"502 Phyllodactylus_bauri,", "503 Phyllodactylus_delcampi,", 
"504 Phyllodactylus_gerrhopyjus,", "505 Phyllodactylus_homolepidurus,", 
"506 Phyllodactylus_inaequalis,", "507 Phyllodactylus_reissi,", 
"508 Phyllodactylus_unctus,", "509 Phyllodactylus_ventralis,", 
"510 Phyllodactylus_wirshingi,", "511 Phyllodactylus_xanti,", 
"512 Phyllopezus_pollicaris,", "513 Phyllurus,", "514 Phymaturus_adrianae,", 
"515 Phymaturus_aguanegra,", "516 Phymaturus_casposo,", "517 Phymaturuscf._maulense,", 
"518 Phymaturus_dorsimaculatus,", "519 Phymaturus_extridilus,", 
"520 Phymaturus_gualcamayo,", "521 Phymaturus_indistinctus,", 
"522 Phymaturus_larioja_ree_504,", "523 Phymaturus_laurenti,", 
"524 Phymaturus_mallimacci,", "525 Phymaturus_nevadoi,", "526 Phymaturus_palluma,", 
"527 Phymaturus_patagonicus,", "528 Phymaturus_punae,", "529 Phymaturus_querque,", 
"530 Phymaturus_spurcus,", "531 Phymaturus_tenebrosus,", "532 Phymaturus_zapalensis,", 
"533 Physignathus_concicicinus,", "534 Placosoma_cordylinum,", 
"535 Placosoma_glabellum,", "536 Plestiodon_reynoldsi,", "537 Polychrus_acutirostris,", 
"538 Potamites_ecpleopus,", "539 Potamites_juruazensis,", "540 Prasinohaema_sp.,", 
"541 Proablepharus_sp.,", "542 Procellosaurinus_tetradactylus,", 
"543 Proctoporus_bolivianus,", "544 Proctoporus_striatus,", "545 Proctoporus_xestus,", 
"546 Proscelotes_aenea,", "547 'Proscelotes_arnoldi_(some_especimens)*1',", 
"548 'Proscelotes_arnoldi_(some_especimens)*2',", "549 'Proscelotes_arnoldi_(some_especimens)*3',", 
"550 Proscelotes_eggeli,", "551 Psammophilus,", "552 'Pseudemonia_(Claireascincus)_',", 
"553 Pseudogonatodes_guinanensis,", "554 Pseudogonatodes_sp,", 
"555 Psilophthalmus_paeminosus,", "556 Ptenopus_carpi,", "557 Ptenopus_garrulus,", 
"558 Ptenopus_kochi,", "559 Ptychoglossus_brevifrontalis,", "560 Ptychoglosus_bicolor,", 
"561 Ptychozoon_kuhli,", "562 Ptyodactylus_hasselquisti,", "563 'Pygomeles_trivittatus_(=Androngo_trivittatus=Amphiglossus_trivittatus)',", 
"564 Quedenfeldtia_trachyblepharus,", "565 'Rankinia_diemensis_*1',", 
"566 'Rankinia_diemensis_*2',", "567 Rhachisaurus_brachylepis,", 
"568 Rhinogecko_missonei,", "569 Rhoptropella_ocellala,", "570 Rhoptropus_afer,", 
"571 Rhynchoedura_ornata,", "572 Ristella_sp.,", "573 'Saiphos_equalis*1',", 
"574 'Saiphos_equalis*2',", "575 Saproscincus_sp.,", "576 Saproscincus_tetradactyla,", 
"577 Sceloporus_sp,", "578 'Scelotes_(algunas_especies)',", "579 Scincopus_sp.,", 
"580 Scincus_scincus,", "581 Sepsina_tetradactyla,", "582 Shinisaurus_crocodilurus,", 
"583 Sigaloseps_sp.,", "584 Simiscincus_sp.,", "585 Sitana,", 
"586 Sitana_ponticeriana,", "587 'Sphaerodacrylus_copei_(S._picturatus)',", 
"588 Sphaerodactylus_klauberi,", "589 Sphaerodactylus_roosvelti,", 
"590 Sphenodon_punctatus,", "591 Sphenomorphus_aignanus,", "592 Sphenomorphus_alfredi,", 
"593 Sphenomorphus_anotus,", "594 Sphenomorphus_bignelli,", "595 Sphenomorphus_brunneus,", 
"596 Sphenomorphus_cinereus,", "597 Sphenomorphus_cranei,", "598 Sphenomorphus_derooyae,", 
"599 Sphenomorphus_fasciatus_,", "600 Sphenomorphus_forbesi,", 
"601 Sphenomorphus_fragilis,", "602 Sphenomorphus_fragosus,", 
"603 Sphenomorphus_judei,", "604 Sphenomorphus_leptofasciatus,", 
"605 Sphenomorphus_longicaudatus,", "606 Sphenomorphus_maindroni,", 
"607 Sphenomorphus_minutus,", "608 'Sphenomorphus_muelleri_(Ictiscincus_muelleri@_Sphenomorphus_mulleri)',", 
"609 'Sphenomorphus_nigriventris*1',", "610 'Sphenomorphus_nigriventris*2',", 
"611 Sphenomorphus_nigrolineatum,", "612 'Sphenomorphus_pratti*_(algunos)',", 
"613 Sphenomorphus_rarus,", "614 'Sphenomorphus_schlegeli_(Sphenomorphus_oxycephalus)',", 
"615 Sphenomorphus_schultzei,", "616 Sphenomorphus_scutatus,", 
"617 'Sphenomorphus_solomonis*_(some)2',", "618 'Sphenomorphus_solomonis*1',", 
"619 Sphenomorphus_tanneri,", "620 Sphenomorphus_taylori,", "621 Stenocercus_doellojuradoi,", 
"622 Stenocercus_trachycephalus,", "623 'Stenodactylus_sthenodactylus*1',", 
"624 'Stenodactylus_sthenodactylus*2',", "625 Stenolepis_ridleyi,", 
"626 'Strophurus_michaelseni(Diplodactylus_michaelseni)',", "627 Takydromus_sexlineatum,", 
"628 Tarentola_americana,", "629 'Tarentola_annnularis*1',", 
"630 'Tarentola_annularis*2',", "631 Tarentola_boehmei,", "632 Tarentola_delalandii,", 
"633 Tarentola_ephippiata,", "634 Tarentola_mauritanica,", "635 MCZ_Re190835_?,", 
"636 Teius_sp.,", "637 Tetrapodophis_amplectus_?,", "638 Thecadactylus_rapicauda,", 
"639 Tiliqua_rugosa,", "640 Tiliqua_sp.,", "641 'Trachylepis_aureopunctata_(Mabuya_aureopunctata)',", 
"642 'Trachylepis_boettgeri_(Mabuya_boettgeri)',", "643 'Trachylepis_comorensis_(Mabuya_comorensis)',", 
"644 'Trachylepis_elegans_(Mabuya_elegans)',", "645 'Trachylepis_gravenhorstii_(Mabuya_gravenhorstii)',", 
"646 'Trachylepis_madagascariensis_(Mabuya_madagascariensis)',", 
"647 'Trachylepis_margaritifera_(Trachylepis_margaritifer)*',", 
"648 Tretioscincus,", "649 Tretioscincus_agilis,", "650 Tretiscincus_bifascatus,", 
"651 Tribolonotus_sp.,", "652 Tropicolotes_tripolitanus,", "653 Tropidophorus_sp.,", 
"654 Tropidurus_melanopleurus,", "655 Tropidurus_spinolosus,", 
"656 Tropidurus_torquatus,", "657 'Tropidurus_torquatus_(UNNEC_6815)',", 
"658 Tropiocolotes_steudneri,", "659 Tupinampis_sp.,", "660 Tympanocryptis_lineata,", 
"661 'Tympanocryptis_tetraporophora*1',", "662 'Tympanocryptis_tetraporophora*2',", 
"663 'Tytthoscincus_atrigularis_(Sphenomorphus_atrigularis)',", 
"664 'Tytthoscincus_butleri_(Sphenomorphus_butleri)_',", "665 'Tytthoscincus_textus_(Sphenomorphus_textum)',", 
"666 'Underwoodisaurus_milii(Phyllurus_milii)',", "667 Uromastix_sp.,", 
"668 Uroplatus_fimbriatus,", "669 Vanzosaura_rubricauda,", "670 Varanus_cf._niloticus,", 
"671 Xenosaurus_sp.,", "672 Zonosaurus_;", paste("\tTREE metasq = ((147,590),((((((232,(183,((455,((304,305),306)),(((468,467),626[%color = 5 ]),((571,((419,420),((417,418),416))),(206,207))))))DIPLODACTYLIDAEE,(103,(513,(((((460,461),(462,(459,457))),463),458),666)))CARPHODACTYLIDAEE),(((10,((165,172),((166,167),((169,170),(168,171))))),(299,(227,294)))EUBLEPHARIDAE,(((564,((((37,38):0.0,34):0.0,35):0.0,36)),((325,((272,271),(269,270))),(164,((554,553),(587,(588,589))))))SPHAERODACTYLIDAE,((((((327,326),(254,(255,(256,(561,257))))),(((208,298),447),(293,((252,251),(250,253))))),((((((152,153):0.0,156):0.0,155):0.0,150):0.0,149)[%color = 5 ],(((658[%color = 5 ],652),(624,623)),(((90,204),568[%color = 5 ]),(((((((((((194,195):0.0,196):0.0,197):0.0,198):0.0,201):0.0,193):0.0,202):0.0,200):0.0,203):0.0,199),(182,((283,284),(282,285)))))))),((211,((((558[%color = 5 ],556):0.0,557[%color = 5 ]),(453,454)),(148,668))),(((146,(303,5[%color = 5 ])),(570,(((143,144),145),((180,179),(482,(481,(480,(483,484)))))))),(((157,151),154),((569,424),492)))))GEKKONIDAE,((638,((41,42),562)),((((302,300):0.0,301),((((((((((501,502):0.0,503):0.0,504):0.0,505):0.0,511):0.0,506):0.0,507):0.0,508):0.0,509):0.0,510)),((512,(260,328)),(628,((631,634),(((630,629),633),632))))))PHYLLODACTYLIDAE))))GEKKOTA,214):0.0,635),((((259,672)GERRHOSAURIDAE,((139,137):0.0,138)CORDYLIDAE),", 
"(3,((((((((((470,471):0.0,472):0.0,473):0.0,474):0.0,475):0.0,477):0.0,476):0.0,478),((((70,(73,75)),(79,((71,74),(68,((72,76),((67,77),(78,69))))))),(456[%color = 5 ],536)),((230,(579,580)),(((311,485),((((122,123),(134,(116,117))),(120,(118,114))),((128,115),(((132,136),(133,(((125,126):0.0,124),(129,127)))),(((((((130[%color = 5 ],131[%color = 5 ]):0.0,111[%color = 5 ]):0.0,112[%color = 5 ]):0.0,135[%color = 5 ]):0.0,113[%color = 5 ]):0.0,119[%color = 5 ]):0.0,121[%color = 5 ]))))),(581,((278,277)[%triangled = on ],((((546,550):0.0,((549,547):0.0,548)),578),(((((((84,80),(60,82)),((85[%color = 5 ],81[%color = 5 ]):0.0,83[%color = 5 ])):0.0,((16,17),563)),18[%color = 5 ]),((((((((((244,241):0.0,236):0.0,238):0.0,237[%color = 5 ]):0.0,234):0.0,240):0.0,242):0.0,243):0.0,235):0.0,239)),(((((432,431),433),434),435),430[%color = 5 ])))))))))SCINCINAE,(((43,44),(1,(653,(410,(608,((((663,665):0.0,664),((((((((((((((((((((((((616,(618,617)),(599,(604,(597,606)))):0.0,591):0.0,592):0.0,593):0.0,594):0.0,595):0.0,596):0.0,598):0.0,600):0.0,601):0.0,602):0.0,603):0.0,605):0.0,607):0.0,(610,609)):0.0,611):0.0,612):0.0,613):0.0,614):0.0,615):0.0,619):0.0,620),(491,490))),(((489,540):0.0,409[%color = 5 ]),(((229[%color = 5 ],((((94,95):0.0,96):0.0,97):0.0,98)):0.0,(163,((574,573),162))),((31,30),(((((((225,223):0.0,222):0.0,226):0.0,224),((288,((290,291),292)),(287,(289,286)))),(268,(((265,(266,264)),(261,267)),(262,263)))),(466,(191,(((372,370[%color = 5 ]):0.0,366[%color = 5 ]),(((354,353),(((371,336),349),(((373,(334,367)),(374,361)),(360,((342,341),((351,350),356)))))),(357,(335,((369,(((345,347):0.0,346),363)),((((348,(344,343)),((339,340),(355,362))),364[%color = 5 ]),((365,(352,(359,358))),((338,337),368[%color = 5 ])))))))))))))))))))),(((651,(181,(213,(192,(639,640))))),((572,320[%color = 5 ]),(233,(205,((647,(((645,(646,644)),(641,642)),643)),(231,429)))))),(((217,215),(317,(427,(442,(426,(441,425)))))),(228,((((486,487),488),(315,((((331,330):0.0,333):0.0,332):0.0,329))),(((62,552),(469,(((452,448),(449,(451,450))),(436,((((408,316):0.0,583):0.0,493):0.0,(274,(584,91))))))),(((184,185),((437,438),(221,(216,((218,219):0.0,220))))),(446,((575,576),((319,318),", 
"(((541,100[%color = 5 ]),279),(((((20[%color = 5 ],421):0.0,422):0.0,423):0.0,101):0.0,102))))))))))))LYGOSOMINAE)))SCINCOIDEA,(((((14,(158,(161,(159,160)))),((313,636)[%triangled = on ],659))TEIIDAE,((((559,560),(12,13)),((((40,39),((((412[%color = 5 ],415):0.0,414):0.0,411):0.0,413)),((178,((32,33),210)),(((464,465),(534,535)),(212,(19,((494,495),((((106,107),(108,(105,109))),(538,539)),((543,545):0.0,544)))))))),((((((((((50,51),52),57),54),(56,55)),(59,49)),53),((48,46):0.0,45)),(58,47)[%color = 5 ]),((567,(99,((173,(174,175)),(295,(297,296))))),((11,((310,309),((177[%color = 5 ],176),(625,4)))),(((650,648):0.0,649),((439,440),(669,542)))))))),((275,276),555))GYMNOPHTHALMIDAE),((66,(63,(64,65)))BIPEDIDAE,(627,((245,(246,247)),(248,(249,314))))LACERTIDAE))LACERTOIDEA,((((671,((280,281)HELODERMATIDAE,258)),(582,((321,322)LANTHANOTIDAE,(((9,8),104),670))))ANGUIMORPHA,((((667,(323,((307,(533,(((445,443):0.0,444),(((186,188),((190,187):0.0,189)),(15,((565,566),((661,662),660))))))),((496,(7,6)),((312,209),((273,428),(((2,((585,586),479)),(92,93)),551[%color = 5 ])))))))AGAMIDAE,((88[%color = 5 ],89[%color = 5 ])[%color = 5 ],((87,86),((141,140),142)))CHAMAELEONIDAEE),((((654,655),(657,656)),(621,622))TROPIDURIDAE,(308,(((((499,500),497),498),577)PHRYNOSOMATIDAE,(21,537)POLYCHROTIDAE)))),(((324,110),(((((((((((((((((((((((((((((((((((((((((((((((((((514,515):0.0,516):0.0,518):0.0,519):0.0,520):0.0,521):0.0,522):0.0,523):0.0,524):0.0,525):0.0,526):0.0,527):0.0,528):0.0,529):0.0,530):0.0,531):0.0,532):0.0,517):0.0,375):0.0,405):0.0,406):0.0,407):0.0,376):0.0,377):0.0,378):0.0,379):0.0,380):0.0,381):0.0,382):0.0,383):0.0,384):0.0,385):0.0,386):0.0,387):0.0,388):0.0,389):0.0,390):0.0,391):0.0,392):0.0,393):0.0,394):0.0,395):0.0,396):0.0,397):0.0,398):0.0,399):0.0,400):0.0,401):0.0,402):0.0,403):0.0,404)LIOLAEMIDAE),(61,(((27,28),29),((22,23),(24,(25,26))))DACTYLOIDAE)))IGUANIA),637)TOXICOFERA))))[% ] [% ] [%  setBetweenBits = triangled setBetweenLong = color ];"
, sep="", collapse = ""), 
"END;")


writeLines(fornexusfile, "Lepitree")
squamata <- ape::read.nexus("Lepitree")

d1 <-
c(2, 2, 0, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 0, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 2, 2, 2, 
3, 3, 3, 0, 2, 2, 2, 0, 2, 2, 0, 0, 0, 2, 2, 2, 0, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 0, 2, 0, 2, 2, 0, 0, 2, 2, 2, 2, 0, 0, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 0, 0, 2, 2, 0, 2, 2, 2, 2, 2, 3, 3, 3, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 0, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 1, 0, 1, 2, 0, 2, 3, 3, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
0, 2, 2, 2, 2, 2, 0, 0, 2, 0, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 0, 1, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 2, 2, 0, 0, 0, 3, 
2, 1, 1, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 0, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 0, 0, 2, 0, 2, 
0, 2, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 
2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 0, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 0, 0, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2, 3, 3, 3, 
3, 3, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 2, 2, 2, 2, 0, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
0, 2, 2, 3, 2, 0, 0, 2, 2, 0, 2, 2, 2, 2, 0, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 3, 2, 
3, 3, 3, 3, 3, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 1, 
2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 2, 
2)
d2 <-
c(3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 0, 2, 2, 0, 2, 1, 2, 2, 2, 1, 2, 3, 2, 3, 0, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 2, 2, 3, 2, 2, 2, 3, 3, 3, 2, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 2, 3, 3, 2, 2, 3, 3, 3, 3, 0, 0, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 0, 2, 3, 3, 1, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 4, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 2, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
0, 3, 3, 3, 3, 3, 0, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 0, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 3, 
0, 3, 3, 2, 2, 0, 0, 2, 2, 2, 2, 2, 2, 2, 0, 0, 2, 0, 0, 0, 0, 
3, 2, 2, 0, 2, 0, 0, 2, 0, 0, 2, 2, 2, 0, 0, 2, 2, 2, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, 2, 3, 3, 3, 0, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 2, 2, 2, 2, 2, 2, 2, 2, 2, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 0, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
2, 2, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3)
d3 <-
c(4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 3, 3, 3, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 3, 3, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 2, 2, 2, 1, 2, 1, 2, 3, 2, 1, 2, 4, 2, 4, 1, 3, 4, 4, 
3, 3, 3, 3, 3, 3, 3, 2, 2, 3, 2, 2, 2, 3, 3, 3, 2, 3, 3, 3, 3, 
3, 3, 4, 3, 3, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 0, 3, 4, 4, 3, 2, 4, 4, 4, 4, 1, 2, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 2, 3, 4, 4, 3, 4, 4, 4, 4, 4, 3, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 3, 3, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
2, 4, 4, 3, 3, 3, 2, 2, 3, 4, 3, 3, 3, 3, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 3, 3, 4, 4, 4, 4, 3, 3, 3, 4, 4, 4, 4, 4, 4, 3, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 3, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 3, 4, 
3, 4, 4, 4, 4, 2, 3, 3, 4, 3, 3, 3, 4, 3, 0, 2, 3, 0, 0, 3, 2, 
4, 3, 3, 2, 3, 2, 0, 3, 0, 0, 3, 3, 3, 3, 2, 3, 3, 3, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 3, 4, 4, 3, 3, 3, 3, 3, 3, 4, 4, 4, 4, 4, 
4, 4, 4, 3, 3, 4, 4, 3, 4, 4, 4, 3, 4, 4, 4, 0, 4, 3, 3, 3, 3, 
4, 3, 4, 4, 4, 4, 4, 4, 3, 3, 3, 3, 3, 3, 3, 3, 3, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 0, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
3, 3, 3, 3, 2, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 2, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 3, 4, 4, 4, 4, 4, 4, 4, 2, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 3, 4, 4, 4, 4, 4, 3, 
4, 4, 4, 4, 4, 4, 4, 3, 4, 3, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4, 4, 4, 4, 4, 4, 3, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 
4)
d4 <-
c(5, 5, 5, 5, 5, 5, 4, 5, 5, 5, 5, 5, 5, 5, 5, 4, 4, 4, 5, 4, 
5, 5, 5, 5, 5, 5, 5, 5, 5, 3, 2, 5, 4, 5, 5, 5, 5, 5, 5, 5, 4, 
4, 4, 4, 2, 2, 2, 2, 2, 1, 2, 3, 2, 1, 2, 4, 2, 5, 1, 4, 5, 5, 
3, 3, 3, 3, 3, 3, 3, 2, 2, 3, 2, 2, 2, 3, 3, 3, 2, 4, 4, 4, 4, 
4, 4, 4, 4, 3, 4, 5, 5, 5, 5, 4, 4, 4, 4, 4, 5, 5, 5, 4, 5, 5, 
5, 5, 5, 5, 5, 5, 4, 0, 3, 4, 4, 3, 2, 4, 4, 4, 4, 3, 3, 4, 4, 
4, 4, 4, 4, 4, 4, 4, 3, 3, 4, 4, 0, 5, 5, 5, 4, 4, 3, 4, 5, 5, 
5, 5, 5, 5, 5, 4, 4, 5, 4, 5, 5, 5, 5, 5, 5, 3, 3, 4, 4, 5, 5, 
5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 4, 5, 5, 5, 5, 5, 5, 5, 5, 
5, 4, 5, 4, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 
4, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 4, 5, 5, 5, 5, 5, 5, 5, 5, 
3, 5, 5, 3, 4, 3, 2, 2, 4, 5, 3, 4, 4, 3, 5, 5, 5, 5, 5, 5, 5, 
5, 5, 5, 5, 5, 5, 5, 4, 4, 5, 5, 5, 5, 5, 5, 4, 4, 5, 4, 5, 5, 
5, 5, 5, 5, 3, 4, 5, 5, 5, 5, 4, 4, 4, 4, 5, 4, 4, 5, 5, 4, 5, 
5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 4, 5, 5, 5, 
5, 5, 5, 5, 4, 5, 4, 4, 5, 5, 4, 5, 5, 5, 5, 5, 5, 5, 5, 4, 5, 
4, 5, 5, 5, 5, 3, 3, 4, 5, 4, 4, 5, 5, 4, 3, 3, 4, 2, 3, 4, 3, 
5, 3, 4, 3, 4, 3, 2, 4, 3, 3, 4, 4, 4, 4, 4, 4, 4, 4, 5, 5, 5, 
5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 
5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 4, 5, 4, 4, 
4, 5, 4, 5, 5, 5, 5, 3, 5, 5, 4, 4, 4, 4, 3, 4, 5, 4, 5, 5, 5, 
4, 5, 4, 3, 3, 5, 5, 3, 4, 4, 4, 3, 5, 5, 5, 2, 4, 3, 3, 3, 3, 
4, 3, 5, 5, 5, 5, 5, 5, 4, 4, 4, 4, 4, 3, 4, 4, 4, 5, 5, 5, 5, 
5, 5, 4, 5, 5, 4, 5, 4, 5, 5, 5, 5, 5, 5, 4, 4, 4, 4, 5, 5, 5, 
5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 
5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 2, 5, 5, 5, 5, 5, 5, 5, 5, 5, 
3, 3, 3, 3, 2, 5, 5, 4, 5, 5, 5, 5, 5, 5, 5, 5, 5, 3, 5, 4, 4, 
5, 5, 5, 5, 4, 5, 3, 4, 5, 5, 5, 4, 4, 5, 2, 5, 5, 5, 5, 4, 5, 
5, 5, 5, 5, 4, 4, 5, 5, 5, 5, 5, 5, 4, 5, 5, 4, 5, 5, 5, 5, 4, 
4, 5, 5, 4, 4, 4, 4, 4, 4, 5, 5, 5, 5, 4, 4, 4, 5, 5, 5, 5, 4, 
5, 5, 5, 5, 5, 5, 5, 3, 5, 3, 4, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 
5, 4, 5, 5, 5, 5, 5, 5, 5, 5, 5, 6, 5, 4, 5, 5, 5, 5, 5, 5, 5, 
4)
d5 <-
c(3, 3, 0, 3, 3, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 0, 0, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 0, 2, 2, 0, 0, 0, 0, 2, 0, 1, 2, 3, 2, 3, 0, 3, 3, 3, 
3, 2, 3, 2, 2, 2, 2, 0, 0, 2, 0, 0, 0, 2, 2, 2, 0, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 0, 0, 3, 3, 0, 0, 3, 3, 3, 3, 0, 0, 2, 3, 
0, 3, 3, 3, 3, 3, 3, 0, 0, 3, 3, 0, 2, 3, 3, 3, 3, 2, 3, 3, 3, 
3, 3, 3, 4, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 0, 0, 3, 3, 3, 3, 
3, 2, 3, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
0, 3, 3, 2, 3, 3, 0, 0, 3, 0, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 0, 3, 3, 3, 0, 3, 0, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 2, 3, 3, 3, 3, 3, 0, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 0, 3, 3, 3, 3, 2, 3, 
0, 2, 3, 2, 3, 0, 0, 3, 3, 2, 3, 3, 3, 0, 0, 0, 0, 0, 0, 0, 0, 
3, 0, 0, 0, 2, 0, 0, 2, 0, 0, 2, 0, 0, 0, 0, 2, 2, 2, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 2, 2, 3, 3, 2, 3, 3, 3, 0, 3, 4, 3, 0, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 2, 3, 2, 2, 2, 2, 0, 0, 0, 2, 0, 4, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 2, 2, 2, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 0, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 2, 2, 3, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 0, 3, 3, 3, 3, 3, 3, 4, 2, 3, 3, 3, 4, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
3)

aux <- castor::asr_max_parsimony(squamata, d1 + 1,  transition_costs="proportional") 
d1r <- apply(aux$ancestral_likelihoods, 1, which.max) - 1

aux <- castor::asr_max_parsimony(squamata, d2 + 1,  transition_costs="proportional") 
d2r <- apply(aux$ancestral_likelihoods, 1, which.max) - 1

aux <- castor::asr_max_parsimony(squamata, d3 + 1,  transition_costs="proportional") 
d3r <- apply(aux$ancestral_likelihoods, 1, which.max) - 1

aux <- castor::asr_max_parsimony(squamata, d4 + 1,  transition_costs="proportional") 
d4r <- apply(aux$ancestral_likelihoods, 1, which.max) - 1

aux <- castor::asr_max_parsimony(squamata, d5 + 1,  transition_costs="proportional") 
d5r <- apply(aux$ancestral_likelihoods, 1, which.max) - 1

flatip <- apply(cbind(d1, d2, d3, d4, d5), 1, paste, collapse = "")
flanode <- apply(cbind(d1r, d2r, d3r, d4r, d5r), 1, paste, collapse = "")

vectorflas <- c(flatip, flanode)
nodosred <- unique(vectorflas)

squamata$edge.length[] <- 0
sq4 <- as(squamata, "phylo4") 
df <- as(sq4, "data.frame")
desdeancdesc <- df[,3:2] 


```

New formulations appearing in internal nodes
after reconstruction of ancestral hand configurations

```{r}
setdiff(nodosred, flatip)
ancestro <- unlist(ancestors(sq4, c(1:nrow(df)), "parent"))
anc_comun <- mrca(squamata)
```


Construct the flow of evolutionary change.
Nodes correspond to phalangeal formulae

```{r}
redflujo <- matrix(0, length(nodosred), length(nodosred))
for(i in 1:nrow(df)){
 link <- match(vectorflas[unlist(desdeancdesc[i,])], nodosred)
 #link es un vector de 2 elementos, el origen y el destino
 redflujo[link[1], link[2]] <- redflujo[link[1], link[2]] + 1
} 
rownames(redflujo) <- colnames(redflujo) <- nodosred
redfinal <- graph_from_adjacency_matrix(redflujo, mode = "directed", 
                                        weighted = TRUE, diag = FALSE)
tkplot(redfinal)##Choose layout Reingold-Tilford
plot(redfinal)#alternative plot
```


To know the root vertex for the Reingold-Tilford layout


```{r}
match(vectorflas[673], nodosred)
```


End of lines code for drawing the Flow Network of Evolutionary Change.
4.Script for Phalangeal Formula Drawing. 
The next graphical function draws a hand. 
The required input is a vector describing the phalangeal formula.


```{r}
dibujomano <- function(flafal, color = "white") {
  pos <- 0
  angs <- c(90 + 75, 90 + 75/2, 90, 90 - 75/2, 90 - 75)/180*pi
  plot(c(0, cos(angs)*(1.2*flafal + 1.2)), c(0, sin(angs)*(1.2*flafal + 1.2)), type = "n", 
       bty = "n", axes = F, xlab = "", ylab = "", asp = 1) 
  rotaciones <- c(75, 75/2, 0, -75/2, -75)*pi/180
  resol <- 100
  x <- cos(pi/2 + seq(-2, 2, length.out = resol)*pi/18)
  y <- sin(pi/2 + seq(-2, 2, length.out = resol)*pi/18) 
  plotrot <- function(coordx, coordy, posdigit){
      alpha <- rotaciones[posdigit]
      dedox <- coordx*cos(alpha) - coordy*sin(alpha)
      dedoy <- coordx*sin(alpha) + coordy*cos(alpha)
      return(cbind(dedox, dedoy))
  }
  for(i in flafal) {
     pos <- pos + 1
     nx <- c(x, rev(x))
     ny <- c(y, rev(y) + (i > 0))
     if(i == 0) dedos <- plotrot(nx, ny, pos)
     else {
      aux <- 1.2*as.vector(rbind(0:(i - 1), NA))
      dedos <- plotrot(rep(c(nx, NA), i), rep(c(ny, NA), i) + rep(aux, rep(c(2*resol, 1), i)), pos)
     }
     polygon(dedos, col = color)
  }
   abline(b = tan(15*pi/180), a = 0, lty = "dotted", col = "blue")
   abline(b = tan((75/2 + 15)*pi/180), a = 0, lty = "dotted", col = "blue")
   abline(b = tan(15*pi/180), a = 0, lty = "dotted", col = "blue")
   abline(b = tan(-1*(75/2 + 15)*pi/180), a = 0, lty = "dotted", col = "blue")
   abline(b = tan(-1*15*pi/180), a = 0, lty = "dotted", col = "blue")
   abline(v = 0, lty = "dotted", col = "blue")
}

```



#Running the example concerning to the plesiomorphic hand configuration



```{r}
dibujomano(c(2,3,4,5,3))
```

