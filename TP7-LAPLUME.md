`Laplume Sorenza`

# TP 7 - Boot, services et processus / Tâches d’administration (2)

## Exercice 1. Personnalisation de GRUB

### 1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il est présent dans votre environnement (vous pouvez aussi commenter son contenu).

Pour renommer le fichier j'utilise la commande **mv 50-curtin-settings.cfg 50-curtin-settings.txt**.

### 2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ; passé ce délai, le premier OS du menu doit être lancé automatiquement.

Il faut modifier la ligne **GRUB_TIMEOUT=10** 

### 3. Lancez la commande update-grub

<pre>
root@serveur:/etc/default# update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.0.0-32-generic
Found initrd image: /boot/initrd.img-5.0.0-32-generic
Found linux image: /boot/vmlinuz-5.0.0-31-generic
Found initrd image: /boot/initrd.img-5.0.0-31-generic
done
</pre>

### 4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte

Les changements ont bien été pris en compte. Il est bien proposer pendant 10 sec.

### 5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres à rajouter au fichier grub.

Il faut rajouté la ligne **GRUB_GFXPAYLOAD=1024x768** dans le fichier /etc/default/grub. 

### 6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splashimages (après installation, celles-ci sont disponibles dans /usr/share/images/grub).

Afin d'ajouter un fond d'écran au grub j'utilise la commande :**GRUB_BACKGROUND=/usr/share/images/grub/Lake_mapourika_NZ.tga**

### 7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*). Installez-en un.

Afin  de configurer un thème j'utilise la commande : **GRUB_THEME="/boot/grub/themes/ubuntu-mate/theme/txt"**

### 8. Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.

J'ajoute au fichier /etc/grub.d/40_custom :

<pre>
menuentry 'Arrêt du système' {
	halt
}
menuentry 'Redémarrage du système' {
	reboot
}
</pre>

### 9. Configurer GRUB pour que le clavier soit en français

J'ai créer un fichier menu.lst dans le répertoire /boot/grub. J'ai remplie le fichier avec les instructions suivante :

<pre>
# Emulation d'un clavier azerty_FR
setkey less backquote
setkey greater tilde
setkey ampersand 1
setkey 1 exclam
setkey tilde 2
setkey 2 at
setkey doublequote 3
setkey 3 numbersign
setkey quote 4
setkey 4 dollar
setkey parenleft 5
setkey 5 percent
setkey minus 6
setkey 6 caret
setkey backquote 7
setkey 7 ampersand
setkey underscore 8
setkey 8 asterisk
setkey backslash 9
setkey 9 parenleft
setkey at 0
setkey 0 parenright
setkey parenright minus
setkey numbersign underscore
# no change for equal
# no change for plus
setkey a q
setkey A Q
setkey z w
setkey Z W
setkey caret bracketleft
# no equivalent for diaresis => we keep the US braceleft
setkey dollar bracketright
# no equivalent for pound => we keep the US braceright
setkey q a
setkey Q A
setkey m semicolon
setkey M colon
setkey bracketleft quote 
setkey percent doublequote
setkey asterisk backslash
setkey bracketright bar
setkey w z
setkey W Z
setkey comma m
setkey question M
setkey semicolon comma
setkey period less
setkey colon period
setkey slash greater
setkey exclam slash
setkey bar question
</pre>

## Exercice 2. Noyau

Dans cet exercice, on va créer et installer un module pour le noyau.

### 1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).


### 2. Créez un fichier hello.c contenant le code suivant :

<pre>
1 #include <linux/module.h>
2 #include <linux/kernel.h>
3
4 MODULE_LICENSE("GPL");
5 MODULE_AUTHOR("John Doe");
6 MODULE_DESCRIPTION("Module hello world");
7 MODULE_VERSION("Version 1.00");
8
9 int init_module(void)
10 {
11 printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");
12 return 0;
13 }
14
15 void cleanup_module(void)
16 {
17 printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");
18 }

</pre>

### 3. Créez également un fichier Makefile :
<pre>
1 obj -m += hello.o
2
3 all:
4 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
5
6 clean:
7 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
8
9 install:
10 cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
</pre>

### 4. Compilez le module à l’aide de la commande make, puis installez-le à l’aide de la commande make install.

### 5. Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod.
### 6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez notamment voir les informations figurant dans le fichier C.
### 7. Déchargez le module ; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module() est appelée” a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande lsmod.
### 8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.

## Exercice 3. Interception de signaux

La commande interne trap permet de redéfinir des gestionnaires pour les signaux reçus par un processus.
Un cas d’utilisation typique est la suppression des fichiers temporaires créés par un script lorsque celui-ci est
interrompu.

### 1. Commencez par écrire un script qui recopie dans un fichier tmp.txt chaque ligne saisie au clavier par l’utilisateur
### 2. Lancez votre script et appuyez sur CTRL+Z. Que se passe-t-il ? Comment faire pour que le script poursuive son exécution ?
### 3. Toujours pendant l’exécution du script, appuyez sur CTRL+C. Que se passe-t-il ?
### 4. Modifiez votre script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP (= CTRL+Z) et SIGINT (= CTRL+C) : dans le premier cas, il doit afficher ”Impossible de me placer en arrière-plan”, et dans le second cas, il doit afficher ”OK, je fais un peu de ménage avant” avant de supprimer le fichier temporaire et terminer le script.
### 5. Testez le nouveau comportement de votre script en utilisant d’une part les raccourcis clavier, d’autre part la commande kill
### 6. Relancez votre script et faites immédiatement un CTRL+C : vous obtenez un message d’erreur vous indiquant que le fichier tmp.txt n’existe pas. A l’aide de la commande interne test, corrigez votre script pour que ce message n’apparaisse plus.

## Exercice 4. Surveillance de l’activité du système

### 1. Dans tty1, lancez la commande htop, puis tapez la commande w dans tty2. Qu’affiche cette commande ?
### 2. Comment afficher l’historique des dernières connexions à la machine ?
### 3. Quelle commande permet d’obtenir la version du noyau ?
### 4. Comment récupérer toutes les informations sur le processeur, au format JSON ?
### 5. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ? Comment afficher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot ?
### 6. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?
### 7. Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message à l’écran d’une maintenance le 26 mars à minuit.
### 8. Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + Fk−2, avec F0 = F1 = 1. Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal virtuel. Que constatez-vous ? Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur l’affichage de tload.
