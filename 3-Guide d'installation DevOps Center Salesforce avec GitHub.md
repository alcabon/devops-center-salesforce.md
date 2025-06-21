# Guide d'installation DevOps Center Salesforce avec GitHub

## Table des matières
1. [Prérequis](#prérequis)
2. [Configuration initiale](#configuration-initiale)
3. [Connexion GitHub](#connexion-github)
4. [Configuration du pipeline](#configuration-du-pipeline)
5. [Premier déploiement](#premier-déploiement)
6. [Bonnes pratiques](#bonnes-pratiques)
7. [Dépannage](#dépannage)

## Prérequis

### Licences et permissions requises
- **Licence DevOps Center** activée sur votre org Salesforce
- **Permissions utilisateur** :
  - DevOps Center Admin
  - Modify All Data
  - Manage Users
- **Accès GitHub** :
  - Compte GitHub avec droits d'administration sur le repository
  - Repository GitHub configuré (public ou privé)

### Environnements Salesforce
- **Org de production** (ou Developer Edition)
- **Org(s) de développement/test** (Sandbox ou Developer Edition)
- Connexions entre les orgs configurées

## Configuration initiale

### 1. Activation de DevOps Center

```bash
# Dans Setup de votre org de production
Setup → DevOps Center → Get Started
```

1. Naviguer vers **Setup** → **DevOps Center**
2. Cliquer sur **Get Started**
3. Accepter les termes et conditions
4. Attendre l'activation (peut prendre quelques minutes)

### 2. Attribution des permissions

```sql
-- Créer un Permission Set pour DevOps Center
Permission Set Name: DevOps_Center_User
Permissions:
- DevOps Center Admin: Checked
- Use DevOps Center: Checked
```

1. **Setup** → **Permission Sets** → **New**
2. Nom : `DevOps_Center_User`
3. Ajouter les permissions :
   - `Use DevOps Center`
   - `DevOps Center Admin`
4. Assigner aux utilisateurs concernés

### 3. Configuration des Connected Apps

#### Création de la Connected App pour DevOps Center
```javascript
// Configuration OAuth pour GitHub + App Launcher
Name: DevOps Center GitHub Integration
Contact Email: admin@yourcompany.com
Callback URL: https://yourinstance.salesforce.com/services/authcallback/DevOpsCenter
Start URL: /sf_devops/DevOpsCenter.app
Scopes:
- Access and manage your data (api)
- Perform requests on your behalf at any time (refresh_token, offline_access)
```

#### ⚠️ Étapes critiques souvent manquées :

**Étape 1 : Créer la Connected App**
1. **Setup** → **App Manager** → **New Connected App**
2. Remplir les informations de base
3. **⚠️ IMPORTANT** : Ajouter la Start URL : `/sf_devops/DevOpsCenter.app`
4. Configurer OAuth comme ci-dessus
5. Sauvegarder

**Étape 2 : Gérer les profils (ÉTAPE CRITIQUE)**
1. **Setup** → **App Manager** 
2. Trouver votre Connected App DevOps Center
3. Cliquer sur **Manage** (dropdown à droite)
4. Cliquer sur **Manage Profiles**
5. **⚠️ SÉLECTIONNER LES PROFILS** qui doivent accéder à DevOps Center
6. Sauvegarder

**Étape 3 : Configurer les Permission Sets**
1. Dans la même page **Manage Connected Apps**
2. Cliquer sur **Manage Permission Sets**
3. **⚠️ SÉLECTIONNER** `sf_devops_NamedCredentials`
4. Sauvegarder

**Étape 4 : Vérification d'accès**
- Rafraîchir le navigateur (F5)
- Vérifier l'App Launcher
- Si toujours absent, accès direct via URL : `https://votre-domaine.salesforce.com/sf_devops/DevOpsCenter.app`

## Connexion GitHub

### 1. Préparation du repository GitHub

```bash
# Structure recommandée du repository
your-repo/
├── force-app/
│   └── main/
│       └── default/
│           ├── classes/
│           ├── triggers/
│           ├── flows/
│           └── objects/
├── sfdx-project.json
├── .gitignore
└── README.md
```

### 2. Configuration du fichier sfdx-project.json

```json
{
    "packageDirectories": [
        {
            "path": "force-app",
            "default": true,
            "package": "YourPackageName",
            "versionName": "ver 1.0",
            "versionNumber": "1.0.0.NEXT"
        }
    ],
    "name": "YourProjectName",
    "namespace": "",
    "sfdcLoginUrl": "https://login.salesforce.com",
    "sourceApiVersion": "60.0"
}
```

### 3. Configuration du .gitignore

```gitignore
# Salesforce specific
.sfdx/
.localdevserver/
deploy/
.vscode/settings.json

# OS generated files
.DS_Store
Thumbs.db

# Logs
*.log

# Dependencies
node_modules/
```

### 4. Connexion GitHub dans DevOps Center

#### Pour les repositories personnels
1. Ouvrir **DevOps Center** depuis l'App Launcher
2. Cliquer sur **Connect to GitHub**
3. Sélectionner **GitHub.com** ou **GitHub Enterprise**
4. Autoriser l'accès à votre compte GitHub
5. Sélectionner le repository à connecter

#### Pour les repositories d'organisation (cas complexe)
```bash
# Problème courant : "Resource protected by organization SAML enforcement"
# Solution en 6 étapes
```

1. **Demander l'accès dans GitHub** :
   - Aller dans l'organisation GitHub
   - Cliquer sur "Request access" si nécessaire
   - Attendre l'approbation du propriétaire

2. **Configurer l'accès third-party dans l'organisation** :
   - Organisation Settings → Third-party access
   - Trouver "Salesforce DevOps Center" 
   - Cliquer "Grant" ou "Approve"

3. **Dans Salesforce DevOps Center** :
   - Profile Icon → Settings
   - Authentication Settings for External Systems
   - Supprimer "DevOps Center GitHub" existant

4. **Reconnecter GitHub** :
   - Retourner dans DevOps Center
   - Créer nouveau projet
   - Se reconnecter avec GitHub (nouvelle authentification)

5. **Vérifier les permissions** :
   - Repo access : Read/Write
   - Organization access : Read
   - Token permissions si nécessaire

6. **Alternative - Token d'accès personnel** :
   ```bash
   GitHub → Settings → Developer settings → Personal access tokens
   Permissions requises :
   - repo (full control)
   - admin:org (read only)
   - workflow (si GitHub Actions utilisé)
   ```

## Configuration du pipeline

### 1. Création d'un nouveau pipeline

```yaml
# Configuration du pipeline DevOps Center
Pipeline Name: Main Development Pipeline
Repository: your-github-repo
Branch Strategy: Git Flow
Environments:
  - Development (DEV)
  - User Acceptance Testing (UAT)
  - Production (PROD)
```

1. Dans DevOps Center, cliquer sur **New Pipeline**
2. Sélectionner le repository GitHub connecté
3. Configurer les paramètres :
   - **Nom du pipeline** : `Main Development Pipeline`
   - **Branche principale** : `main`
   - **Stratégie de branche** : Git Flow

### 2. Configuration des environnements

#### Environnement de développement
```javascript
Environment Name: Development
Salesforce Org: DEV Sandbox
Git Branch: develop
Auto-deploy: Enabled
Tests Required: Unit Tests Only
```

#### Environnement de test
```javascript
Environment Name: UAT
Salesforce Org: UAT Sandbox
Git Branch: release
Auto-deploy: Disabled
Tests Required: All Tests
Code Coverage: 75%
```

#### Environnement de production
```javascript
Environment Name: Production
Salesforce Org: Production
Git Branch: main
Auto-deploy: Disabled
Tests Required: All Tests
Code Coverage: 75%
Approval Required: Yes
```

### 3. Configuration des règles de déploiement

```yaml
Deployment Rules:
  Development:
    - No approval required
    - Run unit tests only
    - Auto-deploy on commit
  
  UAT:
    - Approval from team lead required
    - Run all tests
    - Manual deployment trigger
  
  Production:
    - Approval from project manager required
    - Run all tests with 75% coverage
    - Change set validation
    - Manual deployment only
```

## Premier déploiement

### 1. Préparation du code source

```bash
# Récupération des métadonnées existantes
sfdx force:source:retrieve -m "CustomObject,ApexClass,Flow" -u production

# Conversion et commit
git add force-app/
git commit -m "Initial metadata import"
git push origin develop
```

### 2. Synchronisation initiale

1. Dans DevOps Center, aller dans **Environments**
2. Sélectionner l'environnement de développement
3. Cliquer sur **Sync with Org**
4. Confirmer la synchronisation

### 3. Premier déploiement vers UAT

```bash
# Processus de déploiement
1. Create Pull Request: develop → release
2. Review and approve in GitHub
3. Merge to release branch
4. Trigger deployment in DevOps Center
5. Monitor deployment status
```

1. Créer une **Pull Request** de `develop` vers `release`
2. Faire reviewer le code
3. Merger la PR
4. Dans DevOps Center :
   - Sélectionner l'environnement UAT
   - Cliquer sur **Deploy**
   - Valider les changements à déployer
   - Lancer le déploiement

### 4. Déploiement en production

```bash
# Processus de release
1. Create Pull Request: release → main
2. Final review and testing
3. Merge to main
4. Deploy to production via DevOps Center
5. Tag the release in GitHub
```

## Synchronisation DevOps Center ↔ GitHub Actions

### **🔄 Question clé : Les workflows sont-ils synchronisés ou découplés ?**

**Réponse : Ils sont partiellement découplés avec synchronisation événementielle.**

#### **Flux Standard : Découplage asynchrone**

```bash
DevOps Center → Opérations Git → GitHub Actions (déclenchement) → Execution isolée
     ↓              ↓                    ↓                           ↓
Abstraction    git push/merge     Événements standard          Pas de feedback
   Git         automatique        (push, pull_request)          vers DevOps Center
```

**Comment ça fonctionne :**

1. **DevOps Center exécute des opérations Git** automatiquement :
   ```bash
   # DevOps Center en arrière-plan fait :
   git checkout feature/work-item-123
   git commit -m "DevOps Center: Deploy work item 123"
   git push origin feature/work-item-123
   git merge feature/work-item-123 → develop
   ```

2. **GitHub Actions réagit aux événements Git** :
   ```yaml
   # GitHub Actions se déclenche automatiquement
   on:
     push:
       branches: [main, develop]
     pull_request:
       branches: [main, develop]
   ```

3. **❌ Découplage** : DevOps Center ne connaît PAS les résultats GitHub Actions :
   - DevOps Center ne voit pas si les tests GitHub Actions ont échoué
   - Pas de blocage automatique des déploiements DevOps Center en cas d'échec
   - Les deux systèmes opèrent de manière indépendante

#### **Flux Hybride : Couplage intentionnel**

Dans l'architecture hybride (template Szandor72), le couplage est **intentionnel** :

```yaml
# Quality gates AVANT que DevOps Center puisse merger
on:
  pull_request:
    branches: [integration]  # Branch target de DevOps Center
    paths: ['force-app/**']

jobs:
  quality-gates:
    # Validation obligatoire avant merge vers integration
    # Si échec → merge bloqué → DevOps Center ne peut pas progresser
```

### **📋 Comparaison des approches :**

| Aspect | DevOps Center Standard | Architecture Hybride |
|---------|----------------------|-------------------|
| **Synchronisation** | ❌ Découplée - événementielle seulement | ✅ Synchronisée - quality gates bloquantes |
| **Feedback** | ❌ DevOps Center ignore les résultats GitHub Actions | ✅ GitHub Actions bloque les merges ratés |
| **Complexité** | 🟢 Simple - deux systèmes indépendants | 🟡 Complexe - orchestration intentionnelle |
| **Contrôle qualité** | 🟡 Après coup via marketplace actions | ✅ Préventif via quality gates |

### **⚙️ Points techniques clés :**

#### **1. Événements Git déclencheurs**
```yaml
# GitHub Actions réagit aux opérations DevOps Center
on:
  push:                    # DevOps Center fait git push
  pull_request:           # DevOps Center crée des PR (dans certains workflows)
  workflow_dispatch:      # Déclenchement manuel possible
```

#### **2. Pas de communication bidirectionnelle native**
```bash
❌ DevOps Center → GitHub Actions (pas de communication directe)
✅ DevOps Center → Git Events → GitHub Actions (événementiel)
❌ GitHub Actions → DevOps Center (pas de feedback natif)
✅ GitHub Actions → Branch Protection → Blocage indirect DevOps Center
```

#### **3. Stratégies de couplage indirect**

**A. Via Branch Protection Rules** :
```yaml
# Configuration GitHub pour forcer la synchronisation
Branch Protection Rules:
  - Require status checks to pass (GitHub Actions)
  - Restrict pushes that fail quality gates
  # → DevOps Center ne peut plus merger si GitHub Actions échoue
```

**B. Via notifications externes** :
```yaml
# GitHub Actions notifie Salesforce
- name: Upload results to Salesforce
  run: |
    sfdx force:data:record:create -s DevOps_Action_Result__c \
      -v "Status__c=Failed" -u ${{ secrets.PROD_ORG }}
```

## ⚡ Point crucial : DevOps Center Standard PEUT être synchronisé

### **🔑 Simplification de l'approche hybride**

**Vous n'avez PAS besoin de l'architecture complexe de Szandor72 !**

Le **workflow standard DevOps Center** peut être bloqué par une configuration GitHub simple :

```yaml
# Configuration GitHub Repository → Settings → Branches
Branch Protection Rules sur "main" et "develop":
✅ Require status checks to pass before merging
✅ Require branches to be up to date before merging  
✅ Include administrators (force même les admins à respecter les rules)
```

### **🎯 L'étape 20a est CRUCIALE dans le diagramme**

```bash
GitHub Actions → Status: ✅ checks passed (étape 20a)
                    ↓
            DevOps Center peut merger
                    ↓
            Deployment autorisé vers UAT/PROD

GitHub Actions → Status: ❌ checks failed (étape 20b)  
                    ↓
            🛡️ BLOCAGE automatique
                    ↓
            DevOps Center ne peut PAS merger
```

### **📋 Configuration simple mais efficace**

#### Étape 1 : GitHub Actions basiques
```yaml
# .github/workflows/quality-check.yml
name: Quality Check
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Salesforce CLI
        run: npm install -g @salesforce/cli
      - name: Run Tests
        run: sfdx force:source:deploy --checkonly --testlevel RunLocalTests
```

#### Étape 2 : Branch Protection Rules
```bash
# Via GitHub Interface
Repository → Settings → Branches → Add Rule
Branch name pattern: main
☑️ Require status checks to pass before merging
☑️ Select: Quality Check (your GitHub Action)
☑️ Require branches to be up to date before merging
```

#### Étape 3 : DevOps Center automatiquement synchronisé !
```bash
✅ GitHub Actions valide = DevOps Center peut deployer
❌ GitHub Actions échoue = DevOps Center BLOQUÉ jusqu'à correction
```

### **🚀 Avantages de cette approche simplifiée**

| Aspect | Complexité | Efficacité |
|--------|------------|------------|
| **Setup** | 🟢 Simple (30 min) | ✅ Blocage efficace |
| **Maintenance** | 🟢 Minimal | ✅ Protection automatique |
| **Apprentissage** | 🟢 Facile | ✅ Résultats immédiats |

### **💡 Résumé : Synchronisation facile**

```bash
DevOps Center Standard + Branch Protection = Synchronisation forcée

Pas besoin de :
❌ Architecture multi-repositories
❌ NPM packages centralisés  
❌ Actions réutilisables complexes

Juste besoin de :
✅ GitHub Actions simple (tests/validation)
✅ Branch Protection Rules activées
✅ DevOps Center automatiquement synchronisé !
```

**L'étape 20a du diagramme est effectivement le point de contrôle qui change tout** - c'est là que GitHub valide ou bloque, et DevOps Center doit respecter cette décision grâce aux Branch Protection Rules ! 🎯

### Approche hybride pour dépasser les limitations

DevOps Center seul présente certaines limitations pour les pipelines CI/CD avancés. L'approche template développée par la communauté utilise GitHub Actions réutilisables pour créer un pipeline CI découplé avec des Quality Gates automatisées.

#### Architecture modulaire recommandée

```bash
# Structure de repositories pour approche avancée
┌─ devops-center-template/          # Template vide pour nouveaux projets
├─ devops-center-demo/              # Projet avec métadonnées + workflows
├─ devops-center-actions/           # Actions GitHub réutilisables
└─ devops-center-config-files/      # Package npm avec configs centralisées
```

#### 1. Repository principal (votre projet)
```yaml
# .github/workflows/validate-metadata.yml
name: Validate Metadata
on:
  pull_request:
    branches: [integration]
    paths: ['force-app/**']

jobs:
  metadata-validation:
    uses: your-org/devops-center-actions/.github/workflows/validate-metadata.yml@main
    secrets: inherit
    with:
      target-org: ${{ secrets.INTEGRATION_SANDBOX_SFDX_URL }}
```

#### 2. Actions réutilisables centralisées
```yaml
# Repository devops-center-actions
# .github/workflows/validate-metadata.yml
name: Reusable Metadata Validation
on:
  workflow_call:
    inputs:
      target-org:
        required: true
        type: string
    secrets:
      PROD_DEVOPS_CENTER_HOME_SFDX_URL:
        required: true
      INTEGRATION_SANDBOX_SFDX_URL:
        required: true

jobs:
  sfdx-scanner:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Salesforce CLI
        run: npm install -g @salesforce/cli
      - name: Install SFDX Scanner
        run: sfdx plugins:install @salesforce/sfdx-scanner
      - name: Run PMD Scan
        run: sfdx scanner:run --target "force-app/" --format csv
```

#### 3. Configuration centralisée via npm package
```json
// package.json in devops-center-config-files
{
  "name": "devops-center-local-config-files",
  "version": "1.0.0",
  "files": [
    "pmd-rulesets/",
    "ci-scripts/",
    ".eslintrc.js",
    ".prettierrc"
  ]
}
```

### Pipeline CI/CD avancé

#### Trois workflows configurés automatiquement

**1. Validation des métadonnées (`validate-metadata`)**
```yaml
Triggers: Pull Request vers 'integration'
Quality Gates:
- SFDX Scanner avec règles PMD personnalisées
- Scan spécifique par type (sObjects, Flows, Metadata)
- Prettification automatique des fichiers modifiés
- Validation uniquement des nouveaux/modifiés fichiers
```

**2. Validation du code (`validate-code`)**
```yaml
Triggers: Pull Request vers 'integration'  
Quality Gates:
- Tests Jest
- SFDX Scanner avec règles strictes
- Gestion des fichiers legacy (scan sans bloquer)
- Rapport CSV uploadé dans l'org pour notification
- Scan strict pour nouveaux fichiers
```

**3. Validation de déploiement (`validate-deployment`)**
```yaml
Triggers: Pull Request vers 'integration'
Quality Gates:
- Test deployment vers sandbox d'intégration  
- Exécution de tous les tests locaux
- Validation avec même metadata que DevOps Center utiliserait
```

### Configuration du template avancé

#### Étape 1 : Fork et setup initial
```bash
# 1. Fork du template
git clone https://github.com/your-org/devops-center-template.git
cd devops-center-template

# 2. Activation des GitHub Actions
GitHub Repository → Settings → Actions → General
Enable: "Allow all actions and reusable workflows"

# 3. Installation des config files
npm install devops-center-local-config-files
```

#### Étape 2 : Configuration des secrets
```yaml
# Secrets requis dans GitHub Repository
PROD_DEVOPS_CENTER_HOME_SFDX_URL:     # Pour query DevOps Center records
INTEGRATION_SANDBOX_SFDX_URL:         # Pour test deployments et CSV upload

# Format des SFDX URLs
force://PlatformCLI::5Aep861...@yourinstance.salesforce.com
```

#### Étape 3 : Structure des branches
```bash
# Branches requises pour le workflow
main          # Production (DevOps Center)
integration   # Target branch pour Pull Requests  
develop       # Développement (optionnel)
feature/*     # Branches de fonctionnalités
```

#### Étape 4 : Configuration DevOps Center
```bash
# DevOps Center reste le système principal pour :
1. Gestion des environnements (DEV → UAT → PROD)
2. Déploiements entre environnements  
3. Tracking des changements via Work Items
4. Gouvernance et approbations

# GitHub Actions complète avec :
1. Quality Gates automatisées
2. Tests avancés (PMD, Jest, SFDX Scanner)
3. Validation avant merge
4. Rapports détaillés et notifications
```

### Avantages de l'approche hybride

#### ✅ **Quality Gates automatisées**
- Scan automatique avec règles PMD personnalisées pour sObjects, Flows, et autres métadonnées
- Tests Jest intégrés
- Validation de déploiement automatique

#### ✅ **Gestion intelligente du legacy**
- Fichiers legacy scannés et reportés mais ne bloquent pas le développement nouveau
- Séparation entre code nouveau (règles strictes) et legacy (informatif seulement)

#### ✅ **Actions réutilisables**
- Configuration centralisée dans un package npm
- Workflows réutilisables permettent de faire des changements en un seul endroit pour tous les projets
- Marketplace GitHub Actions pour outils spécialisés

#### ✅ **Intégration transparente**
- DevOps Center reste le système principal de déploiement
- GitHub Actions agit comme quality gate avant les merges
- App Builders n'ont pas besoin d'utiliser VS Code ou CLI pour bénéficier des mêmes avantages que les développeurs

### Limitations à considérer

#### ⚠️ **Complexité initiale**
- Setup plus complexe que DevOps Center seul
- Maintenance de multiple repositories
- Courbe d'apprentissage GitHub Actions

#### ⚠️ **Dépendances externes**
- Dépendance aux GitHub Actions marketplace
- Package npm pour configurations centralisées
- Gestion des versions des actions

#### ⚠️ **Coûts potentiels**
- Consommation de minutes GitHub Actions
- Storage pour artifacts et logs
- Runners self-hosted si besoins spécifiques

### Structure du code
```bash
# Organisation recommandée
force-app/main/default/
├── applications/          # Applications personnalisées
├── classes/              # Classes Apex
├── components/           # Composants Lightning
├── flows/               # Flows
├── layouts/             # Layouts de page
├── objects/             # Objets personnalisés
├── permissionsets/      # Permission Sets
├── profiles/           # Profils
├── tabs/              # Onglets
└── triggers/          # Triggers
```

### Conventions de nommage
```javascript
// Classes Apex
ClassName + Controller/Service/Test
Example: AccountController, AccountService, AccountControllerTest

// Objets personnalisés
BusinessConcept__c
Example: CustomerAccount__c

// Champs personnalisés
FieldName__c
Example: CustomerScore__c
```

### Gestion des branches
```bash
# Stratégie Git Flow
main          # Production
├── release   # Pre-production
└── develop   # Development
    ├── feature/new-feature
    └── hotfix/critical-fix
```

### Tests et couverture
```apex
// Test classes obligatoires
@isTest
public class AccountControllerTest {
    @TestSetup
    static void setupTestData() {
        // Données de test
    }
    
    @isTest
    static void testPositiveScenario() {
        // Test du cas nominal
    }
    
    @isTest
    static void testNegativeScenario() {
        // Test des cas d'erreur
    }
}
```

## Dépannage

### Erreurs courantes

#### 🚨 1. DevOps Center absent de l'App Launcher
```bash
Symptôme: "DevOps Center n'apparaît pas dans l'App Launcher après installation"
Causes multiples et solutions :

Solution A - Profils manquants (PLUS FRÉQUENT):
1. Setup → App Manager → Trouver votre Connected App DevOps Center
2. Dropdown "Manage" → Manage Profiles
3. Sélectionner les profils qui doivent accéder à l'app
4. Sauvegarder et rafraîchir le navigateur

Solution B - Start URL manquante:
1. Setup → App Manager → Connected App DevOps Center
2. Modifier l'app → Basic Information
3. Ajouter Start URL: /sf_devops/DevOpsCenter.app
4. Sauvegarder

Solution C - Permission Set manquant:
1. Setup → App Manager → Connected App DevOps Center
2. Manage → Manage Permission Sets
3. Sélectionner "sf_devops_NamedCredentials"
4. Sauvegarder

Solution D - Accès direct (contournement):
URL: https://votre-domaine.salesforce.com/sf_devops/DevOpsCenter.app

Solution E - Recréation complète:
1. Supprimer la Connected App existante
2. Recréer avec toutes les étapes ci-dessus
3. Attendre quelques minutes et rafraîchir
```

#### 2. Problème d'accès aux repositories GitHub d'organisation
```bash
Symptôme: "The request was invalid. Resource protected by organization SAML enforcement"
Cause: Restrictions SAML de l'organisation GitHub
Solution:
1. Demander l'accès à un propriétaire de l'organisation GitHub
2. Dans GitHub → Organisation Settings → Third-party access
3. Autoriser Salesforce DevOps Center
4. Dans DevOps Center → Settings → Authentication Settings
5. Supprimer "DevOps Center GitHub" et se reconnecter
```

#### 2. Erreur de projet SFDX invalide
```bash
Symptôme: "URL does not reference a valid SFDX project. projectUrl"
Cause: Repository sans structure Salesforce DX
Solution:
1. Créer le fichier sfdx-project.json à la racine
2. Ajouter le dossier force-app/main/default/
3. Vérifier la structure du projet :
   Repository/
   ├── sfdx-project.json
   ├── force-app/
   │   └── main/
   │       └── default/
   └── .gitignore
```

#### 3. Problème de connexion avec email différent
```bash
Symptôme: Impossible de se connecter au repository GitHub
Cause: Email GitHub différent de l'email Salesforce
Solution:
1. Utiliser le même email pour GitHub et Salesforce OU
2. Utiliser un token d'accès personnel GitHub :
   - GitHub → Settings → Developer settings → Personal access tokens
   - Générer un token avec les permissions : repo, admin:org
   - Utiliser ce token lors de la connexion DevOps Center
```

#### 4. Échec de connexion GitHub
```bash
Symptôme: "Unable to connect to GitHub repository"
Solution:
1. Vérifier les permissions GitHub (repos, admin:org)
2. Renouveler le token d'authentification
3. Vérifier la configuration du Connected App
4. Clearing cache et reconnexion
```

#### 5. Échec de déploiement
```bash
Symptôme: "Deployment failed with validation errors"
Solution:
1. Vérifier les erreurs de validation dans DevOps Center
2. Corriger les erreurs dans le code
3. Vérifier les dépendances métadonnées
4. Recommiter et redéployer
```

#### 6. Problèmes de permissions Salesforce
```bash
Symptôme: "Insufficient privileges"
Solution:
1. Vérifier les Permission Sets assignés
2. Vérifier les permissions sur les métadonnées
3. Ajouter "DevOps Center Admin" permission
4. Contacter l'administrateur système
```

#### 7. Erreurs génériques d'interface
```bash
Symptômes: 
- "Sorry to interrupt. This page has an error"
- "An internal server error has occurred"
- "There was an error. Please reload the page"

Solutions:
1. Rafraîchir la page (F5)
2. Vider le cache du navigateur
3. Essayer en navigation privée
4. Vérifier l'édition Salesforce (Enterprise/Unlimited requis)
5. Vérifier la configuration des orgs connectées
```

### Commandes utiles de dépannage

```bash
# Vérification de la connexion org
sfdx force:org:list

# Validation d'un déploiement
sfdx force:source:deploy -c -p force-app/

# Récupération des logs d'erreur
sfdx force:apex:log:tail -c

# Test de connectivité GitHub
git remote -v
git fetch origin
```

### Support et ressources

#### Documentation officielle
- **Documentation officielle** : [Salesforce DevOps Center Docs](https://help.salesforce.com/s/articleView?id=sf.devops_center.htm)
- **Trailhead** : Modules DevOps Center
- **Community** : [Salesforce Developer Community](https://developer.salesforce.com/)
- **Support** : Contacter le support Salesforce

#### Templates et outils communautaires
- **DevOps Center Template** : [Template Repository](https://github.com/Szandor72/devops-center-template) par Szandor72
- **Demo Project** : [DevOps Center Demo](https://github.com/Szandor72/devops-center-demo) avec GitHub Actions configurées
- **Actions réutilisables** : [DevOps Center Actions](https://github.com/Szandor72/devops-center-actions)
- **Config centralisée** : [NPM Package](https://www.npmjs.com/package/devops-center-local-config-files)

#### GitHub Actions pour Salesforce
- **Salesforce Ben Guide** : [Complete Guide to Salesforce DevOps Automation](https://www.salesforceben.com/a-complete-guide-to-salesforce-devops-automation-with-github-actions/)
- **GitHub Actions Marketplace** : Actions pour Salesforce CLI, SFDX Scanner, etc.
- **Salesforce CLI Actions** : Actions officielles pour déploiements automatisés

---

## Checklist de validation

### Configuration de base
- [ ] DevOps Center activé et configuré
- [ ] Permissions utilisateurs assignées (Permission Sets + Profils)
- [ ] **Connected App créée avec Start URL et profils assignés**
- [ ] **DevOps Center visible dans l'App Launcher**
- [ ] Repository GitHub configuré avec la structure appropriée
- [ ] Connexion GitHub réussie (y compris repositories d'organisation)

### Pipeline standard DevOps Center
- [ ] Pipeline créé avec tous les environnements
- [ ] Règles de déploiement configurées
- [ ] Premier déploiement réussi
- [ ] Tests automatisés configurés
- [ ] Processus de review en place

### Architecture hybride (optionnelle - pour équipes avancées)
- [ ] Template DevOps Center forké et configuré
- [ ] GitHub Actions activées ("Allow all actions and reusable workflows")
- [ ] Secrets configurés (PROD_DEVOPS_CENTER_HOME_SFDX_URL, INTEGRATION_SANDBOX_SFDX_URL)
- [ ] Branches `main` et `integration` créées
- [ ] Actions réutilisables configurées depuis repository centralisé
- [ ] Package npm de configuration installé
- [ ] Workflows de validation (metadata, code, deployment) testés
- [ ] Quality gates PMD/SFDX Scanner configurées
- [ ] Gestion fichiers legacy vs nouveau développement
- [ ] Rapports CSV et notifications configurés

### Finalisation
- [ ] Documentation équipe mise à jour
- [ ] Formation équipe sur processus choisi (standard ou hybride)
- [ ] Processus de review et gouvernance en place

**Note** : Cette procédure suppose une configuration standard. Adaptez selon vos besoins spécifiques d'organisation et de gouvernance.
