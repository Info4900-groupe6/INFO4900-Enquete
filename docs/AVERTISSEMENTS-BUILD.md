# Vérification des avertissements du build Quest — 31 octobre 2026

Le build de contrôle réussit sur Unity **6000.6.0f1**, Android IL2CPP ARM64 :
**0 erreur, 7 avertissements**, en **347,73 secondes**. Il reste donc des
avertissements à résoudre dans les composants Unity ; ce bilan ne certifie pas
un build sans avertissement.

## Corrections du projet

| Origine | Correction et vérification |
| --- | --- |
| Diagnostics Android activés avec symboles désactivés | Profil `Meta Quest` réglé sur `Full`. La configuration Android générée contient effectivement `DebugSymbol.Level = Full`. Une archive de symboles est produite à côté de l'APK. |
| Profil passé à `BuildPlayerWithProfileOptions` mais non actif | Dans cette version d'Unity, un essai a encore utilisé `None` malgré `Full` enregistré dans le profil. `CercleProfileBuild` exige maintenant que le profil demandé soit actif et que compilation/importation soient terminées. L'activation est séparée du build pour laisser la recompilation de plateforme se terminer. |
| Directive de shader TMP obsolète | Les trois occurrences de `enable_d3d11_debug_symbols` dans `TMP_SDF-Mobile.shader` et `TMP_SDF-Mobile SpaceWarp.shader` deviennent `enable_debug_symbols`. Le code de rendu et les GUID restent inchangés. |
| Ancienne métadonnée de police, découverte au premier nouveau build | Unity a resérialisé `LiberationSans.ttf.meta` de la version 2 à la version 4. Le GUID et le fichier de police sont conservés. L'avertissement ne réapparaît plus dans le rapport final. |

Pour construire : activer **Meta Quest** dans **File > Build Profiles**, attendre
la fin de l'importation/compilation, puis choisir **Cercle VR > Build > Quest avec
son profil**. Même préalable avec **Cercle PC** pour son menu de build. Le lanceur
refuse désormais un profil inactif avant de lancer la compilation.

## Sept avertissements encore présents

| Nombre | Source | Constat |
| --- | --- | --- |
| 4 | URP 17.6.0, `ShaderLibrary/Lighting.hlsl:93`, `LightingPhysicallyBased` | Variable potentiellement non initialisée : `ClusterDeferred`/DX11, `StencilDeferred`/DX11, `Lit`/DX11 et `Lit`/Vulkan. Ils se reproduisent après sauvegarde hors projet du cache de shaders, redémarrage et réimportation. Leurs chemins désignent maintenant le projet courant. Ce ne sont donc pas seulement des messages périmés. La disparition d'une des cinq variantes du rapport initial n'est pas présentée comme un correctif du code URP. |
| 3 | TextMesh Pro / IL2CPP | Trois méthodes volumineuses sont placées dans leur propre fichier C++ : `TMP_TextParsingUtilities::.cctor`, `TextMeshPro::GenerateTextMesh`, `TextMeshProUGUI::GenerateTextMesh`. Ce diagnostic correspond à la régression Unity **UUM-145600**. |

La recherche effectuée par `UnityEditor.PackageManager.Client.Search` dans cet
éditeur retourne uniquement **URP 17.6.0** et **uGUI 2.6.0** comme versions
disponibles, recommandées et compatibles. Aucun correctif compatible n'a été
identifié dans ces résultats. Les paquets fournis par Unity n'ont pas été
modifiés ; une correction de leurs sources ou une migration de l'éditeur
nécessiterait une validation distincte du rendu et de la chaîne de compilation.

Deux avertissements concernant les shaders HDRP `Hidden/Core/DebugOccluder` et
`DebugOcclusionTest` étaient apparus dans le premier essai avec profil inactif.
Ils sont absents du build final avec le profil Quest actif. Aucune ressource de
diagnostic n'a été supprimée pour obtenir ce résultat.

## Vérifications et livrables

- **211 contrôles Editor** réussis sur les scènes et les principes des cours.
- Contrôle du nouveau garde-fou réussi : profil inactif refusé et date de l'APK
  inchangée ; profil initial rétabli après le contrôle.
- Compilation C# réussie ; paramètres globaux `ProjectSettings.asset` inchangés
  par comparaison SHA-256 avec la sauvegarde avant cette intervention.
- APK : `Builds/Courses-Quest/CercleQuest.apk`, **50 822 628 octets**.
  SHA-256 : `c7f227b8f52ebde001819f052b8ca544ef4231e66c17e2dcdedbce520159b9eb`.
- Symboles : `Builds/Courses-Quest/CercleQuest-0.1.0-v1-IL2CPP.symbols.zip`,
  **229 352 454 octets**. Garder cette archive avec l'APK correspondant.
- Rapport intégral local : `Builds/Courses-Quest/CercleQuest.apk.report.json`.
  Les diagnostics n'ont pas été filtrés dans le rapport.

Les fichiers modifiés ont été sauvegardés dans le dossier local
`warning-review/before` de l'espace de travail, ainsi que l'ancien cache de
shaders. Aucun original de scène, PDF, police ou ressource n'a été supprimé.
Les APK, symboles et caches restent hors du dépôt Git. La cible de l'éditeur
est rétablie à Windows après les essais. Aucun essai sur casque physique n'a
été réalisé lors de cette correction.

## Sources Unity consultées

- [Symboles Android et génération de l'archive](https://docs.unity.com/en-us/engine/6000.6/manual/platform-specific/android/developing/testing-and-debugging/symbols).
- [Remplacement de la directive de débogage des shaders](https://unity.com/releases/editor/alpha/6000.6.0a5).
- [Régression TextMesh Pro UUM-145600](https://issuetracker-mig.prd.it.unity3d.com/issues/textmeshpro-warnings-are-thrown-after-building-ar-mobile-vr-and-mixed-reality-projects).
