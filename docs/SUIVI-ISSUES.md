# Suivi des six points — 1 novembre 2026

Les numéros de la liste transmise sont des repères. Les tickets GitHub créés
pour les suivre portent les numéros **4 à 9** (1 à 3 étaient des pull requests).
Base vérifiée : `main` au commit `3734837`, après fusion des PR #1, #2 et #3.
Les nouveaux changements sont proposés sur `codex/follow-up-project-issues` ;
leur présence dans cette branche ne signifie pas encore leur intégration à main.

| Point | Ticket | Résultat de la vérification et suite |
| --- | --- | --- |
| 1 — Scènes du build | [#4](https://github.com/Narani-bit/Crecle-chatgpt/issues/4) | Déjà intégré par la PR #1 : le profil Quest contient uniquement CercleQuest, à l'index 0. SampleScene est exclue des builds et son original est conservé. APK reconstruit ; installation/lancement sur casque à valider. |
| 2 — Identité | [#5](https://github.com/Narani-bit/Crecle-chatgpt/issues/5) | Corrigé dans cette branche : société et produit CercleVR, identifiant com.cerclevr.game, version 0.1.0 ; réglages globaux et overrides Quest cohérents. Valeurs confirmées dans l'APK. |
| 3 — Secrets et livrables | [#6](https://github.com/Narani-bit/Crecle-chatgpt/issues/6) | Exclusions de signature, secrets et symboles ajoutées ; APK/AAB déjà exclus. Contrôles des règles et analyse ciblée de l'historique réussis. Procédure de livraison documentée. |
| 4 — Modèle Bug | [#7](https://github.com/Narani-bit/Crecle-chatgpt/issues/7) | Modèle ajouté avec tous les champs demandés, en-tête YAML valide. Sa présence dans New issue reste à confirmer après fusion sur main. |
| 5 — Pipeline URP | [#8](https://github.com/Narani-bit/Crecle-chatgpt/issues/8) | Référence rendue explicite sous Assets, sans changement de MSAA ni de Render Scale. Asset chargé, scène exécutée et APK compilé. Détails et limites ci-dessous. |
| 6 — Origine Floor | [#9](https://github.com/Narani-bit/Crecle-chatgpt/issues/9) | Déjà intégré par la PR #2 : override Floor dans la scène et générateur cohérent. Hauteur debout/assis, sol réel et recentrage restent à réceptionner sur casque. |

## Modifications précises

- `ProjectSettings/ProjectSettings.asset` et le profil `Meta Quest.asset` :
  remplacement de DefaultCompany et des identifiants hérités Android, Standalone
  et iPhone par les valeurs confirmées. Les champs `templatePackageId` et
  `templateDefaultScene` décrivent la provenance du projet ; ce ne sont pas des
  identifiants de l'application et ils sont conservés.
- `.gitignore` : ajout de `*.keystore`, `*.jks`, `*.pfx`, `*.symbols.zip`,
  `.env`, `.env.*` (exception `.env.example`) et `google-services.json`.
- `.github/ISSUE_TEMPLATE/bug.md` : commit/version, mode PC/Simulator/Air Link/APK
  Quest 3, scène, niveau/graine, reproduction, résultats, capture, gravité et responsable.
- `Assets/Settings/Quest_RPAsset.asset` et son `.meta` : copie du pipeline Quest
  initial. GUID `6a0f5bfd05d255e498088b08d44f59b1`, référencé par le niveau de
  qualité Meta Quest et le pipeline de secours de `GraphicsSettings.asset`.
  Hors espaces de fin de ligne normalisés, seul le nom diffère de la source.
- README : identité, pipeline, livraison hors Git et suivi des bogues documentés.

**Rectification du diagnostic initial :** le GUID
`83a190b5cf963b042993eaddb87d0522` n'était pas manquant. Unity le résout vers
`Packages/com.unity.render-pipelines.universal/Runtime/Data/UniversalRenderPipelineAsset.asset`.
La copie locale rend cette configuration explicite dans le dépôt. Elle conserve
**MSAA 1 et Render Scale 1**. `Mobile_RPAsset` conserve son Render Scale 0,8 et
n'a pas été substitué au pipeline Quest.

Aucun original, script, scène ou doublon n'est supprimé dans ce suivi.

## Vérifications effectuées

- **211 contrôles Unity Editor réussis**, via `CercleCourseChecks.Run()` :
  scènes, références et principes des cours déjà implémentés.
- **Play CercleQuest**, qualité Meta Quest : `CoursePlayCheck` réussi,
  83 images supplémentaires en 1,20 seconde, XR actif et pose valide dans Meta XR
  Simulator ; carnet ancré dans le monde à environ 1 m, interaction de page et
  contenu du journal vérifiés. Capture inspectée : aucun matériau rose visible.
  Ce petit échantillon ne constitue pas une mesure de performance sur Quest.
- **Build natif du profil Meta Quest réussi**, IL2CPP ARM64, 167,66 s,
  **0 erreur / 5 avertissements**. APK : 50 821 764 octets.
  Lecture du manifeste avec `aapt` : package `com.cerclevr.game`, label CercleVR,
  versionName `0.1.0`, versionCode `1`, architecture `arm64-v8a`.
- Archive locale de symboles créée : 229 352 454 octets.
- Règles Git vérifiées, y compris création temporaire puis retrait de
  `test.keystore` : ignoré et absent des fichiers candidats à un commit.
- Analyse ciblée des **8 commits accessibles et 479 objets texte distincts** :
  aucun chemin sensible suivi ni secret correspondant aux motifs contrôlés
  (clés privées et formats courants de jetons, affectations de secrets).
  Les recherches historiques de champs de signature n'ont révélé aucune valeur
  de mot de passe. Ce contrôle ne garantit pas l'absence de tout secret possible.
- En-tête YAML du modèle Bug analysé ; `git diff --check` réussi.

Les 732 assertions Core documentées dans le README avaient été validées lors
des corrections précédentes ; elles ne sont pas présentées comme relancées ici.
Aucun code de jeu n'a changé dans cette série.

## Limites et points restant ouverts

- Aucun casque ADB connecté : pas d'installation de cet APK ni de validation
  physique du sol, de la hauteur, du recentrage ou du confort. Les tickets #4 et
  #9 restent ouverts pour ces essais ; consigner appareil, commit et résultats.
- Le build conserve quatre avertissements fournisseurs URP
  `LightingPhysicallyBased` (DX11/Vulkan). Le cinquième concerne l'échec d'envoi
  des symboles à Unity Cloud : authentification/configuration de service indisponible.
  L'APK et les symboles locaux ont néanmoins été produits. Les anciens avertissements
  TMP absents de ce build incrémental ne sont pas déclarés corrigés ; voir
  [AVERTISSEMENTS-BUILD.md](AVERTISSEMENTS-BUILD.md).
- Au lancement du Play avec le profil Android actif, OpenXR a émis quatre
  erreurs `OpenXRFeature.enabled cannot be changed while OpenXR is running`,
  depuis `BuildProfileUtilityOpenXR.InitializeMetaQuestSettings` du paquet Unity.
  Le simulateur a aussi signalé ADB inaccessible et un délai RPC dépassé.
  Le contrôle Play a ensuite réussi. Cette session n'est donc pas une validation
  de console sans erreur ; aucun paquet fournisseur n'a été modifié pour masquer
  ces messages.
- Le modèle Bug n'apparaît sur GitHub qu'après intégration dans la branche par
  défaut ; le ticket #7 garde ce dernier contrôle explicite.
- L'APK final doit être livré via Release GitHub ou le lien convenu après les
  essais sur appareil. Aucun APK, symbole ou secret n'est ajouté au dépôt.

Artefacts locaux de validation : `Builds/Courses-Quest/CercleQuest.apk`,
`CercleQuest.apk.report.json` et `CercleQuest-0.1.0-v1-IL2CPP.symbols.zip`.
