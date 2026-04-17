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
- Pipeline : GitHub Actions
- Qualité du code : SonarCloud
- Déploiement : Docker Hub

---

## 2. Les étapes du workflow CI/CD

Le pipeline se déclenche automatiquement à chaque push ou pull request sur la branche `main`. Il est composé de 3 jobs.

### 2.1 Job `test-back` — Tests et couverture back-end

| Propriété | Valeur |
|-----------|--------|
| Outil | GitHub Actions + Maven + JaCoCo + SonarCloud |
| Machine | ubuntu-latest |
| Déclencheur | Chaque push ou pull request sur main |

**Étapes :**
1. Récupération du code source (`actions/checkout@v3`)
2. Installation de Java 11 Temurin (cohérent avec le `pom.xml`)
3. Compilation et exécution des tests unitaires (`mvn clean install`)
4. Génération du rapport de couverture JaCoCo
5. Installation de Java 17 (requis par le plugin SonarCloud 4.x)
6. Analyse statique du code via SonarCloud

> **Note :** Deux versions Java sont utilisées. Java 11 compile et teste le projet. Java 17 est ensuite installé car le plugin SonarCloud 4.x est incompatible avec Java 11.

### 2.2 Job `test-front` — Tests et couverture front-end

| Propriété | Valeur |
|-----------|--------|
| Outil | GitHub Actions + Node.js 18 + Karma |
| Machine | ubuntu-latest |
| Exécution | En parallèle du job test-back |

**Étapes :**
1. Récupération du code source
2. Installation de Node.js 18
3. Installation des dépendances npm
4. Exécution des tests Angular avec Karma en mode ChromeHeadless
5. Génération du rapport de couverture de code

### 2.3 Job `docker` — Déploiement sur Docker Hub

| Propriété | Valeur |
|-----------|--------|
| Outil | GitHub Actions + Docker Hub |
| Condition | Uniquement sur main (push) et si test-back + test-front ont réussi |
| Secrets | DOCKERHUB_USERNAME et DOCKERHUB_TOKEN stockés dans GitHub Secrets |

**Étapes :**
1. Connexion à Docker Hub via les secrets GitHub
2. Build de l'image Docker du back-end Spring Boot
3. Push de l'image `tymsydrame/bobapp-back:latest` sur Docker Hub
4. Build de l'image Docker du front-end Angular
5. Push de l'image `tymsydrame/bobapp-front:latest` sur Docker Hub

> **Principe clé :** Le déploiement Docker ne se déclenche que si tous les tests passent (`needs: [test-back, test-front]`). Il est impossible de déployer du code défaillant.

> **Mise à jour Dockerfile :** L'image `openjdk:11-jdk-slim` était obsolète. Elle a été remplacée par `eclipse-temurin:11-jre-jammy`, l'image Java 11 officielle maintenue par Adoptium.

---

## 3. KPI proposés

| KPI | Seuil recommandé | État actuel | Justification |
|-----|-----------------|-------------|---------------|
| Taux de couverture (Coverage) | 80% minimum | 38.8% — insuffisant | Garantit que la majorité du code est testée et limite les régressions |
| Nouveaux bugs bloquants (New Blocker Issues) | 0 | 0 ✅ | Aucun bug critique ne doit être introduit à chaque contribution |
| Security Hotspots traités | 100% | 3 en attente ⚠️ | Toutes les failles potentielles doivent être analysées et résolues |

Le KPI prioritaire est le taux de couverture de code. Actuellement à **38.8%**, il est très en dessous de l'objectif de 80%. Sur 45 lignes à couvrir, 28 ne sont pas testées.

---

## 4. Analyse des métriques SonarCloud

| Métrique | Valeur actuelle | Interprétation |
|----------|----------------|----------------|
| Security | A (0 open issues) | Aucune faille de sécurité critique |
| Reliability | A (0 open issues) | Aucun bug bloquant détecté |
| Maintainability | A (11 issues mineures) | Code lisible, 11 améliorations mineures |
| Coverage | 38.8% | Insuffisant — objectif KPI : 80% |
| Line Coverage | 37.8% | 28 lignes non couvertes sur 45 |
| Security Hotspots | 3 détectés | À analyser et traiter en priorité |
| Duplications | 0.0% | Aucune duplication de code |

### Interprétation

**Points positifs :**
- Security A et Reliability A : le code est sain, sans bugs bloquants ni failles critiques
- Maintainability A : le code est lisible et bien structuré
- 0% de duplication : aucune répétition de code

**Points à améliorer :**
- Coverage 38.8% : insuffisant par rapport au KPI de 80%. C'est le chantier prioritaire
- 3 Security Hotspots : à analyser manuellement, certains peuvent être des faux positifs

---

## 5. Analyse des avis utilisateurs

| Avis | Problème identifié | Priorité | Lien avec la CI/CD |
|------|--------------------|----------|-------------------|
| "Impossible de poster une suggestion, le bouton plante le navigateur" | Bug front-end sur le bouton de suggestion | Haute | Un test unitaire Angular aurait pu détecter ce comportement |
| "Un bug sur le post de vidéo remonté il y a 2 semaines, toujours présent" | Bug non corrigé — manque de réactivité | Haute | La CI/CD accélère les cycles de correction |
| "Depuis une semaine je ne reçois plus rien" | Problème de notifications ou de flux | Haute | Un test d'intégration aurait détecté cette régression |
| "J'ai supprimé ce site de mes favoris, dommage" | Perte d'un utilisateur | Critique | Évitable avec un pipeline garantissant la stabilité |

### Conclusion

La pipeline CI/CD mise en place apporte les bénéfices suivants à Bob :

- **Automatisation complète** — plus de déploiements manuels via FTP
- **Qualité garantie** — impossible de merger du code qui casse les tests
- **Visibilité totale** — SonarCloud commente automatiquement chaque Pull Request
- **Déploiement fiable** — images Docker générées et publiées automatiquement

**Prochaines actions recommandées :**
1. Augmenter le taux de couverture pour atteindre 80%
2. Analyser et traiter les 3 Security Hotspots
3. Corriger les bugs remontés dans les avis (post de suggestion, post de vidéo, notifications)