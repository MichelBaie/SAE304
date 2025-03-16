# SAE304 - Déployer un service de téléphonie multi-sites

Cette SAÉ a été réalisée dans le cadre de notre deuxième année de BUT Réseaux et Télécommunications, parcours Réseaux Opérateurs Multimédia, au sein de l’IUT de Villetaneuse.

## Introduction

Rédiger une introduction

## Pré-requis

- **VMware Workstation Pro** : Pour créer et gérer les machines virtuelles. VMWare est désormais gratuit pour les particuliers et étudiants. Il intègre automatiquement les outils invités permettant le redimensionnement automatique de l'écran et la gestion simplifiée du presse-papier. Les installateurs pour Linux ou Windows sont disponibles sur le [CDN de VMWare](https://softwareupdate.vmware.com/cds/vmw-desktop/ws/)

## Infrastructure déployée 

![image-20250315161330960](img/image-20250315161330960.png)

Cette infrastructure représente une interconnexion entre **deux sites différents** chacun séparés par un **pare-feu Stormshield**.

Nous n’allons pas reproduire physiquement cette infrastructure.

Sur **VMware**, nous reproduisons cette infrastructure avec cette liste de **VM (Virtual Machines / Machines virtuelles)**.

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

Pensez à bien désactiver le service **DHCP** (nous en déploierons un par site.). Spécifiez également l’adresse du réseau sur chaque **LAN**.

![image-20250315181630303](img/image-20250315181630303.png)

Pour le **LAN_Switch**, spécifiez le réseau en **Bridged** vers l’interface réseau de votre **PC** qui sera branché sur le **switch**.

Maintenant que tous les réseaux sont configurés, <u>il faut les ajouter sur les **machines virtuelles (VM)**</u>.

#### Exemple sur la VM Stormshield Site A

![image-20250316002340606](img/image-20250316002340606.png)

Ici, nous aurons une interface dans le réseau **WAN** et l’autre dans le **LAN_SiteA**.

## Configuration du Site A

>  [!CAUTION]
>
> Ne démarrez pas encore la machine **3CX**.

### Configuration du DHCP

Nous allons d’abord procéder à la configuration du serveur **DHCP**.

>  [!TIP]
>
> Aller dans **VM -> Send Ctrl+Alt+Del** pour déverrouiller la **VM**.
>
> ![image-20250316021006992](img/image-20250316021006992.png)

>  [!NOTE]
>
> Le mot de passe est **Admin01!**

Sur la VM **DHCP_DNS_SiteA**, cliquer sur **Ajouter des rôles et des fonctionnalités**.

![image-20250316022729610](img/image-20250316022729610.png)L

Tout laisser par défaut jusqu’à **Rôles de serveurs** puis sélectionner les deux fonctionnalités **Serveur DHCP** et **Serveur DNS**.

![image-20250316022720109](img/image-20250316022720109.png)

Cliquer sur **Suivant** et laisser toutes les options par défaut.



>  [!NOTE]
>
> Si demandé, cliquer sur Finaliser la configuration **DHCP** à partir de ce menu :<img src="img/image-20250316022750048.png" alt="image-20250316022750048" style="zoom:67%;" />

Nous allons maintenant configurer le service **DHCP** : ![image-20250316022831604](img/image-20250316022831604.png)



Cliquer droit sur **IPv4** et sélectionner **Nouvelle étendue**
Cela nous permettra de créer une **pool DHCP**.

![image-20250316023021507](img/image-20250316023021507.png)

Donnez un nom à l’étendue (p. ex: **sitea.local**) : 

![image-20250316023148591](img/image-20250316023148591.png)

Attribuez la plage d’adresse **192.168.1.100 -> 192.168.1.200**. Le masque sera **255.255.255.0 (/24 CIDR)**

 ![image-20250316023318051](img/image-20250316023318051.png)

Nous allons configurer dès maintenant les **options DHCP**.

- **Routeur (passerelle par défaut) : <u>192.168.1.1</u>**

  ![image-20250317001555551](img/image-20250317001555551.png)

  > [!CAUTION]
  >
  > Attention à bien cliquer sur **Ajouter** pour que l’entrée soit enregistrée.

- **Domaine parent : <u>sitea.local</u>**

- **Adresse IP du serveur DNS : <u>192.168.1.20</u>**

  ![image-20250317001831943](img/image-20250317001831943.png)

Activer **l’étendue**.

En général, il est peu pratique qu’un **routeur/pare-feu** ou qu’un quelconque **serveur** possède une adresse IP qui varie dans le temps. Pour pallier à ce souci, nous allons **réserver** une adresse IP au **pare-feu** et au **serveur 3CX**.

Sur la **VM Stormshield** et la **VM 3CX**, allez dans les paramètres réseau et recueillir **l’adresse MAC** de chaque interface **<u>dans le réseau LAN_SiteA</u>**.

<img src="img/image-20250316113603358.png" alt="image-20250316113603358" style="zoom: 80%;" />

Pour créer la réservation, cliquer sur **Nouvelle réservation** sur l’onglet **Réservations** :

![image-20250316113757564](img/image-20250316113757564.png)

Renseignez le nom de la réservation, l’adresse IP que l’on veut fixer ainsi que l’adresse MAC de la machine.![image-20250316114034388](img/image-20250316114034388.png)

>  [!IMPORTANT]
> De la même manière, fixer l’adresse du **pare-feu Stormshield** à **192.168.1.1**.



Le service **DHCP** est maintenant opérationnel.

### Configuration du DNS

Nous allons maintenant configurer le **serveur DNS**.<img src="img/image-20250316112809706.png" alt="image-20250316112809706" style="zoom:67%;" />

Puisqu’il y a **deux sites**, il faudra donc créer **deux zones DNS**.![image-20250316113053659](img/image-20250316113053659.png)

Cliquez sur **Nouvelle zone…**sur l’onglet **Zones de recherche directes**.

Nous allons définir une **zone principale** ayant pour nom **sitea.local**.

>  [!NOTE]
>
> Nous autorisons les **mises à jour dynamiques** afin de pouvoir mettre dynamiquement à jour la **base DNS** lors d’un **bail DHCP**.

Nous définissons de la même manière une **zone principale** ayant pour nom **siteb.local**.

Dans la zone **sitea.local**, créer **deux hôtes** avec leur adresse IP respective : 

- **VOTRE_FQDN** -> **192.168.1.254**
- **stormshield**.sitea.local -> **192.168.1.1**

<img src="img/image-20250316132210905.png" alt="image-20250316132210905" style="zoom:67%;" />

![image-20250316132405205](img/image-20250316132405205.png)

Dans la zone **siteb.local**, rajoutez de la même manière une entrée pour : 

- **freepbx**.siteb.local -> **1.2.3.2**

- **stormshield**.siteb.local -> **1.2.3.2**

  ![image-20250316140729764](img/image-20250316140729764.png)

>  [!NOTE]
>
> L’adresse IP de **freepbx**.siteb.local est la même que le **stormshield** car nous allons faire une **translation d’adresse (NAT)**.



### Configuration du serveur 3CX

Nous allons déployer un **serveur 3CX** sur le **Site A**. Avant toute chose, il faut créer une licence chez **3CX** puisqu’il s’agit d’une **solution propriétaire**.

[Créer un compte puis se connecter sur **3CX**.](https://login.3cx.com/Account/Login)![image-20250315182300263](img/image-20250315182300263.png)

Cliquer sur **ADD SYSTEM**.![image-20250315182337166](img/image-20250315182337166.png)

Dans notre cas, il s’agit d’une installation **On Premise**, à savoir hébergé dans notre propre environnement.

![image-20250316021828597](img/image-20250316021828597.png)

Choisissez ici un **hostname**, soit un nom pour votre serveur **3CX**.

> [!WARNING]
>
> Bien sélectionner **Use Your Own SSL & FQDN Certificate** car nous ne diffuserons pas *réellement* le serveur en ligne.

Il y a ici un semblant de pré-configuration. Nous choisissons ici le nombre de chiffres dont les **numéros courts** seront composés.

>  [!NOTE] 
>
> Les **numéros courts** (ou plus couramment **extensions**) sont les numéros **internes** aux utilisateurs. 

![image-20250315182642824](img/image-20250315182642824.png)

Nous pouvons tout laisser par défaut ici.

![image-20250315182838688](img/image-20250315182838688.png)

Une fois la licence configurée, nous choisissons la plateforme sur laquelle installer le **serveur 3CX** (choisir **Windows** ici).

![image-20250316022037359](img/image-20250316022037359.png)

>  [!IMPORTANT] 
>
> Penser à bien noter le mot de passe.

Téléchargez le serveur sous format **.exe** ainsi que le fichier de configuration proposé à **l’étape 3** (ou copiez le lien proposé - cela revient au même que le fichier de configuration). 



>  [!CAUTION]
>
> Lors de la première configuration de la **VM 3CX**, il est nécessaire que la **VM** aie un accès à Internet pour qu’elle puisse activer le **FQDN** auprès de 3CX.
>
> <u>Configurer temporairement la carte réseau en **NAT** (Il faut donc que la machine hôte aie un accès internet également.)</u>
>
> ![image-20250316220234709](img/image-20250316220234709.png)
>
> Vous pouvez ensuite démarrer la **VM**.

> [!TIP]
>
> Il devrait déjà être sur le Bureau de la **VM 3CX_Site_A**.



Sur la VM **3CX_Site_A**, installer **3CXPhoneSystem** et choisir la **configuration en ligne de commandes**.

![image-20250315184619730](img/image-20250315184619730.png)



> [!WARNING] 
>
> Si vous rencontrez encore des soucis pour accéder au **serveur 3CX**, tentez de redémarrer la **VM 3CX**.
>
> En parallèle, assurez-vous de bien pouvoir communiquer en **DNS**(*nslookup*), **IP**(*ping*)…

L’interface de **3CX** est disponible sur le port **5015** à l’adresse : http://localhost:5015/ 

Suivre le guide d’installation et y **importer le fichier de configuration**.

Le **serveur 3CX** vous demandera un **certificat** et une **clé privée**. Pour se faire, allez sur votre hôte **Debian** et exécutez la commande : 

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -subj "/C=FR/ST=Ile-de-France/L=Villetaneuse/O=USPN/CN=HOSTNAME.sitea.local" -keyout private.key  -out certificate.crt
```

>  [!CAUTION]
>
> Changez le **CN** (**C**ommon **N**ame) dans la commande par votre **FQDN**.

Vous devriez retrouver dans votre répertoire personnel deux fichiers : 

![image-20250316224257208](img/image-20250316224257208.png)

Glissez ces fichiers dans votre **VM 3CX** puis spécifiez les chemins : 

```batch
C:\Users\Administrateur\Desktop\certificate.crt
C:\Users\Administrateur\Desktop\private.key
```

> [!TIP]
>
> Si vous glissez les fichiers sur le Bureau, vous pouvez copier directement ces chemins.

![image-20250316235930992](img/image-20250316235930992.png)



Une fois la configuration importée, nous pouvons accéder à l’interface web de **3CX**.

> [!CAUTIOn]
>
> Repassez maintenant l’interface réseau de la **VM 3CX** en **LAN_SiteA**
>
> Sur un terminal **PowerShell**, exécutez 
>
> ```batch
> ipconfig
> ipconfig /renew
> ```
>
> ![image-20250317002054323](img/image-20250317002054323.png)
>
> Confirmez bien que **l’adresse IP** de la **VM 3CX** est bien **192.168.1.254**
>
> Notez que la **VM** n’aura plus accès à **Internet** à partir de ce moment.

Le **serveur 3CX** est maintenant opérationnel et accessible via **HTTPS**.

<img src="img/image-20250317002333154.png" alt="image-20250317002333154" style="zoom:67%;" />

>  [!NOTE]
>
> Les identifiants sont ceux donnés lors de la création de licence.

Familiarisez-vous avec l’interface, notamment le menu **Admin**, trouvable en bas des onglets latéraux.

![image-20250317003311824](img/image-20250317003311824.png)

### Ajout d’extensions

Nous allons maintenant ajouter des utilisateurs.

Allez dans **Utilisateurs**, vous y verrez déjà un utilisateur (vous).![image-20250317003516382](img/image-20250317003516382.png)

Vous pouvez en créer un via **Ajouter un utilisateur**.![image-20250317003734278](img/image-20250317003734278.png)

Renseignez les informations relatives à votre nouvel utilisateur.

>  [!NOTE]
>
> L’extension représente le numéro court de l’utilisateur - le numéro que les autres utilisateurs **internes** composeront.
>
> Notez que le **numéro SDA** (***S**élection **D**irecte à **l’A**rrivée*) représente le **numéro long** - le numéro que les autres utilisateurs **externes** (via un **Trunk SIP** que nous configurerons plus tard) composeront.

### Configuration du Softphone

Pour configurer un **softphone**, il faut sélectionner l’utilisateur sur **3CX** qui va l’utiliser.

Allez sur **Téléphone IP**.![image-20250317004235908](img/image-20250317004235908.png)

Pour configurer un **softphone**, il faut sélectionner **Je configurerai le téléphone moi-même**.![image-20250317004338792](img/image-20250317004338792.png)

Il vous sera demandé par quelle interface le softphone doit-il communiquer avec le **serveur 3CX**, utilisez le **FQDN**.![image-20250317004433553](img/image-20250317004433553.png)



Il sera ainsi affiché les identifiants de connexion pour votre **softphone**.

> [!NOTE] 
>
> Notez ces identifiants pour la configuration du **softphone**.![image-20250317004512944](img/image-20250317004512944.png)
