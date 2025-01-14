*Licence 3 Informatique – Option: Systèmes et Réseaux*

*INF 3611: Administration Systèmes*

*Travaux pratiques*

*Par:* **NGOUANFO**

# **TP: Virtualisation avec KVM**

[**Objectifs	1**](#objectifs)

[**Prérequis	1**](#prérequis)

[**Partie 0 : Connaissances générales	1**](#partie-0-:-connaissances-générales)

[**Partie 1 : Installation de KVM et ses outils de gestion	2**](#partie-1-:-installation-de-kvm-et-ses-outils-de-gestion)

[**Partie 2 : Installation et gestion d’une VM	3**](#partie-2-:-installation-et-gestion-d’une-vm)

[**Partie 3 : Installation d’une VM Debian sans assistance	5**](#partie-3-:-installation-d’une-vm-debian-sans-assistance)

[Remarques	5](#remarques)

[**Quelques liens utiles	6**](#quelques-liens-utiles)

# **Objectifs** {#objectifs}

* Installer KVM et ses outils de gestion.  
* Manipuler les commandes principales de gestion des machines virtuelles.  
* Déployer des machines virtuelles sur KVM.  
* Automatiser le déploiement de VM.

# **Prérequis** {#prérequis}

* Une machine Linux (Ubuntu) avec des capacités de virtualisation.  
* Un compte utilisateur avec les privilèges du super utilisateur.  
* Une bonne connaissance de la ligne de commande Linux.

# **Partie 0 : Connaissances générales** {#partie-0-:-connaissances-générales}

KVM (Kernel-based Virtual Machine) est une technologie de virtualisation intégrée au noyau Linux. C’est le module du au noyau Linux qui permet de transformer un système Linux en un hyperviseur de type 1\.

* **libvirt** est une API de virtualisation qui permet d’administrer les VM.  
* **virsh** est une interface en ligne de commande pour gérer les VM via libvirt.  
* **qemu-img** est un outil pour gérer les images disque.  
* **virt-manager :** Interface graphique pour gérer les VM.  
* **virt-install :** Commande pour créer des VM.  
* **virt-viewer :** Client console graphique.  
* **virt-clone :** Outil pour cloner des VM.  
* **virt-top :** Outil pour surveiller les performances des VM.  
* etc.

# **Partie 1 : Installation de KVM et ses outils de gestion** {#partie-1-:-installation-de-kvm-et-ses-outils-de-gestion}

1. Vérifier la compatibilité de votre hôte avec KVM

| egrep \-c '(vmx|svm)' /proc/cpuinfo |
| :---- |

1. Utiliser la commande suivante pour installer Installez KVM et les paquets nécessaires sur votre hôte. Quel est le rôle de chacun de ces paquets?

| sudo apt updatesudo apt install \-y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager |
| :---- |

2. Démarrer du commutateur virtuel par défaut.

| sudo virsh net-start default sudo virsh net-autostart default |
| :---- |

3. Vérifier le chargement du module kvm.

| lsmod | grep kvm |
| :---- |

4. Ajoutez votre utilisateur au groupe libvirt pour gérer les VMs sans nécessité de privilèges root :

| sudo usermod \-aG libvirt $(whoami) |
| :---- |

5. Démarrer et activer le service libvirtd :

| sudo systemctl start libvirtd sudo systemctl enable  libvirtd  |
| :---- |

6. Vérifier que le service libvirt est démarré et fonctionne

| sudo systemctl status libvirtd |
| :---- |

7. Vérifier et valider votre installation :

| virt-host-validate |
| :---- |

Quelques emplacement utiles :

1. Dossier des images disque : /var/lib/libvirt/images/  
2. Fichiers de configurations : /etc/libvirt/qemu/  
3. Dossier des logs : /var/log/libvirt/  
4. …

# **Partie 2 : Installation et gestion d’une VM** {#partie-2-:-installation-et-gestion-d’une-vm}

1. Déployer une machine virtuelle à l’aide de la commande virt-install. Télécharger l’ image du système d’exploitation **Ubuntu-server-22.04**.  
   Exemple de paramètres requis pour installer Ubuntu Server 22.04:  
* Nom de la VM : vm-inf361  
* CPU : 2 cœurs; RAM : 2048 Mo  
* Stockage : 10 Go  
* Image ISO d’Ubuntu : Ubuntu-server-22.04.iso

Lancer l’installation de Ubuntu-server-22.04.iso en exécutant la commande suivante:

| virt-install \\           \--name vm-inf361 \\           \--vcpus 2 \\           \--memory 2048 \\           \--disk size=10 \\           \--cdrom /chemin/vers/votre/Ubuntu-server-22.04.iso \\           \--os-variant ubuntu22.04 |
| :---- |

Gestion des VM avec virsh 

2. Consulter la page de manuel ou d’aide de virsh: man virsh ou virsh \--help  
3. Lister les VM : virsh list \--all  
4. Informations sur une VM : virsh dominfo vm-inf361  
5. Démarrer une VM : virsh start vm-inf361  
6. Arrêter une VM en douceur : virsh shutdown vm-inf361  
7. Arrêter brutalement une VM : virsh shutdown vm-inf361  
8. Redémarrer une VM : virsh reboot vm-inf361  
9. Mettre en pause une VM : virsh suspend vm-inf361  
10. Relancer une VM mise en pause : virsh resume vm-inf361

L’utilitaire virt-clone permet de cloner une VM. Il prend la peine de générer une nouvelle adresse MAC et un nouvel uuid pour le domaine (vm). Il s’occupe également de dupliquer le ou les disques attachés au domaine.

1. C’est quoi un clone d’une VM? Dans quel scénario peut-il être utilisé?   
2. Quels sont les paramètres que vous devez modifier après avoir démarré un clone? Expliquer pourquoi?  
3. Quelles sont les précautions à prendre avant de cloner une VM? Créer un clone de la VM vm-inf361 et nommé le clone-vm-inf361. 

| virt-clone \\  \--original vm-inf361 \\  \--name clone\-vm-inf361 \\  \--file /var/lib/libvirt/images/clone\-vm-inf361.qcow2 |
| :---- |

4. Lister les VM : virsh list \--all

5. Supprimer la VM vm-inf361 et démarrer le snapshot créé précédemment 

| virsh undefine vm-inf361virsh start clone\-vm-inf361 |
| :---- |

6. Créer un snapshot de la VM clone-vm-inf361

| virsh snapshot-create-as clone\-vm-inf361 "snapshot1" "Snapshot avant création d’un utilisateur" |
| :---- |

7. Lister les snapshots

| virsh snapshot-list clone\-vm-inf361 |
| :---- |

8. Créer un utilisateur dans clone-vm-inf361

9. Restaurer la VM  clone-vm-inf361à partir d’un snapshot (snapshot1)

| virsh snapshot-revert clone\-vm-inf361 snapshot1 |
| :---- |

L’utilisateur que vous avez créé existe-t-il toujours après restauration du snapshot? pourquoi?

10. À partir de la ligne de commande de la question 1, écrire un script Bash qui prend en paramètres: nom de la VM, CPU, RAM, taille de disque, image ISO, réseau, etc.) et lance le déploiement d’une VM.

# **Partie 3 : Installation d’une VM Debian sans assistance** {#partie-3-:-installation-d’une-vm-debian-sans-assistance}

L’installation manuelle d’un système peut être chronophage, surtout lorsqu’il s’agit de déployer plusieurs machines. Automatiser ce processus permet de gagner du temps et d'assurer une standardisation des installations. **Kickstart** et **Preseed** sont deux outils couramment utilisés pour automatiser ce processus :

* **Preseed** est un système d’automatisation d’installation utilisé principalement pour les distributions basées sur Debian (Ubuntu, etc.). Il utilise un fichier de configuration qui fournit les réponses aux questions posées par l'installateur Debian.  
* **Kickstart** est utilisé pour les distributions de la famille Red Hat (RHEL, Fedora, CentOS, etc.), Kickstart automatise l'installation en fournissant un fichier de configuration contenant les options nécessaires pour une installation complète.  
1. Télécharger une image de la famille RHEL et placer la dans un répertoire accessible, comme /var/lib/libvirt/images/.  
2. Créer un fichier Kickstart (kickstart.cfg)  contenant les paramètres de l'installation pour l'installation automatisée.  
3. Tester la syntaxe et le contenu de votre fichier Kickstart avant de l'utiliser :

| ksvalidator kickstart.cfg |
| :---- |

4. Sauvegarder ce fichier dans un endroit accessible, comme un serveur HTTP, NFS ou FTP.  
5. Utiliser la commande virt-install pour lancer l’installation sans assistance d’une VM, en spécifiant le fichier Kickstart.  
6. Vous pouvez suivre l'installation via virt-viewer ou virsh console  
   1. **virt-viewer** (interface graphique) : virt-viewer \<nom-vm\>  
   2. **virsh console** (console texte) : virsh console \<nom-vm\>

## **Remarques** {#remarques}

* Vous pouvez rapidement configurer un serveur HTTP local pour le fichier Kickstart avec la commande:

| sudo python3 \-m http.server 8000 \--directory /chemin/vers/ks/ |
| :---- |

# **Quelques liens utiles** {#quelques-liens-utiles}

* Consulter la documentation en ligne sur Libvirt.org : [https://libvirt.org/manpages/virsh.html](https://libvirt.org/manpages/virsh.html)  
* Virtualisation KVM:  
  * [https://gitlab.com/xavki/presentations-kvm-libvirt-fr](https://gitlab.com/xavki/presentations-kvm-libvirt-fr)  
  *  [https://lpic2.goffinet.org/virtualisation\_kvm.html](https://lpic2.goffinet.org/virtualisation_kvm.html)  
  * [https://linux.goffinet.org/administration/virtualisation-linux/virtualisation-kvm/](https://linux.goffinet.org/administration/virtualisation-linux/virtualisation-kvm/)  
* Kickstart Documentation : [https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html)  
* Automating the installation using preseeding : [https://wiki.debian.org/DebianInstaller/Preseed](https://wiki.debian.org/DebianInstaller/Preseed)
