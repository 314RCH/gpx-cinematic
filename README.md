# GPX Cinematic

Application web (un seul fichier, sans build) qui transforme une trace GPX en
animation vidéo 16:9 sur fond satellite, avec export MP4 directement depuis le
navigateur.

## Lancer

```bash
cd /Users/pierre/Projets/gpx2mp4 && python3 -m http.server 5178
```

puis ouvrir <http://localhost:5178/index.html>.

Après une mise à jour de `index.html`, recharger la page sans cache
(Cmd+Maj+R) : le petit serveur Python ne l'interdit pas au navigateur.

Un mini-serveur est nécessaire : ouvrir `index.html` en double-clic peut être
bloqué par le navigateur pour les requêtes réseau (tuiles de carte).

Testé sur Chrome / Edge / Brave et Safari 17+ (l'export MP4 utilise WebCodecs).

## Utilisation

1. Déposer un fichier `.gpx` (ou cliquer sur la zone de dépôt).
2. Régler titre, couleur, caméra et rythme dans le panneau de gauche.
3. `Espace` (ou ▶) pour prévisualiser, la réglette pour se déplacer dans le temps.
4. **Exporter en MP4** : rendu image par image en 1920×1080.

## Déroulé de l'animation

| Phase | Ce qui se passe |
|---|---|
| Intro | Vue globale de la trace, nord en haut, puis zoom vers le point de départ |
| Parcours | Caméra inclinée qui suit le point courant et vise en avant, trace persistante |
| Final | Dézoom vers la vue globale nord en haut + carte de fin : titre, **sous-titre de fin** (champ libre, propre à chaque trace, rien s'il est vide), puis distance, D+, durée |

## Plusieurs traces

On peut charger plusieurs GPX à la fois : sélection multiple, **📁 Ouvrir un
dossier**, ou glisser-déposer d'un dossier entier (sous-dossiers compris).
Les traces apparaissent dans une liste ; un clic rend une trace **active**.

- La trace active est celle que la caméra suit et qui est **exportée**.
- Chaque trace garde ses propres points de passage (départ, passages,
  arrivée, couleurs de tronçons), son titre, son sous-titre et sa durée.
- Les autres traces sont dessinées **en gris** quand elles sont dans le champ,
  avec leurs départs et arrivées (repères gris). Une étiquette grise qui
  tomberait sur un repère de la trace active est masquée.
- Une trace déjà chargée (même contenu) n'est pas ajoutée deux fois.

## Onglets

- **🎬 Vidéo** : l'aperçu 16:9 tel qu'il sera exporté, avec la lecture.
- **🗺 Carte & photos** : la carte réelle, interactive (molette, glisser),
  avec la trace active, les points de passage (déplaçables) et les photos ;
  à droite, un bandeau vertical des vignettes. « 🖈 Placer sur la carte »
  ouvre cet onglet en mode ajout de points.

## Photos et vidéos

**Les médias sont rangés par trace.** Sélectionnez une trace, puis
**📁 Dossier de la trace active** (section Photos & vidéos) : son contenu est
rattaché à cette trace uniquement. La liste des traces indique ce que chacune
contient (📷 photos, 🎬 vidéos) ; l'onglet Carte & photos et la vidéo
n'utilisent que les médias de la trace active. Une organisation simple : un
sous-dossier par jour ou par GPX.

- **Photos** : jpg, png, webp (heic seulement sous Safari). Position et date
  lues dans l'EXIF, y compris le bloc `eXIf` des PNG (exports d'iPhone).
- **Vidéos** : mp4, mov, m4v, webm (HEVC d'iPhone compris sous Chrome/Safari
  macOS). Position et date lues dans les métadonnées QuickTime
  (`com.apple.quicktime.location.ISO6709` ou `©xyz`, date avec fuseau), en ne
  lisant que l'en-tête du fichier. Dans la vidéo, le clip est joué pendant
  l'arrêt du point, plafonné par **Vidéo max. (s)** (12 s par défaut). À
  l'export, chaque image est calée exactement sur l'instant du clip. Le son
  n'est pas repris.

### Ajouter des médias à une trace

Sélectionnez la trace, puis **📁 Dossier de la trace active**. À l'ouverture
d'un projet, les dossiers sont retrouvés automatiquement (voir « Projet »).

**Vidéo « illisible »** : le survol de la mention donne la raison exacte. Une
erreur de lecture passagère (fréquente avec les vidéos HDR d'iPhone) est
retentée une fois automatiquement. Si le navigateur ne sait vraiment pas la
décoder, convertissez-la en H.264 avec l'outil de macOS, puis recopiez sa
position et sa date :

```bash
avconvert --preset Preset1920x1080 --source IMG.mov --output IMG.mp4
```

```bash
exiftool -overwrite_original -TagsFromFile IMG.mov -Keys:GPSCoordinates -Keys:CreationDate IMG.mp4
```

Le bandeau de l'onglet Carte & photos a trois filtres : **Tous**,
**Géolocalisés**, **Non placés** (avec les compteurs).

- **Sphère bleue** : photo géolocalisée par son GPS. **Sphère orange** :
  position placée à la main. **« ? »** : pas de position. Un clic sur la
  vignette « ? » (ou sur 📍 pour déplacer n'importe quelle photo) passe en mode
  placement : le clic suivant sur la carte fixe sa position (Échap annule).
- Une position placée à la main reste prioritaire sur le GPS. Pour une photo
  qui a les deux, **⌖** (sur la vignette) revient au GPS, et le bouton
  **⌖ Rétablir les positions GPS** le fait pour toutes.
- Les positions placées à la main et les choix d'inclusion sont mémorisés
  dans le navigateur **et** dans le projet `.json`, trace par trace (clé : nom,
  taille et date du fichier), et reviennent quand on rouvre le même dossier.
  Un fichier retouché ou réexporté (autre date) les retrouve par son nom.
  Un projet rouvert signale par ⚠ les traces dont il faut recharger le
  dossier. Les fichiers eux-mêmes ne sont jamais modifiés.
- **Aller-retour, boucles** : quand la trace repasse au même endroit, une
  photo est près de plusieurs passages. Le passage retenu est celui qui
  respecte l'ordre chronologique des prises de vue (une photo plus tardive est
  plus loin sur la trace), même si le GPX n'est pas horodaté. Le bouton **⇄**
  de la vignette bascule vers l'autre passage ; ce choix est mémorisé.
  Pour un point de passage, le champ km de la liste choisit le passage.
- **Détours** (une cascade à 2 km de la route…) : un média « hors trace »
  peut être coché à la main ; il s'affiche au passage du point de la trace le
  plus proche (« km 66,1 · détour 2,5 km »).
- Une photo est rattachée à la trace active si elle est à moins de
  l'**écart max.** (200 m par défaut) ; sinon elle est marquée « hors trace ».
  La case **vidéo** de chaque vignette choisit les photos à montrer.

### Placement par l'heure (à valider)

**🕒 Proposer les positions par l'heure** calcule une position pour les photos
sans GPS, sans rien appliquer : elles apparaissent en pointillés, et chaque
proposition se valide (✓) ou se rejette (✗), ou toutes d'un coup.

1. Si une trace chargée est **horodatée** et couvre l'heure de la photo, la
   position vient du GPX. L'heure EXIF n'a pas de fuseau : elle est ramenée
   en UTC par le fuseau EXIF s'il existe, sinon par un décalage calé
   automatiquement sur les photos qui ont une heure GPS, sinon par le champ
   **Horloge − UTC (h)**.
2. Sinon, la position est **interpolée entre les photos géolocalisées** de la
   trace active, au prorata du temps (même horloge, aucun décalage à
   connaître). Avant la première ou après la dernière photo géolocalisée,
   elle est extrapolée à la vitesse moyenne et signalée « à vérifier ».

### Présentation dans la vidéo

- **Mise en avant** (par défaut) : la photo s'envole de sa sphère sur la carte
  jusqu'au grand format centré ; la carte se floute et s'assombrit derrière,
  les indicateurs s'effacent ; la photo zoome lentement (Ken Burns), avec sa
  date et son kilométrage incrustés. Le point s'arrête presque le temps de la
  photo. Les photos d'un même groupe s'enchaînent en fondu (« 2 / 4 »), puis
  la dernière retourne à sa sphère.
- **Encadré discret** : la photo en haut à droite, façon tirage, pendant que
  le point ralentit.

Dans les deux cas, chaque photo retenue a une sphère sur la carte à son
emplacement et un point sur la mini-carte.

**Ralentissement** : les photos proches (moins de max(150 m, 2 % de la trace)
entre deux photos) forment un groupe. Chaque groupe de n photos ajoute
≈ n × durée par photo × 0,85 s au parcours, réparti autour de la zone par une
courbe douce : le point décélère, traverse lentement la zone pendant que les
photos défilent, puis réaccélère. Sur la trace de test, 3 photos à Plan Lachat
font passer le point de 266 m/s à ~50 m/s. La durée de la vidéo augmente
d'autant ; la case « Ralentir le point aux photos » désactive l'effet.

## Projet

**Un projet = un dossier**, par exemple :

```
replay/
├── full.gpxcine.json             (où vous voulez dans le dossier)
├── gpx/                          les .gpx d'origine (font foi s'ils changent)
└── photos/
    ├── 20260815/                 photos et vidéos du 15 août
    └── 20260816/
```

- **📂 Ouvrir** (en haut du panneau) : choisissez le dossier du projet. Le
  fichier `.gpxcine.json` le plus récent est ouvert, puis les photos/vidéos de
  chaque trace sont **rechargées automatiquement** : depuis le dossier noté
  dans le projet, sinon depuis le sous-dossier qui porte le nom ou la date du
  GPX (`20260816` ↔ `20260816.gpx`). Un dossier sans projet mais contenant
  des GPX devient un nouveau projet. On peut aussi y glisser-déposer le
  dossier.
- **GPX modifiés** : les fichiers `.gpx` présents dans le dossier du projet
  (par exemple `replay/gpx/`) font foi. À l'ouverture, une trace dont le
  fichier a changé est mise à jour : points de passage replacés sur la
  nouvelle trace d'après leur position, départ et arrivée aux nouvelles
  extrémités, titres et réglages conservés (sous-titre et durée recalculés
  s'ils étaient restés à leur valeur par défaut). Un GPX nouveau dans le
  dossier est ajouté au projet. Enregistrez ensuite pour garder la mise à jour.
- **💾 Enregistrer** (ou ⌘S) : écrit directement dans le fichier du projet ;
  la première fois, choisissez le dossier où le créer. Les dossiers de médias
  y sont notés en chemins relatifs.
- Contenu : traces GPX, titres, sous-titres, durées, points de passage (nom,
  position exacte, couleur), départ/arrivée, réglages, trace active, et pour
  chaque média sa position placée à la main et son choix d'inclusion. Les
  fichiers photo/vidéo eux-mêmes restent dans leurs dossiers.
- L'ouverture et l'enregistrement dans un dossier nécessitent Chrome ou Edge.
  Ailleurs : ouverture d'un fichier `.json`, enregistrement par
  téléchargement, dossiers de médias à rechoisir (⚠ dans la liste des traces).

## Points de passage

- **Départ** et **Arrivée** sont nommés dans les deux champs en haut de la
  section : les repères sont créés aux extrémités de la trace. Si le GPX
  contient déjà un `<wpt>` à une extrémité, c'est son nom qui est repris.
  Vider un champ retire le repère.
- Les balises `<wpt>` du GPX sont importées et accrochées à la trace.
- **Placer sur la carte** : la carte devient interactive (zoom/déplacement),
  chaque clic ajoute un point accroché à la trace ; les repères se glissent.
- **+ À la position** : ajoute un point à la position courante de la réglette.
- Dans la liste, le champ de droite (km) repositionne le point le long de la trace.

Au passage d'un point, son nom s'affiche en bandeau et son repère s'allume.

### Couleur des tronçons

La pastille de couleur d'un point fixe la couleur de la trace **à partir de ce
point**, jusqu'au prochain point qui en définit une autre. Une pastille estompée
signifie « couleur héritée » : le tronçon précédent continue. Le premier tronçon
prend la couleur générale (section Habillage), et l'arrivée n'a pas de pastille
puisqu'aucun tronçon n'en part. ↺ revient à la couleur héritée.

La trace à venir (pointillés) prend déjà la couleur de ses tronçons. La couleur
du tronçon en cours s'applique aussi au repère de position, à la distance
affichée, au remplissage du profil altimétrique, au triangle de cap réel de la
rose des vents et à la mini-carte. Elle est conservée
dans le projet `.json`.

## Rose des vents

Un compas de marine en haut à gauche indique l'orientation de la carte : le
cadran tourne avec elle (le N rouge pointe toujours le vrai nord), la ligne de
foi blanche fixe en haut et le cap affiché dessous (« NE · 042° ») donnent la
direction de la caméra. Un second triangle, dans la couleur du tronçon en cours,
indique le **cap réel** (direction du déplacement sur la trace) : l'écart entre
les deux triangles montre de combien la caméra, lissée, diffère de la route. Elle fait partie de l'image exportée ; case *Rose des vents*
dans Habillage pour la masquer.

## Mini-carte

À droite de la rose des vents, une vignette **nord toujours en haut** montre
l'emprise de la trace active agrandie de 20 % : fond de carte, trace complète,
parcours effectué dans la couleur de chaque tronçon, position actuelle et un
cône indiquant la direction de la caméra. Les autres traces chargées y
figurent en gris si elles passent dans la zone. Elle apparaît quand le titre
s'efface et disparaît pour la vue finale. Case *Mini-carte* dans Habillage.

## Réglages notables

- **Épaisseur de la trace active** (1 à 2,5×, défaut 1,5×) : la trace suivie
  est dessinée plus épaisse que les traces inactives, qui restent fines et
  grises.

- **Zoom de suivi / Inclinaison** : hauteur et angle de la caméra.
- **Position du point à l'écran** : place le point courant plus ou moins bas,
  donc plus ou moins de visibilité « devant ».
- **Anticipation** : distance de visée en avant pour calculer le cap.
- **Lissage du cap** (m) : lissage géométrique de la direction de la trace.
- **Douceur de la caméra** (s) : inertie de la rotation, appliquée dans le
  domaine temporel de l'animation (filtre à phase nulle, donc sans retard de
  la caméra sur la trajectoire).
- **Rotation maximale** (°/s) : plafond de vitesse angulaire. C'est le réglage
  déterminant dans les lacets de montagne : sur la trace de test, le cap brut
  de la trace atteint 88 °/s alors que la caméra reste à 25 °/s.

- **Liberté du point à l'écran** (%) : autorise le point à s'écarter du centre
  (jusqu'à cette fraction de la largeur d'image). La caméra suit alors la ligne
  moyenne du parcours au lieu de chaque lacet : dans une suite d'épingles, la
  carte ne balaie plus d'un bord à l'autre. 0 % = point toujours au centre.
  Sur la trace de test, à 15 % (défaut), l'accélération de la caméra est
  divisée par 8,5 en moyenne et par 19 en pointe.

Ces réglages dépendent de la durée du parcours : le cap est
recalculé à chaque changement, et reste entièrement déterministe pour que
l'export image par image soit identique à l'aperçu.
- **Défilement** : vitesse constante, ou respect des horodatages du GPX
  (les arrêts sont alors visibles).
- **Relief 3D** : élévation issue des tuiles Terrarium (Mapzen/AWS).

## Export

- MP4 H.264 1920×1080, 24/30/60 i/s, 8 à 28 Mb/s.
- Rendu déterministe : chaque image attend le chargement complet des tuiles,
  aucune tuile floue ni saccade. L'export continue si la fenêtre passe en
  arrière-plan.
- **Préchargement des tuiles** (coché par défaut) : avant de rendre les
  images, l'application parcourt la trajectoire de la caméra, relève les
  tuiles dont chaque vue aura besoin et les télécharge en parallèle dans un
  cache en mémoire. Le bouton **⚡ Précharger les tuiles** fait la même chose à
  la demande, pour une lecture fluide dans l'aperçu.
- Mesuré sur la trace de test : ~145 ms par image sans préchargement,
  **~45–60 ms** après. Une vidéo de 54 s à 30 i/s (1 620 images) sort en
  **1 min 41**, préchargement compris, au lieu d'environ 4 min 30.
- Le cache garde jusqu'à 700 Mo de tuiles pour la session (état affiché sous le
  bouton) ; les tuiles de relief n'ayant pas d'en-tête de cache HTTP, c'est
  lui qui évite de les retélécharger.
- Le fichier est assemblé en mémoire : prévoir ~2 Mo par seconde de vidéo.
- Navigateur sans WebCodecs : repli sur une capture temps réel (WebM, ou MP4 sur Safari).

## Sources de données

- Fond satellite : Esri World Imagery (© Esri, Maxar, Earthstar Geographics)
- Fond topographique : OpenTopoMap (© OpenStreetMap contributors)
- Relief : tuiles Terrarium (Tilezen / Mapzen, hébergées par AWS)
- Rendu : MapLibre GL JS · Multiplexage MP4 : mp4-muxer

Usage personnel : vérifier les conditions d'utilisation de ces services pour un
usage commercial ou intensif.

## Fichiers

- `index.html` — toute l'application (interface, moteur d'animation, export)
- `samples/photos/` — photos de test géolocalisées le long des traces (dont
  un groupe de 3 à Plan Lachat, une hors trace et une sans GPS)
- `samples/` — traces de test voisines : Télégraphe → Valloire, Valloire →
  Galibier, Galibier → Lautaret (pour essayer le chargement d'un dossier)
- `*.gpxcine.json` — projets enregistrés (trace + réglages + points)
