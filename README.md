# ShellGuard

**ShellGuard** est un analyseur statique de sécurité conçu pour détecter les
risques et les vulnérabilités à l'intersection des `Dockerfiles` et des scripts
Shell associés, sans jamais avoir à les exécuter.

---

## Contexte et Objectifs

Lors de la construction d'images Docker, les outils d'analyse statique
habituels (comme ShellCheck ou Hadolint) examinent les fichiers
individuellement. ShellGuard innove en analysant le processus de *build* dans
son ensemble.

L'objectif est d'indentifier les risques de sécurité qui émergent de
l'interaction entre les instructions Docker (comme `COPY`, `RUN`, `USER`, `ENV`)
et l'éxecution des scripts Shell dans l'image.

### Ce que fais ShellGuard :

- **Découverte automatique** des `Dockerfiles` et scripts Shell (`.sh`, `.bash`,
shebangs) dans un projet.
- **Analyse statique sécurisé :** aucun script n'est éxecuté. L'analyse repose
sur le parsing de l'AST via `Tree-sitter`.
- **Moteur de détection avancé :** implémente 14 règles de sécurité ciblant
l'exposition de secrets, l'élévation de privillège ou l'exécution de commandes
dangereuse.
- **Suivi de contexte :** relie les chemins `COPY`/`ADD` à leur utilisation
(`USER`) et des variables sensibles (`ARG`, `ENV`) par étape de build.
- **Rapport multi-formats :** génère des rapports clairs pour le terminal, ou
structurés en JSON ou SARIF.

---

## Installation

1. Assurez-vous d'avoir [Rust et Cargo](https://rustup.rs/) installé sur votre
système.
2. Clonez le dépôt et compilez le projet :

```bash
git clone [https://github.com/1aksne/shellguard.git](https://github.com/1aksne/shellguard.git)
cd ShellGuard
cargo build --release
```

Le binaire exécutable statique sera généré dans `target/release/shellguard`.

---

## Utilisation

### En local (Mode développeur)

Pour lancer une analyse sur un répertoire contenant vos fichiers Docker et
scripts:

```bash
shellguard scan .
```

Cette commande affiche les alertes de sécurité dans votre terminal avec leur
niveau de confiance (HIGH, MEDIUM, LOW) et la ligne concernée.

### En intégration Continue (Mode DevOps)

ShellGuard est pensé s'intégrer nativement dans vos pipelines CI/CD en tant
qu'étape bloquante.

Il configurera un code de sortie d'erreur (`1`) si une vulnérabilité est
détecté selon votre politique 

---

## Contribution

Nous suivons le cycle de développement documenté dans notre cahier des charges.
N'hésitez pas à consulté notre fichier `CONTRIBUTING.md` pour connaître notre
workflow basé sur les branches et les Pull Requests.
