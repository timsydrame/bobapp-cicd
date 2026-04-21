# BobApp — Document d'analyse CI/CD

**Auteur :** Fatoumata Dramé  
**Formation :** Développeur Full Stack Java & Angular — OpenClassrooms  
**Projet :** P10 — Gérez un projet collaboratif en intégrant une démarche CI/CD  
**Date :** Avril 2026  
**Repository :** https://github.com/timsydrame/bobapp-cicd

---

## 1. Présentation du projet

BobApp est une application web de blagues du jour développée par Bob. Face à la croissance du nombre d'utilisateurs et aux difficultés de gestion manuelle des déploiements, une pipeline CI/CD complète a été mise en place pour automatiser les tests, l'analyse qualité et le déploiement.

**Problèmes identifiés avant la CI/CD :**
- Gestion manuelle des pull requests, tests, builds et déploiements
- Déploiements via FTP longs et sources d'erreurs
- Bugs non corrigés rapidement — utilisateurs mécontents
- Peu de contributions externes faute de processus automatisé

**Stack technique :**
- Back-end : Spring Boot (Java 11) + Maven + JaCoCo
- Front-end : Angular + Karma
- Pipeline : GitHub Actions (3 workflows séparés)
- Qualité du code : SonarCloud (back + front)
- Déploiement : Docker Hub

---

## 2. Les étapes du workflow CI/CD

Le pipeline est composé de 3 fichiers de workflow séparés dans `.github/workflows/`.
La branche `main` est protégée — tout merge nécessite une Pull Request avec les checks CI verts.

---

### 2.1 Workflow `ci-backend.yml` — CI Back-end

**Déclencheur :** push ou pull request sur `main`

| Étape | Outil | Description |
|-------|-------|-------------|
| Récupérer le code | actions/checkout@v3 | Clone le repo sur la machine virtuelle |
| Installer Java 11 | actions/setup-java@v3 | Cohérent avec le pom.xml du projet |
| Tests + JaCoCo | mvn clean install | Compile, teste et génère le rapport de couverture |
| Upload coverage | actions/upload-artifact@v4 | Rend le rapport JaCoCo disponible comme artefact |
| Installer Java 17 | actions/setup-java@v3 | Requis par le plugin SonarCloud 4.x |
| Analyse SonarCloud | mvn sonar:sonar | Analyse statique du code Java |

> **Note :** Deux versions Java sont utilisées. Java 11 compile et teste (cohérent avec pom.xml). Java 17 est requis par le plugin SonarCloud 4.x.

---

### 2.2 Workflow `ci-frontend.yml` — CI Front-end

**Déclencheur :** push ou pull request sur `main`

| Étape | Outil | Description |
|-------|-------|-------------|
| Récupérer le code | actions/checkout@v3 | Clone le repo |
| Installer Node.js 18 | actions/setup-node@v3 | Environnement Angular |
| Installer dépendances | npm install | Installation des packages |
| Tests + couverture | Karma + ChromeHeadless | Tests unitaires Angular avec rapport lcov |
| Upload coverage | actions/upload-artifact@v4 | Rend le rapport Karma disponible |
| Analyse SonarCloud | sonarcloud-github-action@v2 | Analyse qualité du code TypeScript/Angular |

> **Configuration SonarCloud front :** un fichier `sonar-project.properties` dans le dossier `front` définit les paramètres d'analyse (sources, tests, rapport lcov).

---

### 2.3 Workflow `cd.yml` — CD Déploiement

**Déclencheur :** push ou pull request sur `main`

| Job | Condition | Description |
|-----|-----------|-------------|
| `build` | Sur toute PR et push | Build les images Docker back et front (sans push) |
| `push` | Uniquement sur push vers main | Push les images sur Docker Hub |

> **Principe clé :**
> - Sur une PR → build uniquement. Un build raté bloque le merge.
> - Sur un merge vers main → push sur Docker Hub avec cache Docker (type=gha) pour optimiser les temps de build.

> **Sécurité :** les credentials Docker Hub sont stockés en GitHub Secrets (DOCKERHUB_USERNAME, DOCKERHUB_TOKEN).

---

### 2.4 Branch protection sur `main`

La branche `main` est protégée avec les règles suivantes :
- Tout commit doit passer par une Pull Request
- Les checks suivants doivent être verts avant le merge :
  - `test-back` (CI Backend)
  - `test-front` (CI Frontend)
  - `build` (CD Pipeline)

---

## 3. KPI proposés

| KPI | Seuil recommandé | État actuel | Justification |
|-----|-----------------|-------------|---------------|
| Taux de couverture back (Coverage) | 80% minimum | 38.8% — insuffisant | Garantit que la majorité du code back est testée |
| Taux de couverture front (Coverage) | 80% minimum | 66.7% — insuffisant | Garantit que la majorité du code front est testée |
| Nouveaux bugs bloquants (New Blocker Issues) | 0 | 0 ✅ | Aucun bug critique ne doit être introduit |
| Security Hotspots back traités | 100% | 3 en attente ⚠️ | Failles potentielles à analyser côté Java |
| Security Hotspots front traités | 100% | 2 en attente ⚠️ | Failles potentielles à analyser côté Angular |

**Comment configurer les KPI dans SonarCloud :**
- Aller sur SonarCloud → projet → **Quality Gate**
- Créer un nouveau Quality Gate personnalisé
- Ajouter les conditions : Coverage >= 80%, New Blocker Issues = 0
- Assigner ce Quality Gate au projet

---

## 4. Analyse des métriques SonarCloud

### Back-end (Java / Spring Boot)

| Métrique | Valeur actuelle | Interprétation |
|----------|----------------|----------------|
| Security | A (0 issues) | Aucune faille critique |
| Reliability | A (0 issues) | Aucun bug bloquant |
| Maintainability | A (11 issues mineures) | Code lisible et maintenable |
| Coverage | 38.8% | Insuffisant — objectif : 80% |
| Line Coverage | 37.8% | 28 lignes non couvertes sur 45 |
| Security Hotspots | 3 détectés | À analyser et traiter |
| Duplications | 0.0% | Aucune duplication |

### Front-end (Angular / TypeScript)

| Métrique | Valeur actuelle | Interprétation |
|----------|----------------|----------------|
| Security Hotspots | 2 détectés | À analyser et traiter |
| Coverage | 66.7% | Insuffisant — objectif : 80% |
| Duplications | 0.0% | Aucune duplication |

---

## 5. Analyse des avis utilisateurs

| Avis | Problème identifié | Priorité | Solution proposée |
|------|--------------------|----------|-------------------|
| "Impossible de poster une suggestion, le bouton plante" | Bug front-end sur le bouton de suggestion | Haute | Ajouter un test unitaire Angular sur ce composant + corriger le bug |
| "Un bug sur le post de vidéo remonté il y a 2 semaines, toujours présent" | Bug non corrigé — processus trop lent | Haute | La CI/CD accélère les cycles de correction et détecte les régressions |
| "Depuis une semaine je ne reçois plus rien" | Problème de notifications ou de flux | Haute | Ajouter un test d'intégration sur le flux de données |
| "J'ai supprimé ce site de mes favoris, dommage" | Perte d'utilisateur due à la mauvaise qualité | Critique | La pipeline CI/CD garantit la stabilité avant chaque déploiement |

---

## 6. Conclusion

**Ce que la CI/CD apporte à Bob :**
- Automatisation complète — plus de déploiements manuels via FTP
- Qualité garantie — impossible de merger du code qui casse les tests
- Analyse automatique du back ET du front avec SonarCloud
- Images Docker générées et publiées automatiquement
- Branche main protégée — tout code est validé avant merge

**Prochaines actions recommandées :**
1. Augmenter la couverture back de 38.8% à 80%
2. Augmenter la couverture front de 66.7% à 80%
3. Analyser et traiter les 3 Security Hotspots back
4. Analyser et traiter les 2 Security Hotspots front
5. Corriger les bugs remontés dans les avis utilisateurs