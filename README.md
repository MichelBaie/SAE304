# SAE304 - Déployer un service de téléphonie multi-sites

Cette SAÉ a été réalisée dans le cadre de notre deuxième année de BUT Réseaux et Télécommunications, parcours Réseaux Opérateurs Multimédia, au sein de l’IUT de Villetaneuse.

## Introduction

Rédiger une introduction

## Pré-requis

- **VMWare Workstation Pro** : Pour créer et gérer les machines virtuelles. VMWare est désormais gratuit pour les particuliers et étudiants. Il intègre automatiquement les outils invités permettant le redimensionnement automatique de l'écran et la gestion simplifiée du presse-papier. Les installateurs pour Linux ou Windows sont disponibles sur le [CDN de VMWare](https://softwareupdate.vmware.com/cds/vmw-desktop/ws/)

## Infrastructure déployée 

![image-20250315161330960](img/image-20250315161330960.png)

Cette infrastructure représente une interconnexion entre **deux sites différents** chacun séparés par un **pare-feu Stormshield**.

Nous n’allons pas reproduire physiquement cette infrastructure.

Sur **VMWare**, nous reproduisons cette infrastructure avec cette liste de **VM (Virtual Machines / Machines virtuelles)**.

![image-20250315174500470](img/image-20250315174500470.png)

## Configuration réseau des machines virtuelles

Pour virtualiser tout cela, nous allons d’abord définir sur **VMWare** les différents réseaux.

<img src="img/image-20250315160945017.png" alt="image-20250315160945017" style="zoom:80%;" /> <img src="img/image-20250315175512660.png" alt="image-20250315175512660" style="zoom:80%;" />

Cliquer sur **Change Settings** pour pouvoir modifier les réseaux internes à **VMWare**.

Via le bouton **Add Network **(et **Rename Network** pour les renommer à souhait), créer **4 réseaux** :

- **LAN_SiteA** : Le réseau **LAN** dédié au **Site A**
- **LAN_SiteB** :  Le réseau **LAN** dédié au **Site B**
- **LAN_Switch** : Le réseau qui permettra de faire le lien entre une machine de **l’IUT** et un **switch **physique (utilisé par la suite pour faire fonctionner des téléphones physiques **Yealink T42U**).
- **WAN** : Le réseau **WAN** qui représentera **Internet** (entre les deux **pare-feux Stormshield**).

![image-20250315181505324](img/image-20250315181505324.png)

Penser à bien désactiver le service **DHCP** (nous en déploierons un par site.). Spécifiez également l’adresse du réseau sur chaque **LAN**.

![image-20250315181630303](img/image-20250315181630303.png)

Pour le **LAN_Switch**, spécifiez le réseau en **Bridged** vers l’interface réseau de votre **PC** qui sera branché sur le **switch**.

Maintenant que tous les réseaux sont configurés, <u>il faut les ajouter sur les **machines virtuelles (VM)**</u>.

#### Exemple sur la VM Stormshield Site A

![image-20250316002340606](img/image-20250316002340606.png)

Ici, on aura une interface dans le réseau **WAN** et l’autre dans le **LAN_SiteA**.

## Configuration du Site A

Nous allons déployer un **serveur 3CX** sur le **Site A**. Avant toute chose, il faut créer une licence chez **3CX** puisqu’il s’agit d’une **solution propriétaire**.

[Créer un compte puis se connecter sur **3CX**.](https://login.3cx.com/Account/Login)

![image-20250315182300263](img/image-20250315182300263.png)

Cliquer sur **ADD SYSTEM**.

![image-20250315182337166](img/image-20250315182337166.png)

Dans notre cas, il s’agit d’une installation **On Premise**, à savoir hébergé dans notre propre environnement.

![image-20250315182627201](img/image-20250315182627201.png)

Choisissez ici un **hostname**, soit un nom pour votre serveur **3CX**.

Il y a ici un semblant de pré-configuration. Nous choisissons ici le nombre de chiffres dont les **numéros courts** seront composés.

Les **numéros courts** (ou plus couramment **extensions**) sont les numéros **internes** aux utilisateurs. 

![image-20250315182642824](img/image-20250315182642824.png)

Nous pouvons tout laisser par défaut ici.

![image-20250315182838688](img/image-20250315182838688.png)

Une fois la licence configurée, nous choisissons la plateforme sur laquelle installer le **serveur 3CX** (choisir **Windows** ici).

![image-20250315183005744](img/image-20250315183005744.png)

Télécharger le serveur sous format **.exe** ainsi que le fichier de configuration proposé à **l’étape 3**. 

Sur la **VM**



![image-20250315184619730](img/image-20250315184619730.png)





