# Découverte du type de fichier
En premier lieu on va essayer de déterminer le type de fichier qu'on a : ``file rev``

````
rev: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=e4dbcb1281821db359d566c68fea7380aeb27378, for GNU/Linux 3.2.0, not stripped
````
On peut également faire un checksec : 
````
checksec rev 
[*] '/home/rabbit/Documents/Workspace/Projects/nightmare/NightmareJourney/helithumper_rev/rev'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
````

# Etapes pour trouver le flag
Maintenant on va petit à petit essayer de comprendre comment fonctionne le binaire, donc d'abord on tente de le run et voir son comportement d'un point de vue externe.
On s'aperçoit qu'on nous demande de saisir un input, et l'input semble être comparé à quelque chose puisqu'on nous retourne une chaîne de caractère spécifiant d'aller nous faire foutre.

On va donc essayer de trouver le système gérant cette comparaison via de l'analyse statique ou dynamique, à votre choix, personnellement je trouve l'analyse dynamique souvent plus pratique, donc j'ai opté pou GDB.

## Utilisation de GDB
Ainsi, via GDB on va parcourir notre binaire et chercher des indices.
La première chose à faire est d'essayer de désassembler la fameuse fonction main de notre programme, on va essayer de le faire manière explicite tel que :

``disass main``

C'est parfait, le main est directement détectable et on peut dès à présent voir sa composition à notre écran.
La première chose qui m'intrigue est l'appel à une fonction validate, on se doutait de part le comportement du programme lorsqu'on l'a run qu'il y avait probablement une comparaison de faite, et avec ce validate, ça semble bien être le cas.
On va donc essayer de décortiquer cette fonction validate afin de trouver potentiellement des indices pour trouver notre flag.

Pour cela, on va en premier lieu désassembler validate : ``disass validate``.

On s'aperçoit de plusieurs instructions mov qui enregistrent des valeurs dans des adresses pointées à registre rbp-OFFSET. 
Tout ça semble fishy, ça ressemble étrangement à une processus de création de chaîne de caractères.
De ce fait, afin d'être certain de ce qui est renvoyé, on va essayer d'afficher la chaîne de caractère générée grâce à la première instruction enregistrant un caractère et la dernière. On va grâce aux adresses relatives de ces instructions créer une boucle pour afficher chaque caractère qui aura été généré.
Tout cela se procède ainsi :

![gdb](https://github.com/zxtNX/NightmareJourney/blob/main/helithumper_rev/gdb_dump_string.png?raw=true)

A noter que l'on peut faire ça après exécution des instructions, donc au mieux on aura au préalable set un breakpoint après la dernière instruction mov soit : ``b *validate+125``.
