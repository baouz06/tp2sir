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

### La classe Person.java
La classe Person.java contient 3 variables du type String pour les informations personnelles avec son numéro id identique et 3 variables du type Collection puisque chaque person reste dans une ou plusieurs résidences et qu'il a une ou plusieurs amis et des équipements électroniques personnelles.

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

Pour toutes les variables que nous avons défini dans cette classe,on a fait l'imlementation des fonctions de getter et setter.
```
@Id @GeneratedValue
	public long getId() {
		return id;
	}

	public void setId(long id) {
		this.id = id;
	}

	public String getNom() {
		return nom;
	}

	public void setNom(String nom) {
		this.nom = nom;
	}
	.
	.
	.
	
```

Une personne peut avoir plusieurs amis et des équipements électroniques ainsi que des différentes résidences, c'est pour ça que dans les fonctions getter de ces trois variables, nous avons utilisé l'annotation "OneToMany".
```
@OneToMany(mappedBy="person_homme", cascade=CascadeType.PERSIST)
	public Collection<Home> getResidences() {
		return residences;
	}

	public void setResidences(Collection<Home> residences) {
		this.residences = residences;
	}
	@OneToMany(mappedBy="friends", cascade=CascadeType.PERSIST)
	public Collection<Person> getFriends() {
		return friends;
	}

	public void setFriends(Collection<Person> friends) {
		this.friends = friends;
	}
	
	@OneToMany(mappedBy="personnes", cascade=CascadeType.PERSIST)
	public Collection<ElectronicDevice> getDevices() {
		return devices;
	}

	public void setDevices(Collection<ElectronicDevice> devices) {
		this.devices = devices;
	}
```

### La classe Home.java

Chaque résidence a une taille, un nombre de pièces, des chauffages, des équipements électroniques. Donc nous avons défini 3 variables du type long pour la taille, le nombre de pièces et le numéro id identique et aussi pour des chauffages une variable du type Collection ainsi que pour savoir le propriétaire de la maison on a défini aussi une variable du type Person.
```
@Column(name="id_home")
	private long id;
	private long taille;
	private long nombre_de_piece ;
	private  Person person_homme;
	private Collection<Heater> chauffage;	
```

Pour toutes les variables que nous avons défini dans cette classe,on a fait l'implementation des fonctions de getter et setter.
```
@Id @GeneratedValue
	public long getId() {
		return id;
	}

	public void setId(long id) {
		this.id = id;
	}

	public long getTaille() {
		return taille;
	}

	public void setTaille(long taille) {
		this.taille = taille;
	}
	.
	.
	.
```

Nous avons utilisé l'annotation "OneToMany" puisqu'une résidence peut avoir plusieurs chauffages par contre comme chaque maison peut avoir un seul propriétaire nous avons choisi l'annotation de "ManyToOne" pour la fonction getperson_homme().

```
@OneToMany(mappedBy="homes", cascade=CascadeType.PERSIST)
	public Collection<Heater> getChauffage() {
		return chauffage;
	}

	public void setChauffage(Collection<Heater> chauffage) {
		this.chauffage = chauffage;
	}
	
	@ManyToOne
	public Person getPerson_homme() {
		return person_homme;
	}

	public void setPerson_homme(Person person_homme) {
		this.person_homme = person_homme;
	}
```

### La classe ElectronicDevice.java


Des équipements électroniques ont une consommation moyenne en Watt/h et une personne peut avoir plusierus équipements électroniques
c'est pour ça qu'on a défini une variable de type long pour la consomation et une variable de type Person,pour savoir les propriétaires de ces équipement.

```
	private long consommation;
	private Person personnes;
```

Nous avons fait l'imlementation des fonctions de getter et setter avec ces deux variables comme au dessous et on utilise l'annotation "ManyToOne" car une personne peut avoir plusieurs équipements électroniques

```
public long getCconsommation() {
		return consommation;
	}
	public void setCconsommation(long cconsommation) {
		this.consommation = cconsommation;
	}
	@ManyToOne
	public Person getPersonnes() {
		return personnes;
	}
	public void setPersonnes(Person personnes) {
		this.personnes = personnes;
	}
	
```

### La classe Heater.java

Pour la classe Heater.java nous avons besoin de trois variables pour savoir le nom et la consomattion de chauffage ainsi que le nom de la résidence ou chaque chauffage se trouve.

```
	private String nom_chauffage ;
	private String puissance;
	private Home homes; 
```
Nous avons fait l'imlementation des fonctions de getter et setter avec ces trois variables comme au dessous et on utilise l'annotation "ManyToOne" car une résidence peut avoir plusieurs chauffages.

```
public String getNom_chauffage() {
		return nom_chauffage;
	}
	public void setNom_chauffage(String nom_chauffage) {
		this.nom_chauffage = nom_chauffage;
	}
	public String getPuissance() {
		return puissance;
	}
	public void setPuissance(String puissance) {
		this.puissance = puissance;
	}
	
	@ManyToOne
	public Home getHomes() {
		return homes;
	}
	public void setHomes(Home homes) {
		this.homes = homes;
	}
```

### La classe HybrideDevice.java

Pour les classes Heater.java et ElectronicDevice.java, on a utilisé la commande "extends HybrideDevice" pour l'héritage. Donc dans notre base de données, on arrive à voir ces deux classes différentes en même colonne avec le champ HybrideDevice grace à la variable id qu'on a défini dans cette classe.

```
public HybrideDevice() {
	// TODO Auto-generated constructor stub
}
public HybrideDevice(String name) {
	super();
	this.name = name;
}
@Id @GeneratedValue
public long getId() {
	return id;
}
public void setId(long id) {
	this.id = id;
}
public String getName() {
	return name;
}
public void setName(String name) {
	this.name = name;
```

