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

## Bonnes pratiques

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

- **Documentation officielle** : [Salesforce DevOps Center Docs](https://help.salesforce.com/s/articleView?id=sf.devops_center.htm)
- **Trailhead** : Modules DevOps Center
- **Community** : [Salesforce Developer Community](https://developer.salesforce.com/)
- **Support** : Contacter le support Salesforce

---

## Checklist de validation

- [ ] DevOps Center activé et configuré
- [ ] Permissions utilisateurs assignées (Permission Sets + Profils)
- [ ] **Connected App créée avec Start URL et profils assignés**
- [ ] **DevOps Center visible dans l'App Launcher**
- [ ] Repository GitHub configuré avec la structure appropriée
- [ ] Connexion GitHub réussie (y compris repositories d'organisation)
- [ ] Pipeline créé avec tous les environnements
- [ ] Règles de déploiement configurées
- [ ] Premier déploiement réussi
- [ ] Tests automatisés configurés
- [ ] Processus de review en place
- [ ] Documentation mise à jour

**Note** : Cette procédure suppose une configuration standard. Adaptez selon vos besoins spécifiques d'organisation et de gouvernance.
