# WriteUp de la box Photobomb

Difficulté : facile

Bonjour à toutes et tous, aujourd'hui je m'attaque à la machine Photobomb sur HackTheBox. 

C'est une machine assigné comme étant facile, et nous allons voir que les démarches pour atteindre les deux flags sont relativement rapides (si l'on cherche aux bons endroits).

On commence par réaliser un NMAP afin de scanner les ports ouverts sur la machine : 

![image](https://user-images.githubusercontent.com/33124690/195099486-b77ad55d-e35f-4a74-bfea-d0e4a677d81e.png)

On s'aperçoie que seul le port 80 (HTTP) ainsi que le port 22 (SSH) sont ouverts. On obtient aussi le nom de domaine (photobomb.htb) qu'il faudrait ajouter dans notre ficher "/etc/hosts"

Lorsque l'on se rend sur le site, on se rend compte qu'il n'y a pas grand chose, c'est un site vitrine (d'apparence). 

![image](https://user-images.githubusercontent.com/33124690/195100699-d756413d-d18f-4b08-9fb7-203ba7989aef.png)

Un lien permet de se connecter, mais sans identifiant, c'est impossible. Je vais lancer une énumération avec gobuster en fond, en parralèle, je continuerai l'énumération à la main afin de voir si l'on peut trouver quelque chose d'intéressant.

