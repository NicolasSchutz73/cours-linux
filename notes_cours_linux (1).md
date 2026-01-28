# Notes de cours - Administration système Linux

## 1. Introduction aux composants serveur

### Origines de Linux : Linus Torvalds et le "hobby project"

**1991** : Linus Torvalds, étudiant à l'Université d'Helsinki, bidouille sur un PC Intel 386 et développe sous MINIX (OS éducatif).

**Au départ** : un émulateur de terminal + accès bas niveau → le projet "glisse" vers un noyau (kernel).

**Objectif** : un OS libre "à lui", inspiré d'Unix/MINIX mais écrit from scratch (pas de code MINIX).

**Annonce sur Usenet** (comp.os.minix) : _"I'm doing a (free) operating system… (just a hobby)"_

**La communauté s'en empare** : publication des premières versions (0.01) → contributions → naissance de l'écosystème.

### Services essentiels
- **SSHD** : Service SSH pour les connexions distantes sécurisées
- **systemd** : Gestionnaire central de tous les services (démarrage, arrêt, logs)
- **journalctl** : Outil de gestion et consultation des logs système
- **Gestionnaires de paquets** : apt, yum, etc. pour installer, mettre à jour et supprimer des paquets

### Outils réseau
- Configuration d'adresses IP
- Résolution de problèmes réseau et DNS

### Serveurs web (séances 3-4)
- **Apache** et **Nginx** : les deux serveurs web les plus utilisés
- **Reverse proxy** : configuration et mise en place
- **Monitoring** et sécurisation du serveur

---

## 2. Linux : kernel vs userland

### Comprendre pourquoi Linux ≠ distribution

**Kernel Linux** : 
- Gestion du matériel, mémoire, processus, réseau
- C'est le "cœur" du système d'exploitation
- Créé par Linus Torvalds

**GNU/userland** : 
- Shells (bash, zsh)
- Coreutils (ls, cp, grep, etc.)
- Bibliothèques système (libs)
- Outils et utilitaires

**Distribution** = kernel + userland + packaging + conventions

💡 C'est pourquoi on dit "GNU/Linux" : GNU apporte les outils, Linux apporte le noyau.

---

## 3. Distributions : lesquelles / pourquoi

### Debian / Ubuntu : serveurs, stabilité
- Très stables
- Énorme dépôt de paquets
- Très utilisées en serveur
- Ubuntu : cycles LTS (support long terme)
- Gestionnaire de paquets : **apt**

### RHEL/Alma/Rocky : écosystème entreprise
- Red Hat Enterprise Linux (RHEL) : monde entreprise
- Support long terme
- Normes strictes
- AlmaLinux et Rocky Linux : alternatives libres à RHEL
- Gestionnaire de paquets : **yum/dnf**

### Arch : rolling (moins adapté prod)
- Rolling release (toujours à jour)
- Très formateur pour apprendre
- ⚠️ Moins adapté à la production (mises à jour fréquentes)
- Gestionnaire de paquets : **pacman**

### Critères de choix
- **Stabilité** : fréquence des mises à jour
- **Support** : durée du support et sécurité
- **Paquets** : disponibilité des logiciels
- **Sécurité** : rapidité des correctifs

### Serveur sans GUI : pourquoi ?
- **Moins de surface d'attaque** : moins de logiciels = moins de failles potentielles
- **Moins de consommation** : CPU/RAM économisés
- **Plus proche "prod"** : les vrais serveurs n'ont pas d'interface graphique
- **Administration à distance** : via SSH uniquement

---

## 4. Flux d'entrée/sortie (stdin, stdout, stderr)

### Les trois flux standards
- **stdin (0)** : Entrée standard
- **stdout (1)** : Sortie standard (quand tout fonctionne bien)
- **stderr (2)** : Sortie d'erreur (messages d'erreur)

### Redirection des flux

#### Redirection de stdout
```bash
ls / > liste.txt          # Redirige vers un fichier (écrase)
echo "salut" >> salut.txt # Ajoute à la fin du fichier
```

⚠️ **Important** : 
- `>` écrase le fichier existant
- `>>` ajoute à la suite (append)

#### Redirection de stderr
```bash
ls /inexistant 2> erreur.txt  # Redirige les erreurs uniquement
```

#### Redirection combinée
```bash
commande > debug.log 2> erreur.log  # Sépare stdout et stderr
commande &> tout.log                # Redirige tout (stdout + stderr)
```

### Cas d'usage pratique
Exemple avec un serveur web (nginx) :
```bash
nginx > debug.log 2> erreur.log
```
- `debug.log` : contient les logs de visite (codes 200, 404, etc.)
- `erreur.log` : contient les erreurs fatales PHP, etc.

---

## 5. Les pipes (|)

### Principe
Le pipe permet de passer la sortie standard (stdout) d'une commande comme entrée standard (stdin) d'une autre commande.

### Exemples d'utilisation

#### Compter le nombre de fichiers
```bash
ls | wc -l  # Compte le nombre de lignes (= nombre de fichiers)
```

#### Rechercher dans un résultat
```bash
cat salut.txt | grep "2"  # Affiche uniquement les lignes contenant "2"
```

#### Filtrer des logs
```bash
cat erreur.log | grep "error" | grep "memory"
# Trouve toutes les lignes contenant "error" ET "memory"
```

#### Trier par taille
```bash
du -h | sort  # Affiche les tailles de fichiers triées
```

### Philosophie Linux
Chaque commande fait **une seule chose, mais bien**. On combine les commandes pour obtenir des résultats complexes.

💡 **Astuce** : On peut enchaîner autant de pipes qu'on veut !

---

## 6. La commande xargs

### Problème résolu
Certaines commandes (comme `rm`) n'acceptent pas stdin, uniquement des arguments en ligne de commande.

### Solution avec xargs
```bash
ls *.txt | xargs rm  # Supprime tous les fichiers .txt
```

`xargs` transforme le flux stdin en arguments de commande.

### Tester avant d'exécuter
```bash
ls *.txt | xargs -n1 echo rm  # Affiche ce qui sera exécuté sans le faire
```

Options utiles :
- `-n1` : passe un argument à la fois (crée plusieurs commandes)
- `-n` : spécifie le nombre maximum de paramètres par commande

⚠️ **Toujours tester avant une commande destructive !**

### Exemples pratiques

#### Vérifier la taille de logs
```bash
find /var/log -name "*.log" | xargs du -h | sort
```

#### Méthode moderne avec find -exec
```bash
find /var/log -name "*.log" -exec du -h {} \;
```

Les deux méthodes (xargs et -exec) sont valides. La méthode avec `-exec` est plus moderne et souvent trouvée dans les scripts actuels.

---

## 7. Autocomplétion avec Tab

- Appuyer sur **Tab** : complète automatiquement
- Appuyer **deux fois sur Tab** : liste toutes les possibilités

Exemples :
```bash
cat sal[Tab]           # Complète en "salut.txt"
cat test[Tab][Tab]     # Liste tous les fichiers commençant par "test"
```

💡 **Gain de temps énorme** au quotidien !

---

## 8. Permissions des fichiers

### Commande de base
```bash
ls -l  # Affiche les permissions détaillées
```

### Structure des permissions
```
-rw-r--r--  1  lilian  staff  1234  date  fichier
│└─┬─┘└─┬─┘└─┬─┘
│  │    │    └─ Autres (others)
│  │    └────── Groupe (group)
│  └─────────── Propriétaire (user/owner)
└────────────── Type (- = fichier, d = dossier)
```

### Les trois types de droits
- **r (read)** : Lecture
- **w (write)** : Écriture/Modification
- **x (execute)** : Exécution

### Exemple concret
```
-rw-r--r--  lilian  staff
```
- **Propriétaire (lilian)** : `rw-` (lecture + écriture)
- **Groupe (staff)** : `r--` (lecture seule)
- **Autres** : `r--` (lecture seule)

### Qui peut faire quoi ?
- **lilian** : peut lire ET modifier le fichier
- **john** (membre du groupe staff) : peut lire uniquement
- **tous les autres** : peuvent lire uniquement

### Modifier les permissions avec chmod

#### Méthode symbolique (avec lettres)
```bash
chmod u+x script.sh    # Ajouter exécution pour le propriétaire (user)
chmod g-w fichier.txt  # Retirer écriture pour le groupe
chmod o+r doc.txt      # Ajouter lecture pour les autres (others)
chmod a+x script.sh    # Ajouter exécution pour tous (all)
```

#### Méthode octale (avec chiffres)
La notation octale est plus précise et très utilisée. Chaque droit a une valeur :
- **r (read)** = 4
- **w (write)** = 2  
- **x (execute)** = 1

On **additionne** ces valeurs pour obtenir le chiffre final :
- **7** = 4+2+1 = rwx (tous les droits)
- **6** = 4+2 = rw- (lecture + écriture)
- **5** = 4+1 = r-x (lecture + exécution)
- **4** = 4 = r-- (lecture seule)
- **3** = 2+1 = -wx (écriture + exécution)
- **2** = 2 = -w- (écriture seule)
- **1** = 1 = --x (exécution seule)
- **0** = 0 = --- (aucun droit)

#### Exemples pratiques
```bash
chmod 755 script.sh
# 7 (propriétaire) = rwx
# 5 (groupe) = r-x
# 5 (autres) = r-x

chmod 644 fichier.txt
# 6 (propriétaire) = rw-
# 4 (groupe) = r--
# 4 (autres) = r--

chmod u+x script.sh     # Ajoute l'exécution au propriétaire
chmod 755 script.sh     # Équivalent : rwxr-xr-x
```

💡 **Astuce** : 755 est très courant pour les scripts, 644 pour les fichiers texte.

⚠️ **Attention** : Éviter 777 (tous les droits pour tout le monde) sauf cas très spécifique, c'est un risque de sécurité !

### Changer le propriétaire et le groupe

#### Changer le propriétaire (chown)
```bash
chown lilian test.sh           # Change le propriétaire
chown lilian:staff test.sh     # Change propriétaire ET groupe
chown -R lilian dossier/       # Change récursivement dans un dossier
```

#### Changer le groupe (chgrp)  
```bash
chgrp staff test.sh            # Change uniquement le groupe
chgrp -R staff dossier/        # Change récursivement
```

💡 **Important** : Seul le propriétaire actuel ou root peut changer les permissions d'un fichier.

---

## 9. sudo : pourquoi c'est sensible

### Concept de root vs sudo
- **root** = pouvoir total sur le système (superutilisateur)
- **sudo** = élévation contrôlée et traçable (loggable)
- Mauvaise configuration = escalade de privilèges facile

### Bonnes pratiques
- ❌ **Ne JAMAIS travailler en root** pour les tâches quotidiennes
- ✅ Utiliser un compte utilisateur normal
- ✅ Utiliser `sudo` uniquement quand nécessaire
- ✅ Les actions sudo sont enregistrées dans les logs

### Vérifier les droits sudo
```bash
sudo -v              # Vérifier si on a les droits sudo
sudo -l              # Lister ce qu'on peut faire avec sudo
groups               # Voir les groupes de l'utilisateur
```

### Ajouter un utilisateur au groupe sudo
```bash
# En tant que root ou avec sudo
usermod -aG sudo nom_utilisateur
```

---

## 10. Installation et configuration

### Mini quiz de diagnostic (avant de commencer)

Avant d'installer, testez vos connaissances :

| Question | Commande | Réponse attendue |
|----------|----------|------------------|
| Afficher le dossier courant ? | `pwd` | Chemin absolu |
| Lister fichiers cachés ? | `ls -a` | Tous les fichiers (y compris ceux commençant par .) |
| Chercher un mot dans un fichier ? | `grep "mot" fichier.txt` | Lignes contenant "mot" |
| Voir l'IP de la machine ? | `ip a` | Adresses IP des interfaces réseau |

### Prérequis
- **VirtualBox** installé (compatible Windows, Mac Intel et Mac Apple Silicon)
  - Bien choisir l'image correspondant à votre processeur (Intel ou Apple)
- **ISO Debian** (version **netinst** recommandée)

### Utilisateurs
Le système nécessite au minimum :
- Un compte **root** (super administrateur)
- Un utilisateur standard (ex: lilian)

---

## 11. Commandes utiles supplémentaires

### tail -f : Suivre un fichier en temps réel
```bash
tail -f /var/log/error.log  # Affiche les nouvelles lignes en direct
```

💡 **Astuce de débogage** : Ouvrir deux écrans
- Écran 1 : Application/site web
- Écran 2 : `tail -f error.log` pour voir les erreurs en temps réel

### echo : Afficher du texte
```bash
echo "Salut"              # Affiche "Salut"
echo "Salut" > salut.txt  # Écrit "Salut" dans un fichier
```

### grep : Rechercher dans un texte
```bash
grep "erreur" fichier.txt           # Recherche "erreur" dans le fichier
grep "erreur" fichier.txt | grep "memory"  # Recherche multiple
```

---

## Résumé des commandes essentielles

| Commande | Usage | Exemple |
|----------|-------|---------|
| `ls` | Lister les fichiers | `ls -l` |
| `cat` | Afficher le contenu d'un fichier | `cat fichier.txt` |
| `grep` | Rechercher dans un texte | `grep "mot" fichier.txt` |
| `wc -l` | Compter les lignes | `ls \| wc -l` |
| `du -h` | Afficher les tailles de fichiers | `du -h /var/log` |
| `sort` | Trier des résultats | `du -h \| sort` |
| `find` | Trouver des fichiers | `find /var/log -name "*.log"` |
| `tail -f` | Suivre un fichier en temps réel | `tail -f debug.log` |
| `xargs` | Convertir stdin en arguments | `ls \| xargs rm` |
| `echo` | Afficher du texte | `echo "Hello"` |

---

## Conseils pratiques

### 1. Maîtriser les bases plutôt que tout connaître
Un bon administrateur système ne connaît pas 800 000 commandes, mais sait combiner intelligemment 10-20 commandes de base.

### 2. Toujours tester avant de détruire
Utiliser `echo` ou `-n1` avec `xargs` pour vérifier ce qui sera exécuté :
```bash
ls *.txt | xargs -n1 echo rm  # Affiche sans exécuter
```

### 3. Utiliser Tab pour l'autocomplétion
Gain de temps énorme au quotidien !

### 4. Séparer les logs normaux et les erreurs
Facilite grandement le débogage :
```bash
commande > debug.log 2> erreur.log
```

### 5. Surveiller l'espace disque
Les fichiers de log peuvent exploser l'espace disque (exemple WordPress : 47 Go de logs !).
Vérifier régulièrement :
```bash
du -h /var/log | sort -h
```

### 6. Cas d'usage : Chercher les gros fichiers de log
```bash
find /var/log -name "*.log" | xargs du -h | sort -h
```

---

## Points importants à retenir

### Flux de données
- **stdout (1)** = sortie normale → s'affiche par défaut
- **stderr (2)** = sortie d'erreur → s'affiche par défaut aussi
- On peut **rediriger** ces flux vers des fichiers ou d'autres commandes

### Redirection vs Pipe
- **Redirection (`>`, `>>`, `2>`)** : Envoie vers un fichier
- **Pipe (`|`)** : Envoie vers une autre commande

### xargs
Transforme stdin en arguments pour les commandes qui n'acceptent pas stdin directement.

### Permissions
- 3 catégories : **propriétaire**, **groupe**, **autres**
- 3 droits : **lecture (r)**, **écriture (w)**, **exécution (x)**

---

## Notes complémentaires

### Pourquoi séparer stdout et stderr ?
Permet de :
- Garder des logs de fonctionnement normal dans un fichier
- Garder des logs d'erreurs dans un autre fichier
- Traiter différemment les succès et les échecs

Exemple concret : Un script qui tourne en tâche de fond (background) n'affiche rien à l'écran, donc on redirige tout vers des fichiers pour garder une trace.

### Enchaînement de pipes
On peut enchaîner autant de pipes qu'on veut :
```bash
cat fichier.log | grep "error" | grep "memory" | wc -l
```

Cela permet de construire des traitements complexes à partir de commandes simples.

---

**Date de création des notes** : Janvier 2026  
**Système** : Linux Debian (recommandé pour l'apprentissage)
