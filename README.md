# WriteUp de la box Photobomb

Difficulté : facile

Bonjour à toutes et tous, aujourd'hui je m'attaque à la machine Photobomb sur HackTheBox. 

C'est une machine assigné comme étant facile, et nous allons voir que les démarches pour atteindre les deux flags sont relativement rapides (si l'on cherche aux bons endroits).

## Enumération

On commence par réaliser un NMAP afin de scanner les ports ouverts sur la machine : 

![image](https://user-images.githubusercontent.com/33124690/195099486-b77ad55d-e35f-4a74-bfea-d0e4a677d81e.png)

On s'aperçoie que seul le port 80 (HTTP) ainsi que le port 22 (SSH) sont ouverts. On obtient aussi le nom de domaine (photobomb.htb) qu'il faudrait ajouter dans notre ficher "/etc/hosts"

Lorsque l'on se rend sur le site, on se rend compte qu'il n'y a pas grand chose, c'est un site vitrine (d'apparence). 

![image](https://user-images.githubusercontent.com/33124690/195100699-d756413d-d18f-4b08-9fb7-203ba7989aef.png)

Un lien permet de se connecter pour accéder à une autre partie du site. Néanmoins, sans creds, impossible de se connecter, il va falloir trouver une solution. On peut quand même remarquer qu'en cas d'échec d'authentification, on est redirigés sur "/printers".

![image](https://user-images.githubusercontent.com/33124690/195103675-392f6fa7-ed1d-4be0-9de6-2e5de3c0514c.png)

On lance une énumération des "dirs" avec gobuster en fond, en parralèle, on continue l'énumération à la main afin de voir si l'on peut trouver quelque chose d'intéressant.

![image](https://user-images.githubusercontent.com/33124690/195105567-1f95810a-096e-415f-9be4-2d3d1354bc33.png)

## Accès au "/printers"

Le résultat du gobuster ne sera pas intéressant car toutes les pages que l'on trouve nécessite une authentification. Toujours nétant qu'en inspectant le code source de la page, on s'aperçoit qu'un fichier JS est utilisé. En y accédant, on obtient des creds visiblement déstiné au support technique du site web.

![image](https://user-images.githubusercontent.com/33124690/195106155-64a3d6f0-0ce7-4f7f-aa11-d58bd4662bd9.png)

Il suffit d'utiliser ces creds pour nous connecter ! :)


