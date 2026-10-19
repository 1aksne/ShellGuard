# Guide de Contribution et Workflow Git

Bienvenue dans le projet! Pour assurer une collaboration fluide et garder un
historique propre, nous utilisons un workflow basé sur les branches et les Pull
Requests (PR).

---

## 1. Les Branches

- **`main` :** C'est la branche de production. Le code y est toujours stable,
testé et fonctionnel. Personne ne push directement sur `main`.
- **branche de travail :** Pour chaque nouvelle tâche, créer une branche à
partir de `main`.
    - Format : `type/description-courte`
    - Exemple : `feat/add-authentification`, `fix/bug-display-button`,
    `docs/update-readme`

---

## 2. Le Workflow de Base

1. **Mettre à jour son dépot local :**
```bash
git checkout main
git pull origin main
```
2. **Créer une branche pour sa tâche :**
```bash
git checkout -b feat/nom-de-la-fonctionnalité
```
3. **Travailler et faire des commits régulier.**
4. **Pousser sa branche sur le server :**
```bash
git push -u origin feat/nom-de-la-fonctionnalité
```
5. **Ouvrir une PR** sur Github vers la branche `main`.

---

## 3. Convention de Commits

Nous utilisons la norme "karma" pour générer des historiques lisibles. Chaque
message de commit doit commit par un préfixe :

- `feat` : Nouvelle fonctionnalité.
- `fix` : Correction d'un bug.
- `docs` : Ajout ou modification de documentation.
- `style` : Formatage du code (espace, point-virgules, etc sans changer la
logique).
- `refactor` : Modification du code qui n'ajoute ni fonctionnalité ni
correction de bug
- `test` : Ajout ou modification de tests.

**Forme d'un commit type :** `prefixe(scope) : courte description de ce qui a
été ajouter ou modifier`

---

## 4. Validation (Code Review)

Avant d'être merge dans `main`, chaque PR doit :

- Etre éprouvée par au moins un autre developpeur.
- Passer les tests automatiques (si CI/CD en place).
- Etre exempte de conflits avec la branche `main`.
    


