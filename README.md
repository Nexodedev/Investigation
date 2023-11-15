faire un journal de l'aventure avec des screens
Je vais rédiger ce journal en anglais et français du fait que certaines informations fournies sont en anglais mais la langue utilisée pour ma docuementation personnelle est le français.
---
1)
Vidéo présentation de la situation, un ransomware a chiffré les données de l'entreprise et une rançon de 5 millions de monero est demandée à payer.
---
2)
La deadline se site demaine 9 h du matin.

Voici comment se déroulera l'enquête :
1. I'll prepare groups of leads for you to choose from.
2. Investigate each lead by completing the activity.
3. At the end of each phase of the investigation, you'll report to me and we'll discuss your findings.
---
Information : We know that Amari Rivera was the author of the leaked file.

Step 1)
Search for Leaked File:
[To determine who leaked the file and how, we’ll need to find it on our environment. I’d start by running a search in Purview of our SharePoint sites. See if you can figure out where Amari saved the file after he authored it. Good luck!]

Une fois sur la machine administrateur, je commence par aller sur Purview.
Dessus j'ai créé une recherche sur le nom du fichier recherché avec l'auteur.

[Investi01]

Step 2)
Investigate Amari in Sentinel & Defender:
[Was Amari’s device compromised and how? Start in Microsoft Sentinel as we always do, investigate Amari’s device and see what you can find. If you find something, continue your investigation in Microsoft 365 Defender.]

[Investi02]
Je commence par ouvrir Microsoft Sentinel, dans la liste des incidents je détecte que un incident lié à Microsoft Defender est présent.
Je commence donc mon investigation dessus.
[investi03]
Je remarque qu'un powershell malicieux a été exécuté.
Avec les informations que j'ai je pense que celà pourrait correspondre à l'installation du ransomware (à vérifier).
En tout cas ce que je vois c'est que ce terminal malieux a été ouvert sur le pc105 avec l'user de amari.rivera qui est la personne d'où les documents ont leak.
Cela me renforce dans l'idée que j'ai surement à faire au ransomware.
[Investi04]
En regardant le pc105 je vois que dessus le seul user est amari rivera et que surtout il y a deux alertes importantes : Reflective dll loading detected et malicious powershell invoked.

[Investi05]
Dans la timeline je vois bien les différentes étapes utilisées par l'attaquant pour installer son ransomware sur la machine.
Il faudrait maintenant que je trouve comment il est entré. J'ai vu que la machine comporte plusieurs vulnérabilités, peut être que l'une d'entre elle existait pre-attaque et a été utilisée par l'attaquant comme entry point.
J'ai détecté qu'une attaque de spray password a été effecutée le jour avant l'installation du ransomware sur amari rivera, je pense donc avec grande certitude que son mot de passe a été compromis le jour avant par l'attaquant.
[Investi06]
Je passe ensuite par les logs, dans ceux ci je commence par rechercher amire.rivera, le nombre de résultats retournés étant trop grand j'affine cette recherche.
Pour cela j'entre que je cherche dans SecurityAlert donc user.
Après avoir effectué cette recherche j'ai identifiés plusieurs informations utiles.
[Investi07]
J'ai ensuite découvert aussi la liste des process en cours et patch.exe le présumé malware y est présent ainsi que cmd.exe qui est surement corélé au powershell.

Step 3)
Investigate Amari in Azure AD Identity Protection:
[Amari's user identity might be compromised! Use Azure Active Directory (AD) Identity Protection to investigate. Let me know if you see any anomalies.]

Je commence par me rendre dans Azure Active Directory.
Dans Identity Protection | Risk detections j'ai identifiés 7 indices en rapport avec Amari Rivera le 27/10 en rapport avec l'attaque par spray de password.
Dans Risky users pour Amari Rivera j'ai trouvé 3 indices aussi.

Step 4)
Set Up Insider Risk Policy
[Let's keep an eye on Amari and his team. Set up an insider risk policy for the eCommerce app team. We haven’t configured sensitivity labels yet, but make sure it protects any credit card information stored on their SharePoint site.]

J'ai maintenant créé des policies de risk sur l'application d'ecommerce pour les cartes de crédit.

Voici la fin d'investigation du matin.
[Investi08]

Set Up Compliance Policies:
[The information in the leaked file is confidential and should have been protected. The legal and executive team want us to set up a sensitivity label for the eCommerce app team. It should encrypt files and emails that contain credit card information. Use an auto-labeling policy to apply it.]

Je dois maintenant créer sur Purview un label pour chiffrer les mails et fichiers contenant des informations de cartes de crédit.
Maintenant que j'ai créé celle-ci je peux passer à la suite.
Ici on est encore dans une phase d'amélioration et pas de recherche du coupable.
J'ai donc configuré une "auto-labeling policy" pour exchange, sharpoint et les comptes onedrive.
J'ai du créer cette "policy" en mode simulation pour que je puisse évaluer si ma policy était bien réglée et pouvoir ensuite la modifier/régler en fonction des résulstats. Et une fois que les résultats sont satisfaisants je peux alors valider réellement la modification.

Investigate Amari’s Device in Microsoft 365 Defender:
[We need to find out more about how this attack took place. Check Amari's device for evidence of the curl command being run on it, or any other information that provides more detail on the attack. Are there any other suspicious files?]

Je commence maintenant mon enquête plus en profondeur sur microsoft defender.
J'ai commencé par faire du "hunting" en utilisant KQL pour trouver des traces d'adresses IP utilisées par les attaquants.
Ensuite dans le menu d'alertes j'ai trouvé des alertes conercnant des actions malveillantes effectuées par les attaquants.
Sur le device affecté par l'attaque j'ai trouvé des fichiers notables.

Search for Internal Communication Containing the IP Address:
[The external IP address used in the attack is an Indicator of Compromise (IoC). We should search our environment for emails, documents, and Teams communication for information regarding this IoC.]

Avec les adresses IPs des attaquants j'ai pu chercher du contenu lié à des échanges email et Teams.
Pour cela j'ai filtré sur l'endroit et la condition.
Après exportation et analyse des échanges email de la victime. J'ai remarqué qu'angel brown a envoyé par email la commande curl malveillante.
Elle pourrait donc être elle même infectée ou être l'attaquante.


Investigate IP Address in Sentinel:
[The external IP address used in the attack is an Indicator of Compromise (IoC). We need to figure out if the IoC has been seen in our environment by any other sources including devices and Azure resources. We should set up an Analytics rule to immediately notify us if the IoC is accessed again.]

J'ai vérifié que Amira est la seule à s'être connectée à la machine infectée et créé une règle.
J'ai configuré la règle pour qu'elle regarde après des évènements sur le réseau liés à l'adresse IP de l'attaquant.
Via cette règle si l'adresse IP attaquante réeffectue des actions sur le réseau cela sera directement détectés dans les logs.

Configure Windows Security Baseline:
[You noticed our devices are not configured with a standard security configuration. We should configure the devices to use a Windows Security Baseline. Not only will this help protect our users and devices, but it will also allow our team to quickly eliminate possible attack vectors based on the security configuration.]
[Investi09]
Configuration des divices.
J'ai créé un profile pouir la Windows Security Baseline pour configurer les devices de façon automatique en appliquant les meilleurs politiques de sécurité dessus.
Après analyse j'ai pu découvrir que Powershell a essayé de passer outre l'antivirus en appelant les APIs de Win32 directement.

J'en ai conclu à la fin de cette phase qu'il y a eu un autre fichier extrait ShoppingList.zip .
Step 4)

Configure Azure AD Identity Protection:
J'ai découvert que Azure AD Identity Protection n'était pas configuré automatiquement pour gérer le risque utilisateur et le risque d'authentification.
J'ai donc remédié à cela en le configurant.

Investigate Angel's Sign-In Logs:
J'ai maintenant essayé de découvrir si les attaquants auraient pu compromettre l'identité d'Angel.
Si oui les soupçons sur elle pourrait être amoindri si non augmentés.
Le résultat est qu'après vérification rien n'a été trouvé concernant un vol de son identité.

Investigate Angel in Sentinel and Microsoft 365 Defender:
Après enquête sur le device qu'a utilisé Angel rien de suspect n'a été trouvé.
Cela me laisse penser que si elle est la source de cette attaque elle a du utiliser ce device pour se connecter à un autre device comme une connexion SSH par exemple.

Communication Compliance Search:
J'ai alors créé une nouvelle recherche avec dans laquelle j'utilise d'autres conclusions précédents que j'ai ensuite exportées.

Investigate Tomo’s Device in Sentinel and Microsoft 365 Defender:
J'ai alors découvert que le device de Tomo était connecté à celui d'Angel.
Ce qui correspond à la théorie que j'ai émise plus haut qu'elle aurait pu sur son device se connecter à un autre device pour effectuer ses actions.
J'ai donc après exploré les pistes en cherchant sur les devices utilisés par Tomo et j'ai découvert la présence d'évènements liées à une connexion RDP.


Pour finir après avoir effectuer toutes les tâches.
J'en conclu que c'est Angel Brown qui a effectué cette attaque via RDP.
Après l'avoir accusée celle-ci avoue et j'ai donc réussi mon enquête.

En résumé tout a commencé par la découverte de la situation.
Pour débuter l'enquête il a fallu déterminer qui, quand, où et comment.
Ce que j'ai fait en observant via les différents outils les différentes informations disponibles.
En paralèle j'ai contribué à la sécurisation de l'infrastructure avec des ajouts de règles, policies et autres.
Au début je comprends assez vite que le ransomware a été installé via le compte utiliser de Amari sur le pc105.
Je comprends que le malware se now patch.exe et je sais quels fichiers ont été impactés.
Lors de la suite de mes investigations j'ai pu recréer la timeline de l'attaque de quelle machine vers quelle machine de quelle ip vers quelle ip et de quel user vers quel user.
J'ai ensuite découvert que Angel n'a pas été comprise  et qu'il n'y a eu aucune activité sur son compte ce qui m'a laissé pensé que c'était un indice envers elle.
Et après avoir découvert que une connexion RDC a été effectuée sur le compte de Tomo par Angel.
Et donc j'en ai conclu qu'Angel a envoyé les commandes malicieuses via la connexion RDP sur le device de Tomo.

[Investi10]
[Investi11]
