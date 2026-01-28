# Notes de cours - Administration système Linux

## 1. Introduction aux composants serveur

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

## 2. Flux d'entrée/sortie (stdin, stdout, stderr)

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

## 3. Les pipes (|)

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

## 4. La commande xargs

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

## 5. Autocomplétion avec Tab

- Appuyer sur **Tab** : complète automatiquement
- Appuyer **deux fois sur Tab** : liste toutes les possibilités

Exemples :
```bash
cat sal[Tab]           # Complète en "salut.txt"
cat test[Tab][Tab]     # Liste tous les fichiers commençant par "test"
```

💡 **Gain de temps énorme** au quotidien !

---

## 6. Permissions des fichiers

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

---

## 7. Installation et configuration

### Prérequis
- **VirtualBox** installé (compatible Windows, Mac Intel et Mac Apple Silicon)
  - Bien choisir l'image correspondant à votre processeur (Intel ou Apple)
- **ISO Debian** (version **netinst** recommandée)

### Utilisateurs
Le système nécessite au minimum :
- Un compte **root** (super administrateur)
- Un utilisateur standard (ex: lilian)

---

## 8. Commandes utiles supplémentaires

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
