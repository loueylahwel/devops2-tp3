# TP3 — Réponses

**Binôme :** NOM Prénom — NOM Prénom

> Brouillon. Les mesures ci-dessous viennent d'un lancement **local** (Node, sans Docker).
> Les parties marquées **TODO** demandent Docker, SonarQube ou des captures d'écran sur votre machine.

## Partie 1 — Pyramide des tests

**Q1.** `tests/unit` : tests unitaires, Vitest, pas de stack (fonctions pures de `lib/cours.ts`).
`tests/component` : tests de composant, Vitest + Testing Library dans jsdom, pas de stack.
`tests/integration` : tests d'intégration, Vitest, Strapi doit tourner (appels à l'API).
`tests/e2e` : tests de bout en bout, Playwright dans un vrai navigateur, toute la stack doit tourner.

**Q2.** La base est large parce que les tests unitaires sont rapides, peu coûteux et faciles à
diagnostiquer. Le sommet est étroit parce que les tests E2E sont lents, fragiles et coûteux à
maintenir ; on n'en écrit donc que pour les parcours critiques.

## Partie 2 — Tests unitaires

**Q3.** Avant correction : 9 tests dans `cours.test.ts`, 1 échoue. Message : `AssertionError: expected
'1 h 0' to be '1 h'`, `Expected: "1 h"`, `Received: "1 h 0"`. `Expected` est la valeur attendue par le
test, `Received` la valeur réellement renvoyée par la fonction.

**Q4.** `formatDuration` affiche toujours les minutes après les heures : pour 60, `rest` vaut 0 et la
fonction renvoie `1 h 0`. La correction ajoute une condition : si `rest === 0`, on n'affiche que
`${hours} h`.

**Q5.** Avant ajout des tests (commit de base, mesuré localement) : `lib/cours.ts` a 88,23 % des
instructions, 75 % des branches, 85,71 % des fonctions et 93,33 % des lignes. La branche non couverte
est à la **ligne 48** du fichier d'origine : c'est la branche « heure pleine » de `formatDuration`, celle
du test qui échoue.

**Q6.** Après correction et ajout des deux tests : `cours.ts` passe à **100 % de branches** et 100 %
de lignes (mesure locale : 19 tests réussis sur 19). Les deux tests exécutent la branche « liste vide »
(`filterCours([], …)`) et la combinaison recherche + niveau. Il reste 85,71 % de fonctions couvertes
pour `cours.ts` : une fonction n'est pas appelée par les tests de ce module (à vérifier dans le rapport
HTML `frontend/coverage/index.html`).

**Q7.** La couverture de lignes indique quelles lignes ont été exécutées ; la couverture de branches
indique quelles issues de chaque condition ont été testées (vrai et faux). Un code couvert à 100 % n'est
pas forcément sans bug : on peut exécuter une ligne sans vérifier le résultat. Exemple du TP : avant
correction, la ligne `return \`${hours} h ${rest}\`` était exécutée par le test à 90 minutes, donc
« couverte », alors que le cas `60` était faux.

## Partie 3 — Tests de composant

**Q8.** TODO — à compléter après lecture de `tests/component/CoursCard.test.tsx`. Outil : Testing Library
dans jsdom (environnement configuré dans `vitest.config.mts`).

**Q9.** TODO. Réponse attendue : `getByRole` et `getByText` testent ce que voit et lit l'utilisateur ; une
classe CSS est un détail d'implémentation qui change sans que le comportement change.

**Q10.** TODO. Réponse attendue : `app/page.tsx` est un composant serveur qui appelle Strapi ; il est
couvert par les tests d'intégration et E2E.

## Partie 4 — Tests d'intégration

**Q11.** Les tests d'intégration vérifient le contrat entre le front et l'API Strapi réelle (format de
réponse, appels HTTP), ce qu'aucun test unitaire ou de composant ne fait puisqu'ils n'appellent pas
Strapi.

**Q12.** Strapi arrêté (mesuré localement) : les 4 tests d'intégration échouent avec
`ECONNREFUSED 127.0.0.1:1337` et `::1:1337`. Les tests unitaires passent toujours parce qu'ils n'ont
pas besoin du réseau. `--no-deps` empêche `docker compose run` de redémarrer les services déclarés dans
`depends_on` (ici Strapi et PostgreSQL).

**Q13.** Non. Autres causes possibles : Strapi non démarré ou pas encore prêt, permissions du rôle Public
non accordées, données de seed absentes ou différentes, variable `STRAPI_URL` mal configurée.

## Partie 5 — Tests de bout en bout

**Q14.** TODO — à compléter après `docker compose run --rm e2e` (Docker requis). Le test attendu échoue
parce que le conteneur `frontend` a été construit avant la correction de `formatDuration`.

**Q15.** TODO. Réponse attendue : le test unitaire trouve la cause (la fonction et la ligne) le plus
vite ; le test E2E prouve que l'utilisateur voit le bug.

**Q16.** TODO pour la durée E2E. Mesures locales des tests unitaires et de composant :

| Niveau | Nombre de tests | Durée d'exécution | Besoin de la stack ? |
|---|---|---|---|
| Unitaire + composant | 19 (après correction) | ~2,6 s (Vitest) | Non |
| Intégration | 4 (échouent sans Strapi) | ~0,4 s (échec immédiat) | Oui (Strapi) |
| E2E | TODO | TODO | Oui (toute la stack) |

**Q17.** TODO. Un test « flaky » réussit ou échoue de façon aléatoire. Playwright réduit ce risque avec
l'attente automatique des éléments et les assertions `expect(...)` qui réessaient jusqu'au délai.

## Partie 6 — SonarQube

**Q18.** TODO — à compléter avec votre `sonar-project.properties`.

**Q19–Q27.** TODO : résultats du Quality Gate et du tableau de bord SonarQube (Docker requis).

## Captures à faire sur votre machine

`captures/couverture.png`, `captures/playwright-echec.png`, `captures/sonar-rouge.png`,
`captures/sonar-vert.png`.
