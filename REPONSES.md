# TP3 — Réponses

**Binôme :** Louey Lahwel (travail réalisé seul)

> Mesures relevées en exécutant la stack et les tests dans Docker (sauf Q5, mesurée avec Node en
> local sur le commit de départ). Captures dans `captures/`.

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
de lignes (19 tests réussis sur 19, mesure confirmée dans Docker avec `docker compose run --rm unit`).
Les deux tests exécutent la branche « liste vide » (`filterCours([], …)`) et la combinaison recherche +
niveau. Il reste 85,71 % de fonctions couvertes (6 sur 7) : la fonction jamais appelée est le callback
`(topic) => normalize(topic).includes(needle)` de la ligne 48, qui cherche dans les mots-clés. Dans tous
les tests, le terme cherché est trouvé dans le titre, donc le `||` s'arrête avant d'examiner les
mots-clés. Un test cherchant un mot présent seulement dans `topics` le couvrirait.

**Q7.** La couverture de lignes indique quelles lignes ont été exécutées ; la couverture de branches
indique quelles issues de chaque condition ont été testées (vrai et faux). Un code couvert à 100 % n'est
pas forcément sans bug : on peut exécuter une ligne sans vérifier le résultat. Exemple du TP : avant
correction, la ligne `return \`${hours} h ${rest}\`` était exécutée par le test à 90 minutes, donc
« couverte », alors que le cas `60` était faux.

## Partie 3 — Tests de composant

**Q8.** `CoursCard.test.tsx` vérifie le **rendu** du composant : le titre est un lien vers
`/cours/abc123`, la durée formatée (« 1 h 30 ») et le libellé du niveau apparaissent dans la page, la
liste des mots-clés est affichée (ou absente s'il n'y en a pas). Un test unitaire de `formatDuration`
vérifie seulement la chaîne renvoyée, pas qu'elle soit bien affichée au bon endroit. Le navigateur est
simulé par **jsdom** (`environment: "jsdom"`), configuré dans `frontend/vitest.config.mts` ; Testing
Library (`render`, `screen`) s'appuie dessus.

**Q9.** `getByRole` et `getByText` cherchent les éléments comme un utilisateur (ou un lecteur
d'écran) les perçoit : un lien avec tel nom, un texte visible. Une classe CSS est un détail
d'implémentation : on peut la renommer sans changer le comportement, et le test casserait pour rien.
Chercher par rôle vérifie aussi au passage l'accessibilité (rôle et nom accessibles corrects).

**Q10.** `app/page.tsx` est un composant serveur asynchrone qui appelle Strapi au moment de la
requête. Le tester en composant demanderait de simuler Next.js et le réseau. Il est couvert par les
tests **E2E**, qui chargent la vraie page avec le vrai Strapi (et les appels à l'API par les tests
d'intégration).

## Partie 4 — Tests d'intégration

**Q11.** Les tests d'intégration vérifient le contrat entre le front et l'API Strapi réelle (format de
réponse, appels HTTP), ce qu'aucun test unitaire ou de composant ne fait puisqu'ils n'appellent pas
Strapi.

**Q12.** Strapi arrêté (`docker compose run --rm --no-deps integration`) : les **4 tests échouent**,
chacun avec `Error: Test timed out in 5000ms` (l'appel à `http://strapi:1337` ne reçoit jamais de
réponse), en 20,7 s au total. Les tests unitaires, relancés juste après, passent toujours (19/19)
parce qu'ils n'appellent ni le réseau ni Strapi. `--no-deps` empêche `docker compose run` de redémarrer les services déclarés dans
`depends_on` (ici Strapi et PostgreSQL).

**Q13.** Non. Autres causes possibles : Strapi non démarré ou pas encore prêt, permissions du rôle Public
non accordées, données de seed absentes ou différentes, variable `STRAPI_URL` mal configurée.

## Partie 5 — Tests de bout en bout

**Q14.** Le scénario **« la fiche du cours d'introduction affiche sa durée »** échoue (3 réussis,
1 échoué). Erreur : `expect(locator).toBeVisible() failed`, le texte exact `1 h` est introuvable dans
`main` après 5 s. La capture d'écran (`captures/playwright-echec.png`) montre la fiche avec
**« Durée 1 h 0 »** ; la trace montre que la navigation, le clic et le titre sont bons, et que seule la
dernière assertion échoue. C'est le même bug que Q3 (`formatDuration(60)` renvoie `1 h 0`). Il échoue
alors que le code est corrigé parce que le conteneur `frontend` tourne avec une **image construite
avant la correction** : tant qu'on ne reconstruit pas (`docker compose up -d --build frontend`), le site
déployé contient l'ancien code. Après reconstruction : 4/4.

**Q15.** Le test **unitaire** trouve la cause le plus vite : il désigne la fonction, la valeur
d'entrée et la valeur obtenue, en quelques millisecondes. Le test **E2E** prouve que l'utilisateur est
touché : il voit réellement « 1 h 0 » sur la fiche du cours.

**Q16.** Mesures (dans Docker, après correction) :

| Niveau | Nombre de tests | Durée d'exécution | Besoin de la stack ? |
|---|---|---|---|
| Unitaire + composant | 19 | 3,1 s | Non |
| Intégration | 4 | 0,9 s (Strapi démarré) | Oui (Strapi + PostgreSQL) |
| E2E | 4 | 4,1 s (9,2 s avec l'échec et ses 5 s d'attente) | Oui (toute la stack + navigateur) |

**Q17.** Un test « flaky » réussit ou échoue d'une exécution à l'autre sans que le code change (délai
réseau, élément pas encore affiché, ordre d'exécution). Ici, Playwright le limite par l'**attente
automatique** des éléments avant d'agir, et par des assertions qui **réessaient** jusqu'au délai
(`toHaveCount(6)`, `toBeVisible()`, `toHaveURL(...)`) au lieu de lire l'état une seule fois. Les
sélecteurs par rôle (`getByRole`) sont aussi plus stables que des sélecteurs CSS. `retries: 0` dans
`playwright.config.ts` évite en plus de masquer un test instable.

## Partie 6 — SonarQube

**Q18.** `frontend/sonar-project.properties` complété :

```properties
sonar.projectKey=tp3-cours
sonar.projectName=TP3 - Cours DevOps
sonar.sources=app,components,lib
sonar.tests=tests
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.exclusions=node_modules/**,.next/**,coverage/**,playwright-report/**,test-results/**
sonar.coverage.exclusions=app/**,lib/api.ts,components/CoursList.tsx,components/DevOpsLoop.tsx,*.config.*
sonar.sourceEncoding=UTF-8
sonar.qualitygate.wait=true
```

- `sonar.projectKey=tp3-cours` : identifiant unique du projet dans SonarQube (le gate y est associé).
- `sonar.sources=app,components,lib` : les trois dossiers du code applicatif analysé.
- `sonar.tests=tests` : le code des tests, analysé à part (il n'est pas compté comme code à couvrir).
- `sonar.javascript.lcov.reportPaths=coverage/lcov.info` : le rapport de couverture écrit par Vitest,
  relatif au dossier `frontend/` (monté dans `/usr/src` pour le scanner).

On exclut `app/**` et `lib/api.ts` du calcul de couverture parce que ce code n'est **pas exécuté par
les tests Vitest mesurés** : les pages Next.js (composants serveur) sont couvertes par les tests E2E,
et `api.ts` par les tests d'intégration, qui ne produisent pas de rapport lcov. Les inclure ferait
baisser la couverture sans refléter un vrai manque de tests.

**Q19.** Analyse avec le Quality Gate TP3 : **statut Failed (rouge)**, le scanner s'arrête avec
`QUALITY GATE STATUS: FAILED` (code de sortie 3). Condition en échec : **Vulnerabilities = 1, est
supérieur à 0** (sur le code global). Les autres conditions passent (couverture 97,8 % ≥ 90 ; pas de
nouveau code). Mesures : **0 bug, 1 vulnérabilité, 0 security hotspot, 14 code smells, couverture
97,8 %, duplication 5,6 %** (2 blocs dupliqués, 778 lignes de code). Capture : `captures/sonar-rouge.png`.

**Q20.**
- **Bug** : code qui fait probablement quelque chose de faux à l'exécution. Notre rapport : 0 bug.
- **Vulnérabilité** : faiblesse exploitable par un attaquant, à corriger. Exemple : le mot de passe
  écrit en dur dans `lib/api.ts` (règle `typescript:S2068`).
- **Security hotspot** : code sensible pour la sécurité qu'un humain doit **revoir** pour décider s'il
  pose problème (ce n'est pas forcément une faille). Notre rapport : 0.
- **Code smell** : problème de maintenabilité, le code marche mais il est plus difficile à lire ou à
  faire évoluer. Exemple : `components/CoursList.tsx`, complexité cognitive 16 pour 15 autorisés
  (`typescript:S3776`), ou l'import inutilisé de `useEffect` (`typescript:S1128`).

**Q21.** Règle **`typescript:S2068` — mot de passe écrit en dur** (« Review this potentially
hard-coded password », sévérité Major), ligne 4 de `lib/api.ts` : `const API_PASSWORD =
"Tp3-Cours-2026!"`. Risque : toute personne qui lit le code (dépôt, image Docker, bundle) récupère le
mot de passe, et il ne peut pas changer sans modifier le code.
Après correction, le mot de passe vient de l'**environnement** : `.env` (non commité) définit
`STRAPI_CLIENT_PASSWORD` → Docker Compose le substitue dans `docker-compose.yml`
(`STRAPI_CLIENT_PASSWORD: ${STRAPI_CLIENT_PASSWORD}`) → le conteneur `frontend` reçoit la variable →
le code la lit avec `process.env.STRAPI_CLIENT_PASSWORD` à l'exécution. `.env.example` ne contient
qu'une valeur d'exemple (`change-moi`).
Même dans un dépôt privé, un secret dans le code est dangereux : il reste **dans tout l'historique
Git**, il est copié dans chaque clone, fork, sauvegarde et image construite, il est visible par toutes
les personnes qui ont accès au dépôt (y compris plus tard), et le rendre public par erreur suffit à le
divulguer. Changer le secret oblige à modifier le code et à redéployer.

**Q22.** La nouvelle version de `CoursList.tsx` fait disparaître **5 code smells** (14 → 9) :

| Règle | Sévérité | Problème | Ce qui le règle |
|---|---|---|---|
| `typescript:S1128` | Minor | import inutilisé de `useEffect` | l'import ne garde que `useState` |
| `typescript:S1128` | Minor | import inutilisé de `sortBySession` | retiré de l'import `@/lib/cours` |
| `typescript:S3776` | Critical | complexité cognitive de `describeResults` : 16 pour 15 autorisés (`if`/`else` imbriqués) | la logique est découpée : `describeEmpty` traite le cas « 0 résultat », `describeResults` n'a plus qu'un ternaire |
| `typescript:S6660` | Major | un `if` seul dans un `else` (ligne 17) | plus de `else { if … }` : retours anticipés et ternaires |
| `typescript:S6660` | Major | un `if` seul dans un `else` (ligne 24) | idem ; le cas `count === 1` disparaît car `${count} cours` donne aussi « 1 cours » |

Le nouveau code calcule une seule fois la partie « de niveau … » (`niveau`), ce qui supprime aussi la
répétition des messages.

**Q23.** Le code dupliqué est le bloc de métadonnées d'un cours : la liste `<dl className="cours-meta">`
(Séance, Durée, Niveau) et la liste des mots-clés `<ul className="topics">`, présentes à l'identique
dans `components/CoursCard.tsx` (lignes 12-35) et `app/cours/[documentId]/page.tsx` (lignes 21-44),
24 lignes chacune. Pour le supprimer : extraire ce bloc dans un composant réutilisable (par exemple
`components/CoursMeta.tsx` qui reçoit `cours` en prop) et l'utiliser dans les deux fichiers, avec un
test de composant dédié.

**Q24.** La dette technique est l'**estimation du temps nécessaire pour corriger tous les code smells**
(métrique `sqale_index`). SonarQube l'exprime en durée (minutes, heures, jours de 8 h) et en note de
maintenabilité (A à E selon le ratio dette / temps de développement estimé). Ici : **63 min avant**
les corrections, **45 min après** (les 5 code smells corrigés valaient 18 min).

**Q25.** Le **New Code** est le code ajouté ou modifié depuis une référence (par défaut la version
précédente du projet, ou une branche, un nombre de jours…). Conditions du gate TP3 sur le nouveau code
(ajoutées d'office par SonarQube) : couverture du nouveau code < 80 %, lignes dupliquées dans le
nouveau code > 3 %, nouveaux problèmes (issues) > 0, security hotspots revus < 100 %. SonarQube
recommande de juger le nouveau code (« Clean as You Code ») parce qu'on ne peut pas exiger de corriger
d'un coup toute la dette d'un vieux projet : en imposant que chaque modification soit propre, la
qualité globale s'améliore au fil du temps, et l'équipe reste responsable de ce qu'elle écrit
maintenant.

**Q26.** Avec `sonar.qualitygate.wait=true`, le scanner **attend** que SonarQube ait calculé le
Quality Gate, puis **se termine en erreur** s'il est rouge (`QUALITY GATE STATUS: FAILED`,
`EXECUTION FAILURE`, code de sortie non nul : 3 dans notre cas). Dans un pipeline CI, ce code
d'erreur **fait échouer l'étape** : le pipeline s'arrête et on peut bloquer le merge ou le
déploiement automatiquement, sans que quelqu'un doive aller regarder le tableau de bord.
Capture après correction : `captures/sonar-vert.png`.

## Partie 7 — Synthèse

**Q27.** Ordre proposé, du plus rapide et du moins coûteux au plus lent :

1. **Installation et build** (`npm ci`, `npm run build`, build des images) : inutile d'aller plus loin
   si le code ne compile pas.
2. **Tests unitaires + de composant avec couverture** (~3 s, sans stack) : retour le plus rapide, ils
   localisent précisément les erreurs.
3. **Analyse SonarQube** (avec la couverture de l'étape 2) et son **Quality Gate** : analyse
   statique, pas besoin de lancer l'application.
4. **Tests d'intégration** (démarrer Strapi + PostgreSQL) : vérifient le contrat avec l'API.
5. **Tests E2E** (toute la stack + navigateur) : les plus lents et les plus fragiles, lancés en dernier
   sur une application déjà validée par les niveaux précédents.
6. Publication de l'image / déploiement, seulement si tout est vert.

Doivent **bloquer le merge** d'une pull request : le build, les tests unitaires et de composant, le
Quality Gate SonarQube (vulnérabilités, couverture du nouveau code), les tests d'intégration et les
tests E2E des parcours critiques. Les étapes rapides (1 à 3) peuvent tourner à chaque push ; les
étapes 4 et 5 au moins sur chaque pull request.

## Captures

`captures/couverture.png`, `captures/playwright-echec.png`, `captures/sonar-rouge.png`,
`captures/sonar-vert.png`.
