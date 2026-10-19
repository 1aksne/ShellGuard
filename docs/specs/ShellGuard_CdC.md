# Cahier Des Charges Fonctionnel Et Technique

**Projet :** ShellGuard
**Version :** 1.0.0
**Auteur :** anne lukas ouhaj abderazzak
**Date :** 14/10/2026

---

## 1. Contexte et Objectifs

### 1.1 Contexte

Un build Docker peut impliquer un Dockerfile et plusieurs scripts Shell. Des
outils existants, tels que ShellCheck et Hadolint, apportent des contrôles
utiles, mais leur analyse individuelle ne suffit pas toujours à reconstruire
les relations entre instructions Docker, scripts copiés ou exécutés, variables
sensibles et utilisateur courant.

### 1.2 Problématique

La problématique de ShellGuard est donc la suivante : comment détecter des
risques de sécurité qui apparaissent dans les interactions entre un Dockerfile
et les scripts Shell associés, en analysant le build comme un ensemble, sans
exécuter les fichiers analysés ?

### 1.3 Objectifs

Concevoir et développer un analyseur statique qui relie les informations du
Dockerfile et des scripts Shell afin de détecter, localiser et rapporter des
risques de sécurité pertinents.

Découvrir les Dockerfiles et scripts Shell entrant dans le périmètre de
l’analyse. Analyser la structure des fichiers à l’aide de Tree-sitter et
de règles ciblées par expressions régulières. Relier les instructions COPY/ADD
aux chemins de fichiers dans l’image et les commandes qui exécutent les scripts.
Suivre, dans les limites définies, la propagation de valeurs sensibles et le
contexte de l’utilisateur par étape de build. Implémenter 14 règles de détection
identifiées dans la conception. Produire des rapports en terminal, JSON et SARIF.
Permettre une exécution automatisée dans GitHub Actions et une décision de réussite
ou d’échec selon une politique configurable. Évaluer l’outil sur des exemples
de régression et un corpus de projets réels.

### 1.4 Périmètre

- **In-Scope (Inclu) :**
    - Dockerfiles correspondant au motif `Dockerfile*`.
    - Scripts .sh et .bash, ainsi que les fichiers identifiés comme scripts
    Bash par une ligne shebang #!/bin/bash.
    - Fichiers ciblés par les instructions `COPY` et `ADD`, dans les limites de
    résolution définies.
    - Instructions Docker pertinentes, notamment `ARG`, `ENV`, `COPY`, `ADD`,
    `RUN`, `CMD`,
    ENTRYPOINT et USER.
    - Builds multi-stage, avec les limites de prise en charge documentées.
    - Analyse statique : aucun script ni commande du projet analysé ne doit être
    exécuté par ShellGuard.
    - Rapports terminal, JSON et SARIF.
    - Intégration prévue dans GitHub Actions.
- **Out-of-Scope (Exclu) :**
    - docker-compose et les fichiers Kubernetes YAML.
    - Scripts zsh et PowerShell.
    - Analyse du code applicatif général, notamment les fichiers Python et
    JavaScript qui ne sont pas dans le périmètre des fichiers analysés.
    - Inspection du contenu interne des images de base.
    - Observation du comportement à l’exécution des conteneurs.
    - Prise en charge garantie de toutes les extensions BuildKit ou de toutes
    les constructions Docker avancées.
    - Les jokers dans `COPY`, les instructions `COPY --from` entre étapes,
    `ADD`
    depuis une URL, les heredocs Dockerfile et certaines constructions Bash
    constituent des points à tester ou des limites connues ; ils ne doivent pas
    être présentés comme entièrement pris en charge sans validation.

---

## 2. Utilisateurs et Cas d'usage

### 2.1 Personas

- **P1 - Développeu / Sysadmin :** Utilise l'outil localement dans son terminal
pour vérifier ses scripts avant de commit.
- **P2 - Ingénieur DevOps / SecOps :** Intègre l'outil comme étape bloquante
dans les pipelines CI/CD.

### 2.2 Scénario d'usage type

1. **Préparation :** le développeur prépare son projet contenant un Dockerfile et
des scripts Shell.
2. **Lancement :** il exécute `shellguard scan .` dans le terminal.
3. **Analyse :** ShellGuard analyse les fichiers et identifie les liens entre
les instructions Docker et les scripts exécutés.
4. **Détection :** il recherche les risques de sécurité, comme les secrets
exposés, les commandes dangereuses et les privilèges excessifs.
5. **Rapport :** ShellGuard affiche les alertes avec le fichier concerné, la
ligne et le niveau de risque.
6. **Correction :** le développeur corrige les problèmes et relance l'analyse.
7. **Intégration CI/CD :** ShellGuard peut bloquer le pipeline GitHub Actions
si des alertes dépassent le seuil de gravité configuré.

---

## 3. Specifications Fonctionnelles

### 3.1 Priorisation MoSCoW

#### Must Have (Indispensable pour la v1)

- **Découverte ciblée (EF-01) :** Détecter les fichiers `Dockerfile*`, les
scripts `.sh`, `.bash`, ainsi que les fichiers avec shebang `#!/bin/bash`.
- **Analyse strictement statique (EF-02) :** Analyser le code sans jamais
exécuter les scripts ou les commandes de build.
- **Parsing AST (EF-03) :** Parser le Dockerfile et les scripts Bash à l'aide
de Tree-sitter, en réservant les regex à des détections ciblées (clés, tokens).
- **Modélisation Docker & Suivi d'état (EF-04 & EF-05) :** Lier les instructions
`COPY`/`ADD` aux scripts résolus et suivre par étape l'utilisateur (`USER`)
ainsi que les variables sensibles (`ARG`, `ENV`).
- **Moteur de détection (EF-07) :** Implémenter et tester les 14 règles de
sécurité définies (règles X-001 à X-006, B-001 à B-006, D-001 et D-002).
- **Gestion du niveau de confiance (EF-08) :** Réduire la confiance dès qu'une
relation ou un chemin ne peut pas être suivi avec certitude.
- **Rapports multi-formats (EF-09) :** Produire des comptes-rendus d'analyse
aux formats Terminal, JSON et SARIF.
- **Interface CLI et validation CI (EF-10 & EF-11) :** Proposer la commande
`shellguard scan .` et retourner un code de sortie selon la politique 
`fail_on = high` et `min_confidence = medium` pour GitHub Actions.

#### Should Have (Important)

- **Propagation des flux de données (EF-06) :** Effectuer une analyse de
propagation de valeurs (taint analysis) dans les scripts pris en charge (ex.
paramètre $\rightarrow$ variable $\rightarrow$`eval`).
- **Gestion des builds multi-stage :** Supporter les étapes de build
multiples dans les limites de conception documentées.
- **Cadre de benchmark et d'évaluation (EF-12) :** Automatiser les tests sur cas
réels avec le corpus prévu (~300 Dockerfiles, 500 scripts, 30–50 paires CVE) et
calculer précision/rappel.

#### Could Have (Optionnel)

- **Support des Dockerfile heredocs :** Gérer la syntaxe heredoc sous réserve
de validation par des tests spécifiques.
- **Résolution des jokers (wildcards) :** Gérer les motifs à jokers dans les
chemins sources de l'instruction `COPY`.
- **Prise en charge de `COPY --from` :** Suivre les transferts de fichiers
entre différentes étapes de build.
- **Téléchargement externe via `ADD` :** Résoudre les fichiers distants
récupérés depuis des URL.
- **Constructions Bash complexes :** Tenter l'analyse des scripts inclus avec
source, des blocs `trap` et de l'expansion indirecte `${!x}`.


#### Won't Have (Exclu de cette itération)

- **Orchestration et conteneurs avancés :** Prise en charge des fichiers
`docker-compose` et des manifests Kubernetes YAML.
- **Autres langages Shell :** Prise en charge des scripts Zsh et PowerShell.
- **Code applicatif général :** Analyse approfondie du code source applicatif (
fichiers Python, JavaScript, etc.) situé hors du périmètre de build.
- **Analyse dynamique et runtime :** Inspection du comportement à l'exécution
des conteneurs ou inspection du système de fichiers interne des images de base.
- **Intégrations IDE :** Plugins dédiés pour environnements de développement (
VS Code, Neovim, JetBrains).
- **Constructions Docker complexes non validées :** Prise en charge garantie de
toutes les extensions BuildKit avancées.


### 3.2 Matrice des Exigences Fonctionnelles

| ID | Module | Description | Priorité | Critère de validation |
| :--- | :--- | :--- | :--- | :--- |
| **EF-01** | Découverte | Découvrir automatiquement les Dockerfiles (`Dockerfile*`), scripts Shell (`.sh`, `.bash`) et fichiers avec shebang `#!/bin/bash` dans le dépôt. | Must | 100 % des fichiers cibles du périmètre identifiés sans analyser le code applicatif général hors périmètre. |
| **EF-02** | Sécurité d'analyse | Réaliser une analyse strictement statique sans jamais exécuter les scripts ou commandes analysés. | Must | Aucune commande système ni script ciblé n'est exécuté lors du scan. |
| **EF-03** | Moteur AST | Parser les Dockerfiles et les scripts Bash via Tree-sitter et restreindre les expressions régulières aux usages ciblés (clés, tokens). | Must | Génération complète de l'AST sans crash sur les syntaxes Bash et Dockerfile documentées. |
| **EF-04** | Modèle de relations | Relier les chemins de `COPY`/`ADD` aux scripts référencés dans l'image (instructions `RUN`, `CMD`, `ENTRYPOINT`). | Must | Résolution correcte de la correspondance entre chemin hôte et chemin image sur les cas standards. |
| **EF-05** | Suivi du contexte | Suivre les valeurs sensibles (`ARG`, `ENV`) et l'utilisateur courant (`USER`) à travers chaque étape de build. | Must | Conservation correcte de l'état en mémoire des variables sensibles et de l'utilisateur actif par étape. |
| **EF-06** | Propagation de flux | Effectuer une analyse de flux de données (*taint analysis*) limitée à l'intérieur des scripts Shell pris en charge. | Should | Détection de la transmission d'une valeur contaminée jusqu'aux instructions critiques (ex. `eval`, `sh -c`). |
| **EF-07** | Règles de détection | Implémenter et valider les 14 règles de sécurité définies (`X-001` à `X-006`, `B-001` à `B-006`, `D-001`, `D-002`). | Must | Validation de cas de test unitaires positifs et négatifs pour chacune des 14 règles. |
| **EF-08** | Gestion de confiance | Associer un niveau de confiance (`HIGH`, `MEDIUM`, `LOW`) à chaque alerte et le dégrader en cas d'incertitude dans la résolution. | Must | Présence systématique du champ de niveau de confiance dans chaque constat émis. |
| **EF-09** | Rapports | Générer les résultats sous trois formats distincts : affichage console (Terminal), format structuré (`JSON`) et format standard (`SARIF`). | Must | Schémas JSON et SARIF conformes et vérifiés par des tests automatisés. |
| **EF-10** | Politique CI | Fournir un code de sortie configurable pour décider du succès ou de l'échec du pipeline CI. | Must | Code de retour `1` si au moins une détection `HIGH` avec confiance $\ge$ `MEDIUM`, sinon code `0`. |
| **EF-11** | Interface CLI | Mettre à disposition la commande `shellguard scan .` pour lancer l'analyse sur un répertoire cible. | Must | Commande opérationnelle en local et dans l'action `.github/workflows/shellguard.yml`. |
| **EF-12** | Évaluation | Évaluer l'outil sur des cas de test unitaires et un corpus réel (~300 Dockerfiles, 500 scripts, 30–50 paires CVE). | Should | Rapport d'évaluation fournissant la précision et le rappel mesurés sur un jeu de test gelé. |

---

## 4. Spécifications Techniques et Contraintes

### 4.1 Stack Technologique & Écosystème Rust

- **Langage principal :** Rust (édition 2021/2024), garantissant la sûreté
mémoire, la rapidité d'exécution et la production d'un binaire autonome
statique.
- **Moteur d'analyse syntaxique (AST) :** Crate `tree-sitter` avec les
grammaires natives associées (`tree-sitter-bash` et parser Dockerfile)
pour l'extraction structurelle.
- **Moteur de regex & détection :** Crate `regex` pour le matching rapide de
patterns d'entropie, de formats de clés connus et d'heuristiques de nommage
(`TOKEN`, `PASSWORD`).
- **CLI & Formats de sortie :** Crates `clap` (gestion CLI avancée pour
`shellguard scan .`), `serde` / `serde_json` (sérialisation des rapports JSON
et du schéma standard SARIF v2.1.0).
- **Intégration & CI/CD :** Git et GitHub Actions
(`.github/workflows/shellguard.yml`).

### 4.2 Performances & Environnement

- **Systèmes cibles :** Linux (x86_64, aarch64), macOS.
- **Empreinte & Déploiement :** Binaire statique unique sans dépendance système
externe requise lors de l'exécution en pipeline CI.
- **Temps d'exécution :** Analyse quasi instantanée tirant parti de la vitesse
native de Rust lors du parcours de l'AST.

### 4.3 Sécurité de l'Analyseur (Principe Zéro-Exécution)

- **Analyse strictement passive :** ShellGuard n'exécute aucun binaire, script
ou directive `RUN` issus des projets scannés.
- **Isolation réseau :** Traitement 100 % hors-ligne sans requêtes externes.
- **Périmètre clos :** Aucune inspection du système de fichiers interne des
images de base ni observation de conteneurs au runtime.

### 4.4 Robustesse & Gestion de l'Incertitude

- **Gestion des erreurs :** Modèle d'erreurs robuste (`Result`/`Option`)
évitant les panics et assurant une gestion gracieuse des fichiers mal formés.
- **Indice de confiance :** Calcul d'un niveau d'incertitude explicite (`HIGH`,
`MEDIUM`, `LOW`) sur chaque constat, dégradé en cas d'ambiguïté sur les chemins
ou flux de variables.

---

## 5. Organisation, Jalons et Livrables

### 5.1 Livrables attendus

1. **Code source Rust :** Projet Cargo structuré en modules (`scanner`, `ast`,
`rules`, `report`), documenté et couvert par des tests unitaires et
d'intégration.
2. **Binaire autonome :** Exécutable compilé pour les plateformes cibles
(Linux x86_64/arm64, macOS).
3. **Moteur d'analyse Tree-sitter :** Parsing syntaxique robuste des
Dockerfiles et des scripts Bash couplé à la résolution des chemins hôte/image.
4. **Moteur de détection :** Implémentation complète et testée des 14 règles de
sécurité (`X-001` à `X-006`, `B-001` à `B-006`, `D-001`, `D-002`).
5. **Rapports multi-formats :** Sorties Terminal enrichies, flux JSON et
fichiers conformes SARIF v2.1.0.
6. **Action CI :** Fichier de workflow GitHub Actions configuré avec la
politique d'échec (`fail_on = high`, `min_confidence = medium`).
7. **Banc d'évaluation & Benchmark :** Scripts de test sur corpus réel
(~300 Dockerfiles, 500 scripts, 30–50 paires CVE) avec mesure documentée de
la précision et du rappel.

### 5.2 Jalons prévisionnels (Planning sur 12 semaines)

| Phase | Semaines | Périmètre / Objectifs | Livrable clé |
| :--- | :--- | :--- | :--- |
| **M1** | S1 | Initialisation du workspace Cargo, configuration des crates de base, interfaces et tests syntaxiques préliminaires. | Workspace Rust et jeux de tests initiaux |
| **M2** | S2 – S3 | Intégration des bindings Tree-sitter, découverte des fichiers cibles et génération de l'AST. | Module de parsing AST opérationnel |
| **M3** | S4 – S5 | Modélisation des chemins `COPY`/`ADD`, gestion du multi-stage et suivi de l'état `USER`, `ARG` et `ENV`. | Graphe de contexte de build validé |
| **M4** | S6 – S8 | Implémentation modulaire des 14 règles de sécurité et couverture de tests unitaires (cas positifs/négatifs). | Moteur de détection (14 règles) |
| **M5** | S9 – S10 | Sérialisation Terminal, JSON et SARIF via `serde`, gestion des codes d'erreur CLI et workflow GitHub Actions. | Binaire CLI et action CI opérationnelle |
| **M6** | S11 – S12 | Évaluation sur le corpus de benchmark gelé, double annotation humaine, rapport final et préparation de la soutenance. | Rapport technique et release binaire v1.0 |

### 5.3 Gestion des Risques

| Risque identifié | Probabilité | Impact | Stratégie de mitigation |
| :--- | :---: | :---: | :--- |
| **Courbe d'apprentissage des bindings C / Tree-sitter en Rust** | Moyenne | Moyen | Utiliser les crates officielles de haut niveau (`tree-sitter` et grammaires précompilées) dès le jalon M1. |
| **Complexité de l'analyse de flux (taint analysis) en Bash** | Élevée | Moyen | Définir un périmètre strict pour la propagation intra-script, assumer les limites documentées (`trap`, `source`) et dégrader la confiance (`LOW`). |
| **Faux positifs sur l'entropie et les secrets** | Élevée | Élevé | Combiner plusieurs critères (noms de variables, motifs de tokens connus, entropie) et calibrer les seuils avec des jeux d'essai unitaires. |
| **Gestion des chemins et des wildcards complexes dans `COPY`** | Moyenne | Moyen | Traiter les cas statiques directs en priorité et émettre un avertissement explicite avec confiance réduite sur les cas ambigus. |
