# Application des cours à CercleVR

Lecture du 31 octobre 2026 : **10 PDF, 201 pages**, présents dans le dossier local
`Document/`. Le texte de toutes les pages a été extrait et lu ; douze pages clés
de schémas et de recommandations ont également été rendues et examinées.
Les PDF originaux restent intacts et ne sont pas ajoutés au dépôt.
L'inventaire et les empreintes SHA-256 sont dans [course-sources.json](course-sources.json).

Les numéros ci-dessous sont les **pages PDF**, première page comprise. Les cours
décrivent aussi des modèles pédagogiques et des fonctions matérielles : les
appliquer ne signifie pas réimplémenter le suivi ou l'optique d'un Quest.

## Correspondance entre les dix cours et le projet

| Cours | Pages / principes retenus | Application et preuve |
| --- | --- | --- |
| `1_INTRODUCTION.pdf` — 15 p. | 6, 8–10 : perception, cohérence des sens et architecture ; 15 : traçabilité du projet collectif | Séparation Core / comportements Unity / runtime OpenXR conservée. Confort des déplacements et de la lecture amélioré. Changements, tests et limites tracés dans Git et ce document. La répartition des rôles entre étudiants reste à compléter par l'équipe. |
| `2_Transformations 3d_PartieI.pdf` — 30 p. | 2–5, 18–30 : repères, TRS, composition non commutative, transformations actives/passives | XR Origin à l'échelle 1 ; transformations locales conservées pour les objets et boutons. Placement du carnet par position et quaternion, sans imposer de mouvement à la caméra suivie. Tests d'une chaîne objet → monde → caméra et de son inverse. |
| `2_Transformations 3d_PartieII.pdf` — 24 p. | 2–6, 11–24 : inverse, changements de repère, quaternions | `ClipboardMount.ReadingPose` garde une orientation de lecture sans roulis hérité ; cas tête verticale testés. Vérifications de conservation des distances et d'inverse quaternion. Aucun filtrage ajouté au suivi de tête. |
| `3_PipelineGraphique_frustum de vue.pdf` — 13 p. | 3–13 : perspective, clipping, near/far et projection | Caméras PC et Quest : near **0,05 m**, far **30 m**, adaptés à la salle de 12 m. Ces valeurs sont des choix du projet, pas des valeurs imposées par le PDF. FOV et matrices stéréoscopiques restent fournis par OpenXR. |
| `3_PipelineGraphique_Unity_VR - 2.pdf` — 14 p. | 2–11 : Editor/Player, composants, hiérarchie ; 12–14 : shaders et budget d'image | Réglages enregistrés dans les scènes et reproduits par le générateur. Tests de compilation et Play séparés du build Android. URP et ses shaders existants conservés. Budget GPU/CPU à mesurer sur casque selon le protocole ci-dessous. |
| `4-Perception_visuelle_PartieI-1.pdf` — 24 p. | 5–17, 23 : indices de profondeur, parallaxe, échelle, IPD et accommodation | Carnet VR ancré dans le monde pendant la lecture ; déplacement/rotation du rig ne déplacent pas le carnet. Sol et hauteur réelle de suivi cohérents avec Floor. Les objets, occlusions et perspectives du décor sont conservés. Aucune IPD artificielle. |
| `5-HMD_I.pdf` — 16 p. | 6–14 : optique, frustums asymétriques, distinction plan proche / plan focal | Une Main Camera XR avec Tracked Pose Driver ; aucune seconde caméra créée pour les yeux. OpenXR conserve les vues/projections par œil. Les plans de découpage ne prétendent pas changer l'accommodation optique. |
| `5-HMD_II.pdf` — 20 p. | 2–17 : distorsion et aberration chromatique ; 18–20 : holographie | Aucune seconde correction de lentille ni aberration chromatique artificielle ajoutée au rendu. La compensation matérielle reste à la charge du runtime du casque. Pas de simulation holographique sur un afficheur à plan focal fixe. |
| `5-Perception_visuelle_PartieII_Conseils de conception par Meta.pdf` — 15 p. | 2–8 : couleur ; 9–14 : distance, stéréo, HUD, réticule, origine de suivi | Lecture placée à environ **1 m** plutôt que 0,82 m ; minimum axial de placement 1 m en VR. Carnet physique avec Canvas World Space et collisions 3D, sans surimpression forcée. Origine Floor. Flux Linear existant conservé, aucune conversion sRGB/P3 arbitraire. Les états ont déjà des mots/numéros en plus des couleurs. |
| `6-IMU.pdf` — 30 p. | 2–14 : pose, calibration, dérive ; 15–28 : fusion ; 29–30 : rôle Unity/OpenXR | `XrTrackingGuard` exige une position et une orientation valides du runtime avant les actions du jeu et le rappel du carnet. Un périphérique 3DDL, déconnecté ou non suivi ne suffit pas. Aucun intégrateur gyro ni filtre complémentaire superposé à la pose OpenXR. Tracked Pose Driver Update + Before Render conservé. |

## Changements de comportement

### Lecture et repères

Sur Quest, le carnet est placé dans le monde puis reste immobile lorsque la tête
ou le XR Origin se déplace. L'utilisateur bénéficie ainsi de la parallaxe attendue.
Le bouton **Lire** le place devant soi à environ un mètre. **Réduire** le replace
sur le côté. **C** ou **X sur la manette gauche** le rappelle en mode lecture après
un déplacement. Cette commande déplace uniquement le carnet.

Le placement initial attend LateUpdate, après la mise à jour de pose de la caméra,
et une pose valide du runtime. Une caméra non encore suivie au démarrage ne fige
donc pas le carnet dans un repère provisoire.

Sur PC, le carnet reste attaché à la caméra pour conserver l'usage souris ; son
mode lecture passe également à environ un mètre. Le bouton Lire/Réduire est mis
à jour quelle que soit la commande utilisée. Un `clipboardAnchor` explicite reste
compatible avec un carnet tenu en main ; Lire le place dans le monde en VR.

La distance est vérifiée **au placement**, pas garantie si l'utilisateur avance
physiquement ensuite. Le cours recommande 0,5 m comme minimum pour une fixation
prolongée et 1 m pour une interface : le placement vise 1 m, sans modifier la pose
de l'utilisateur pour le maintenir à distance.

Unity utilise x droite, y haut, z avant. Les exercices emploient plusieurs
conventions de repères : leurs matrices ne sont pas copiées littéralement.
`Transform.rotation` et `localRotation` sont des quaternions ; le constructeur
Unity les ordonne `(x, y, z, w)`, contrairement à la notation `(w, x, y, z)` des
diapositives. Le changement de repère testé est
`p_camera = inverse(T_camera) * T_objet * p_local`.

### Déplacement et suivi

Le profil de déplacement initial du Starter Assets permettait une translation
continue sur la main gauche. Les deux gestionnaires de la scène Quest utilisent
maintenant **téléportation et rotation par crans** ; le cran de 45° existant est
conservé. C'est un choix de conception pour limiter les mouvements artificiels
continus, pas une mesure de confort validée sur tous les utilisateurs.

Le XR Origin demande **Floor** ; le runtime utilisé lors du test confirme ce mode.
L'échelle reste 1. La hauteur de secours de 1,65 m ne sert qu'au mode Device si
le runtime l'utilise : elle ne remplace pas la hauteur réelle en mode Floor.

Le contrôle de suivi bloque les actions `WorldAction` et le rappel du carnet en
l'absence d'une pose 6DDL valide. Il ne remplace pas la gestion système de la perte
de suivi et ne simule pas une nouvelle pose. La locomotion XRI reste gérée par XRI.

### Projection, rendu et couleur

Les plans 0,05/30 m couvrent la pièce et réduisent le rapport far/near inutile de
la configuration initiale. Ils ne modifient ni le FOV matériel, ni l'IPD, ni les
matrices par œil, ni la distance focale de l'afficheur. Les tests vérifient une
caméra perspective unique par scène et la présence du Tracked Pose Driver sur Quest.

Le projet utilise déjà le rendu Linear. Les couleurs d'état restent accompagnées
de numéros, libellés et symboles ; elles ne sont pas la seule information. Aucun
gamut P3 n'est déclaré sans ressources et matériel adaptés. Les captures miroir
servent à vérifier la disposition, pas à certifier les couleurs vues dans le casque.

Le build via le connecteur utilisait les options globales et incluait le pipeline
PC malgré l'activation du profil Quest. `CercleProfileBuild` transmet désormais
le **profil lui-même** à `BuildPlayerWithProfileOptions`, afin d'appliquer sa liste
de scènes et ses overrides de qualité, graphisme et Player Settings.

## Vérifications automatisées et essais

- **211 contrôles Editor** : scènes, scripts présents, clipping, caméra unique,
  Floor, échelle XR, suivi avant rendu, locomotion, cas limites de placement,
  validité de pose, composition/inversion des transformations et quaternions.
- **732 assertions Core** : 24 énigmes et quatre campagnes ; logique inchangée.
- **Play PC et Quest** : journal contenu dans sa zone, neuf Canvas World Space du
  carnet, texte ne capturant pas les rayons, bouton Page + atteint par un raycast.
- **Quest** : pose de lecture à 1,00045 m ; carnet immobile après translation de
  0,30 m et rotation de 45° du rig, restauré immédiatement après le test ; suivi
  valide et Floor confirmés. 73 images avancent pendant l'observation de 1,21 s.
  Le dernier contrôle après protection du démarrage retrouve le carnet latéral
  à 1,061 m, sans parent caméra, puis valide de nouveau la lecture avec 83 images avancées.
- **PC** : mode de lecture et parent caméra vérifiés ; 75 images avancent pendant
  l'observation de 1,21 s. Ces cadences d'éditeur ne mesurent pas les performances Quest.
- **Campagnes complètes** : six victoires PC et six victoires Quest, transitions et
  porte finale confirmées ; respectivement 835 et 857 images avancées.
- Captures PC/Quest examinées. Aucune nouvelle erreur de compilation ou exception
  de jeu relevée pendant ces essais.
- **Build Quest final par profil natif** : Android IL2CPP ARM64, réussite en
  113,15 s, **0 erreur / 10 avertissements**, APK de **50 822 632 octets** dans
  `Builds/Courses-Quest/CercleQuest.apk`. Le rapport confirme Mobile_RPAsset et
  le pipeline URP du niveau de qualité Meta Quest ; PC_RPAsset est absent.
  Le build intermédiaire via le connecteur n'est pas utilisé comme preuve de
  conformité des overrides du profil.

Un contrôle ultérieur corrige les symboles de diagnostic, les directives de
shader TMP et une ancienne métadonnée de police. Le dernier build réussit avec
**0 erreur / 7 avertissements URP/TMP** ; voir [le bilan des avertissements](AVERTISSEMENTS-BUILD.md).
Il exige désormais l'activation préalable du profil demandé. Windows IL2CPP n'a pas
été relancé : les outils C++ / SDK manquants lors de l'audit restent un prérequis.

Les tests de Play pilotent les comportements et font un raycast réel ; ils ne
prouvent pas l'appui physique sur X, les gestes de manette ou le confort humain.

### Relancer les vérifications

Dans Unity, hors Play : **Cercle VR > Tests > Verifier les principes des cours**.
Le test ne sauvegarde pas les scènes ; il ferme uniquement celles qu'il a ouvertes.
Un échec lève une exception avec la condition en cause.

Pour le Core :

```sh
dotnet run --project Tests/CoreChecks/CoreChecks.csproj --configuration Release
```

Les scripts `Tests/Unity/CoursePlayCheck.cs` et `Tests/Unity/PlayCampaignCheck.cs`
sont fournis hors d'Assets. Avec le connecteur facultatif, démarrer Play, attendre
que la partie soit prête puis les exécuter avec `script run`, entrées respectives
`CoursePlayCheck.Main` et `PlayCampaignCheck.Main`. Le second termine la campagne
de test : repartir d'une nouvelle session Play avant de le relancer.

## Vérification physique restante

| Essai | Critère observable |
| --- | --- |
| Taille et sol | En position debout puis assise, comparer sol, hauteur du regard, table et porte ; aucun changement d'échelle du rig. |
| Parallaxe et lecture | Pencher la tête latéralement : le carnet reste stable ; rappeler avec X, lire sans devoir s'approcher à quelques centimètres. Tester plusieurs tailles d'utilisateur. |
| Entrées | Viser Page +, Lire, Réduire et les personnages ; vérifier X, téléportation et crans de rotation, sans déclenchement double. |
| Perte de suivi | Perdre puis retrouver le suivi dans des conditions contrôlées ; aucune accusation/validation avec pose invalide, reprise des actions après récupération. |
| Stéréoscopie | Inspecter chaque œil puis les deux : même texte, occlusions cohérentes, pas d'effet apparaissant dans un seul œil. |
| Couleur et contraste | Vérifier dans le casque le texte sur papier, les noms et les états ; ne pas conclure depuis la capture ordinateur. |
| Performance | Profiler CPU et GPU sur l'appareil pendant lecture, téléportation et génération/transitions, en particulier avec 12 personnages. Relever fréquence effective et images manquées. |

Budgets issus du cours Pipeline Unity p. 14 : **72 Hz ≈ 13,9 ms**, **90 Hz ≈
11,1 ms**, **120 Hz ≈ 8,3 ms** par image. La fréquence effective du runtime doit
être mesurée ; elle n'est pas imposée à 90 Hz dans le code. Le temps CPU et le
temps GPU doivent chacun respecter le budget, avec de la marge. Aucun résultat
sur casque réel, latence mouvement-photon ou garantie d'absence d'inconfort
n'est revendiqué ici.

## Conservation et traçabilité

Les deux scènes gardent leurs GUID. Les réglages sont des overrides d'instance
sur le prefab XR ; les Starter Assets fournisseurs ne sont pas modifiés.
Les fichiers CercleVR et les paramètres globaux ont été sauvegardés localement
avant cette intervention. Aucun PDF, matériau, script ou scène préexistants n'a
été supprimé. Les captures temporaires ont été déplacées hors d'Assets.

L'optique holographique, les capteurs bruts, les haptiques ultrasonores et les
interfaces gustatives cités dans les cours nécessitent du matériel ou une autre
portée de projet. Ils ne deviennent pas des fonctionnalités fictives de ce jeu.
Les rôles d'équipe, réunions et contributions humaines demandés par l'introduction
restent à renseigner par les membres concernés, sans attribution inventée.
