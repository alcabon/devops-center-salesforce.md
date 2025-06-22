# Installer Node.js dans n'importe quel répertoire Windows

Oui, absolument \! En tant qu'expert Node.js, je peux vous confirmer qu'**il est tout à fait possible d'installer Node.js dans n'importe quel répertoire Windows sans nécessiter d'élévation de privilèges (droits administrateur)**, ce qui est une solution excellente pour les laptops avec des restrictions d'installation.

Voici les méthodes principales pour y parvenir :

### 1\. Utiliser la version `.zip` (recommandé pour votre cas)

C'est la méthode la plus simple et la plus directe pour contourner les restrictions :

  * **Téléchargez l'archive `.zip` :** Rendez-vous sur la page de téléchargement officielle de Node.js ([https://nodejs.org/en/download/](https://nodejs.org/en/download/)). Cherchez la section "Windows Binary" (ou "Windows Installer" et choisissez l'option "64-bit" ou "32-bit" selon votre système, mais assurez-vous de prendre le fichier **.zip**).
  * **Décompressez l'archive :** Une fois téléchargée, décompressez le contenu du fichier `.zip` dans un répertoire de votre choix où vous avez les droits d'écriture. Par exemple, vous pouvez le décompresser dans `C:\Users\VotreNomUtilisateur\Nodejs` ou directement sur un lecteur externe, une clé USB, etc.
  * **Ajouter au PATH (facultatif mais recommandé) :** Pour pouvoir exécuter `node` et `npm` depuis n'importe quelle ligne de commande, vous devrez ajouter le chemin du dossier décompressé (celui qui contient `node.exe` et `npm.cmd`) à votre variable d'environnement `PATH`.
      * Recherchez "Modifier les variables d'environnement système" dans le menu Démarrer de Windows.
      * Cliquez sur "Variables d'environnement...".
      * Dans la section "Variables utilisateur pour [VotreNomUtilisateur]", trouvez la variable `Path` et cliquez sur "Modifier...".
      * Clique   z sur "Nouveau" et ajoutez le chemin complet vers le dossier où vous avez décompressé Node.js (par exemple, `C:\Users\VotreNomUtilisateur\Nodejs`).
      * Validez toutes les fenêtres avec "OK".
      * Redémarrez votre terminal ou invite de commande pour que les changements prennent effet.

### 2\. Utiliser NVM for Windows (Node Version Manager)

Bien que NVM for Windows soit un installateur et puisse potentiellement nécessiter des droits lors de sa première installation, une fois installé, il permet de gérer et d'installer différentes versions de Node.js sans privilèges supplémentaires. Si vous avez la possibilité de faire une installation initiale avec des droits (même temporairement, ou si un administrateur peut le faire pour vous une seule fois), c'est une excellente solution pour la flexibilité.

  * **Téléchargez NVM for Windows :** [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases) (Prenez le fichier `nvm-setup.zip` ou `nvm-setup.exe`).
  * **Installation initiale :** Si vous utilisez le `.exe`, il peut demander des droits. Si c'est le cas et que vous êtes bloqué, la méthode `.zip` est préférable. Cependant, si vous pouvez l'installer, il vous demandera un répertoire d'installation.
  * **Utilisation de NVM :** Une fois NVM installé, vous pouvez installer n'importe quelle version de Node.js sans droits administrateur via des commandes comme :
    ```bash
    nvm install 18.17.0 # Installe Node.js version 18.17.0
    nvm use 18.17.0   # Utilise cette version
    ```
    NVM installe les versions de Node.js dans un répertoire qu'il gère lui-même (généralement `C:\Users\VotreNomUtilisateur\AppData\Roaming\nvm`), et il manipule les liens symboliques et le PATH pour vous, sans nécessiter de droits supplémentaires après son installation initiale.

### 3\. Utiliser un gestionnaire de packages portable (ex: Scoop)

Des outils comme Scoop ([https://scoop.sh/](https://scoop.sh/)) sont conçus pour installer des outils en ligne de commande de manière portable et sans nécessiter de droits administrateur.

  * **Installation de Scoop :**
    ```powershell
    Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
    irm get.scoop.sh | iex
    ```
    (Cette commande est à exécuter dans PowerShell. La première ligne peut nécessiter une confirmation si votre politique d'exécution est plus restrictive).
  * **Installation de Node.js avec Scoop :**
    ```powershell
    scoop install nodejs
    ```
    Scoop installera Node.js dans votre répertoire utilisateur (`C:\Users\VotreNomUtilisateur\scoop\apps`) et gérera automatiquement le `PATH`.

### Avantages de ces approches pour votre situation :

  * **Pas d'élévation de droits :** Le principal avantage est que vous n'avez pas besoin de droits administrateur pour installer ou utiliser Node.js.
  * **Installation locale :** Node.js est installé dans un répertoire de votre choix, généralement dans votre espace utilisateur, ce qui est parfait pour les environnements restreints.
  * **Portable :** Surtout avec la méthode `.zip`, vous pouvez même placer le dossier Node.js sur une clé USB et l'exécuter depuis n'importe quel ordinateur Windows.

### Recommandation spécifique pour les laptops très restreints :

Pour le cas de laptops qui restreignent les installations de manière stricte, la **méthode de l'archive `.zip` est la plus fiable et la plus simple**. Elle ne nécessite aucune exécution d'installateur et vous donne un contrôle total sur l'emplacement des fichiers. L'étape d'ajout au `PATH` est très recommandée pour le confort d'utilisation, mais même sans elle, vous pourriez toujours exécuter `node.exe` en naviguant directement dans le répertoire d'installation.

N'hésitez pas si vous avez d'autres questions \!
