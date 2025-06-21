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

```javascript
// Configuration OAuth pour GitHub
Name: DevOps Center GitHub Integration
Contact Email: admin@yourcompany.com
Callback URL: https://yourinstance.salesforce.com/services/authcallback/DevOpsCenter
Scopes:
- Access and manage your data (api)
- Perform requests on your behalf at any time (refresh_token, offline_access)
```

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

1. Ouvrir **DevOps Center** depuis l'App Launcher
2. Cliquer sur **Connect to GitHub**
3. Sélectionner **GitHub.com** ou **GitHub Enterprise**
4. Autoriser l'accès à votre compte GitHub
5. Sélectionner le repository à connecter

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

#### 1. Échec de connexion GitHub
```bash
Symptôme: "Unable to connect to GitHub repository"
Solution:
1. Vérifier les permissions GitHub
2. Renouveler le token d'authentification
3. Vérifier la configuration du Connected App
```

#### 2. Échec de déploiement
```bash
Symptôme: "Deployment failed with validation errors"
Solution:
1. Vérifier les erreurs de validation dans DevOps Center
2. Corriger les erreurs dans le code
3. Recommiter et redéployer
```

#### 3. Problèmes de permissions
```bash
Symptôme: "Insufficient privileges"
Solution:
1. Vérifier les Permission Sets assignés
2. Vérifier les permissions sur les métadonnées
3. Contacter l'administrateur système
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
- [ ] Permissions utilisateurs assignées
- [ ] Repository GitHub configuré avec la structure appropriée
- [ ] Pipeline créé avec tous les environnements
- [ ] Règles de déploiement configurées
- [ ] Premier déploiement réussi
- [ ] Tests automatisés configurés
- [ ] Processus de review en place
- [ ] Documentation mise à jour

**Note** : Cette procédure suppose une configuration standard. Adaptez selon vos besoins spécifiques d'organisation et de gouvernance.
