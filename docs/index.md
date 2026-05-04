# Algorithmes & Structures de données en PHP

Bienvenue dans ce tutoriel complet sur les algorithmes et les structures de données, **entièrement en PHP 8.x**.

---

## Objectif

Ce tutoriel vous emmène de zéro à expert : comprendre la complexité algorithmique, maîtriser les structures de données fondamentales, et appliquer les grands paradigmes algorithmiques sur des problèmes concrets.

Tous les exemples sont écrits en **PHP 8.2+** avec le typage strict activé.

---

## Plan du tutoriel

```
Partie 1 — Fondations
├── Chapitre 1 : Complexité algorithmique (Big O)
├── Chapitre 2 : Tableaux & listes dynamiques
├── Chapitre 3 : Piles, files & deques
└── Chapitre 4 : Listes chaînées

Partie 2 — Structures avancées
├── Chapitre 5 : Tables de hachage
├── Chapitre 6 : Arbres binaires & BST
├── Chapitre 7 : Tas & files de priorité
└── Chapitre 8 : Graphes

Partie 3 — Algorithmes
├── Chapitre 9  : Algorithmes de tri
├── Chapitre 10 : Algorithmes de recherche
├── Chapitre 11 : Programmation dynamique
└── Chapitre 12 : Gloutons & Diviser pour régner
```

---

## Prérequis

- PHP 8.2+ installé (`php -v`)
- Connaissance des bases du langage (variables, fonctions, classes)
- Aucune bibliothèque externe n'est requise — la librairie standard PHP suffit

---

## Comment lire ce tutoriel

!!! tip "Conseil de lecture"
    Chaque chapitre suit la même structure :

    1. **Concept** — explication claire avec schéma
    2. **Implémentation PHP** — code complet et commenté
    3. **Analyse de complexité** — tableau temps / espace
    4. **Exercices** — pour consolider les acquis

!!! info "Convention de code"
    Tous les fichiers commencent par `declare(strict_types=1);` et utilisent le typage PHP 8 natif (`int`, `string`, `array`, `mixed`, génériques via PHPDoc).

---

## Cheat sheet Big O

| Complexité   | Nom             | Exemple typique              |
|-------------|-----------------|------------------------------|
| O(1)        | Constante       | Accès tableau par indice     |
| O(log n)    | Logarithmique   | Recherche binaire            |
| O(n)        | Linéaire        | Parcours de liste            |
| O(n log n)  | Log-linéaire    | Tri fusion, tri rapide       |
| O(n²)       | Quadratique     | Tri à bulles, tri insertion  |
| O(2ⁿ)       | Exponentielle   | Fibonacci naïf, sous-ensembles |
| O(n!)       | Factorielle     | Permutations, TSP brute force |

---

Bonne lecture !
