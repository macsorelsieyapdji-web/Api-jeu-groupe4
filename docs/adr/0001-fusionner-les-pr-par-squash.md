\# 1. Fusionner les pull requests par squash



\- \*\*Date\*\* : 2026-10-09

\- \*\*Statut\*\* : Proposée



\## Contexte



Toute modification de ce dépôt passe par une pull request relue (une issue, une branche, une PR). GitHub propose trois méthodes de fusion, et le bouton vert propose par défaut « Create a merge commit ». Il faut une règle commune pour que l'historique de `main` reste lisible.



La première PR du projet (#4, statistiques) compte trois commits : le test, la correction, puis un retour à la ligne demandé par `ruff`. Le correcteur lit l'historique de `main`.



\## Options envisagées



1\. \*\*Merge commit.\*\* Pour : tous les commits sont conservés, l'historique est complet. Contre : les commits de travail (« ajouter le retour à la ligne final ») et un commit de fusion s'ajoutent sur `main`.

2\. \*\*Squash and merge.\*\* Pour : un commit par PR relue, dont le titre devient le message ; `git log` se lit comme une liste de changements validés. Contre : le détail des commits disparaît de `main`.

3\. \*\*Rebase and merge.\*\* Pour : historique linéaire, commits conservés, sans commit de fusion. Contre : exige des commits déjà propres, et réécrit leurs identifiants.



\## Décision



Nous retenons le squash : un commit sur `main` correspond à une PR relue.



\## Conséquences



\- `git log` sur `main` est court et lisible : une ligne par changement validé.

\- Le titre de la PR devient le message du commit : il faut le soigner.

\- Le détail des commits disparaît de `main` : l'ordre « test avant correction » de la PR #4 n'est visible que dans la PR, pas dans l'historique de `main`.

\- À revoir si une PR doit garder des commits atomiques (plusieurs intentions, ou besoin de `git bisect` fin) : on choisirait alors le rebase pour cette PR.

