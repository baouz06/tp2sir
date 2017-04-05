# TP 2 de Systèmes d'informations répartis


## Objectifs du TP

1. Comprendre les mécanismes de JPA
2. Réaliser une application en utilisant JPA en se plaçant dans un cadre classique de développement sans serveur d’application au départ.

## Sujet

L’objectif de ce projet est de construire une application type réseau social permettant de comparer 
sa consommation électrique avec ses amis, ses voisins etc. dans la lignée de opower.

## Le modèle métier de projet

On utilise le concept de Personne ayant un nom un prénom, un mail, une ou plusieurs résidence. 
Chaque résidence a une taille, un nombre de pièce, des chauffages, des équipements électroniques. 
Ses équipements ont une consommation moyenne en Watt/h.

![model](https://cloud.githubusercontent.com/assets/15005875/24684497/b92f1ffe-19a6-11e7-9628-f9a097ea34e2.png)


## Démarrage de la base de données

Nous utilisons le script de démarrage de la base de données (run-hsqldb-server.sh) 
et le script du démarrage du Manager (show-hsqldb.sh).

On modifie notre fichier de persistance.xml pour la connexion de la base de donnée

```
<properties>
        <property name="hibernate.dialect" value="org.hibernate.dialect.HSQLDialect"/>
	<property name="hibernate.hbm2ddl.auto" value="create"/>
    	<property name="toplink.target-database" value="HSQL"/>
        <property name="hibernate.connection.driver_class" value="org.hsqldb.jdbcDriver"/>
        <property name="hibernate.connection.username" value="sa"/>
        <property name="hibernate.connection.password" value=""/>
        <property name="hibernate.connection.url" value="jdbc:hsqldb:hsql://localhost/"/>
        <property name="hibernate.max_fetch_depth" value="3"/>  
</properties>
```
Dans notre fichier de pom.xml nous utilisons ces dépendances au dessous.
```
<dependency>
	<groupId>org.hibernate</groupId>
	<artifactId>hibernate-entitymanager</artifactId>
	<version>4.1.7.Final</version>
</dependency>
<dependency>
	<groupId>org.hsqldb</groupId>
	<artifactId>hsqldb</artifactId>
	<version>2.2.8</version>
</dependency>		
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.21</version>
</dependency>
```
##  Les classes en entité. 
Nous avons 5 classes en entité
1. Person.java
2. Home.java
3. ElectonicDevice.java
4. Heater.java
5. HybrideDevice

La classe Person.java contient 3 variables de type String pour les informations personnels avec son numero id identique et 
3 variables de type Collection puisque chaque person reste dans une residence,a des amis et a des machines electics
```
@Column(name="id_person")
	long id;
	private String nom;
	private String prenom;
	private String mail;
	private Collection<Home> residences;
	private Collection<Person> friends;
	private Collection<ElectronicDevice> devices;
```
