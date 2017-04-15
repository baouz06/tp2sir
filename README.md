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

Chaque classe est une entité : donc on utilise l'annotation @Entity pour que JPA comprenne qu'il s'agit d'une entité et qu'il va créer une table portant le même nom que la classe.
Toutes les entités possèdent  un id ayant pour annotation @Id
### La classe Person.java
La classe Person.java contient 3 variables du type String pour les informations personnelles avec son numéro id identique et 3 variables du type Collection puisque chaque person reste dans une ou plusieurs résidences et qu'elle a une ou plusieurs amis et des équipements électroniques personnelles.

@Entity
public class Person{
@Column(name="id_person")
	long id;
	private String nom;
	private String prenom;
	private String mail;
	private Collection<Home> residences;
	private Collection<Person> friends;
	private Collection<ElectronicDevice> devices;
```
Remarque : toutes les annotations sont faites sur les getter et setters pour plus de compatibilité.
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

Une personne peut avoir plusieurs amis et des équipements électroniques ainsi que des différentes résidences, c'est pour ça que dans les fonctions getter de ces trois variables, nous avons utilisé l'annotation "OneToMany", person_homme est l'attribut de type Person dans la classe Home.
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

@Entity
public class Home  {
@Column(name="id_home")
	private long id;
	private long taille;
	private long nombre_de_piece ;
	private  Person person_homme;
	private Collection<Heater> chauffage;	

Nous avons utilisé l'annotation "OneToMany" puisqu'une résidence peut avoir plusieurs chauffages,mais une  maison ne peut avoir qu'un seul propriétaire donc nous avons choisi l'annotation "ManyToOne" pour la fonction getperson_homme().

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

### Même principe pour les autres classes


### La classe HybrideDevice.java

Les classes Heater.java et ElectronicDevice.java sont les classes filles de HybrideDevice.java.
pour réaliser l'heritage, lesdeux classes filles ne peuvent pas posséder un id. 
Donc les deux sous classes heritent de la classe  mere qui a comme annotation @Inheritance(strategy=InheritanceType.SINGLE_TABLE),ici nous avons choisi d'avoir une seule table (InheritanceType.SINGLE_TABLE) avec des réferencement au sous classes


@Entity
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)
public class HybrideDevice {
public HybrideDevice() {
	// TODO Auto-generated constructor stub
}
public HybrideDevice(String name) {
	super();
	this.name = name;
}
```

## Le Chargement des données dans la base 

Dans notre fichier  JpaTest java on définit une variable du type "EntityManager" qui nous permet d'utiliser les fonctions de "select","insert","update", etc. entre JPA, API et la base de données et la variable du type "EntityManagerFactory" nous permet de créer la classe entity Manager.

Pour mettre à jour les données régulièrement on fait toutes les actions dans la transaction.

```
public class JpaTest {
	private static EntityManager manager;
	public JpaTest(EntityManager manager) {
		this.manager = manager;
	}
	public static void main(String[] args) {
		EntityManagerFactory factory = Persistence
				.createEntityManagerFactory("example");
		EntityManager manager = factory.createEntityManager();
		EntityTransaction tx = manager.getTransaction();
		tx.begin();
```		

aprés la connexion à la base de données, on procède comme ceci:
```
 Person personne2 = new Person();
          personne2.setNom("Khaled");
          personne2.setPrenom("emre");
          personne2.setMail("email");
	 
	  Home home = new Home();
          home.setNombre_de_piece(2);
          home.setTaille(50);
          home.setPerson_homme(personne2);
          
	  ElectronicDevice electronicDevice = new ElectronicDevice();
          electronicDevice.setConsommation(100);
          electronicDevice.setPersonnes(personne2);
```
