# Java

Java est un langage orienté objet, à typage statique, compilé en **bytecode** exécuté par la **JVM** (Java Virtual Machine). Sa devise historique — _« write once, run anywhere »_ — tient au fait que le même bytecode tourne sur toute plateforme disposant d'une JVM.

Cet aide-mémoire survole rapidement les fondamentaux orientés objet, puis se concentre sur l'usage réel en **web full-stack** : API REST, persistance et authentification avec **Spring Boot**, le standard de fait de l'écosystème.

## JDK, JRE, JVM

| Composant | Rôle |
|---|---|
| **JVM** | Machine virtuelle qui exécute le bytecode `.class` |
| **JRE** | JVM + bibliothèques standard (pour _exécuter_ une application) |
| **JDK** | JRE + outils de compilation (`javac`, `jar`, `javadoc`…) — nécessaire pour _développer_ |

### Versions LTS

Java suit un rythme de deux versions par an, avec une version **LTS** (Long Term Support) tous les 2-3 ans. En 2026, les LTS de référence sont **Java 21** (2023, très répandue en production) et **Java 25** (2025, la plus récente). Spring Boot 4.x exige au minimum **Java 17**.

```bash
java --version     # version d'exécution
javac --version    # version du compilateur
```

> Pour gérer plusieurs JDK sur une machine, utiliser [SDKMAN!](https://sdkman.io/) (Linux/macOS) ou des distributions comme Temurin (Eclipse Adoptium), Amazon Corretto, Azul Zulu.

---

## Fondamentaux OOP (rapide)

La programmation orientée objet consiste à modéliser le domaine sous forme d'**objets** qui regroupent des données (attributs) et les comportements qui les manipulent (méthodes). En Java, tout code vit dans une **classe** : la classe est le plan (le modèle), l'objet en est une instance concrète créée avec `new`.

```java
// Fichier Personne.java — le nom du fichier doit correspondre à la classe publique
public class Personne {
    // Attributs (encapsulés : private)
    private String nom;
    private int age;

    // Constructeur
    public Personne(String nom, int age) {
        this.nom = nom;
        this.age = age;
    }

    // Getters / setters (accès contrôlé)
    public String getNom() { return nom; }
    public void setAge(int age) { this.age = age; }

    // Méthode
    public String saluer() {
        return "Bonjour, je suis " + nom;
    }
}
```

### Les 4 piliers

| Pilier | Principe | Mécanisme Java |
|---|---|---|
| **Encapsulation** | Cacher l'état interne, exposer un contrat | `private` + getters/setters |
| **Héritage** | Réutiliser et spécialiser une classe | `extends` |
| **Polymorphisme** | Un même appel, plusieurs comportements | `@Override`, surcharge |
| **Abstraction** | Définir un contrat sans implémentation | `abstract`, `interface` |

### Héritage et polymorphisme

L'**héritage** (`extends`) permet à une classe fille de réutiliser les attributs et méthodes de sa classe mère, puis de les compléter ou de les redéfinir. Le **polymorphisme** est la conséquence : une variable de type `Personne` peut référencer un `Employe`, et l'appel de `saluer()` exécutera la version la plus spécifique (celle de `Employe`). C'est ce qui permet d'écrire du code générique qui s'adapte au type réel de l'objet à l'exécution.

```java
public class Employe extends Personne {
    private double salaire;

    public Employe(String nom, int age, double salaire) {
        super(nom, age);        // appel du constructeur parent
        this.salaire = salaire;
    }

    @Override
    public String saluer() {    // redéfinition (polymorphisme)
        return super.saluer() + ", employé";
    }
}
```

### Interfaces

Une interface définit un contrat. Une classe peut en implémenter plusieurs (là où l'héritage de classe est unique).

```java
public interface Notifiable {
    void notifier(String message);           // méthode abstraite

    default void notifierUrgent(String m) {  // méthode par défaut (Java 8+)
        notifier("[URGENT] " + m);
    }
}

public class Utilisateur extends Personne implements Notifiable {
    @Override
    public void notifier(String message) {
        System.out.println(message);
    }
}
```

### `record` — classes de données immuables (Java 16+)

Idéal pour les **DTO** : génère automatiquement constructeur, getters, `equals`, `hashCode` et `toString`.

```java
public record ArticleDTO(Long id, String titre, double prix) {}

var dto = new ArticleDTO(1L, "Clavier", 49.9);
dto.titre();   // "Clavier" — accesseur généré (sans préfixe get)
```

### `enum`

Une énumération définit un ensemble fermé et fini de valeurs constantes nommées. Elle remplace avantageusement les chaînes ou les entiers « magiques » : le compilateur garantit qu'on ne peut utiliser qu'une des valeurs prévues, ce qui élimine toute une classe d'erreurs.

```java
public enum Role {
    USER, ADMIN, MODERATOR;
}

Role r = Role.ADMIN;
if (r == Role.ADMIN) { /* ... */ }
```

---

## Essentiel du langage moderne

Ce que l'on croise réellement dans un backend Java.

### Types

Java distingue deux familles de types. Les **primitifs** (`int`, `double`, `boolean`…) stockent directement leur valeur et ne peuvent pas être `null`. Les **objets** (toute classe, dont `String` et les wrappers comme `Integer`) sont manipulés par référence et peuvent valoir `null`. Le typage étant statique, le type de chaque variable est fixé et vérifié à la compilation.

```java
// Primitifs (minuscule) : stockés par valeur
int n = 42;
long grand = 10_000_000_000L;
double prix = 19.99;
boolean actif = true;
char c = 'A';

// Objets (majuscule) : références, peuvent être null
String texte = "Bonjour";
Integer boxed = 42;            // wrapper autour de int

// Inférence de type (Java 10+)
var liste = new ArrayList<String>();  // type déduit
```

### Collections

Le _framework_ Collections fournit les structures de données courantes, chacune avec ses garanties. `List` conserve l'ordre d'insertion et autorise les doublons ; `Set` interdit les doublons ; `Map` associe des clés à des valeurs. On programme contre l'**interface** (`List`, `Set`, `Map`) et on choisit l'**implémentation** (`ArrayList`, `HashSet`, `HashMap`) selon les besoins de performance.

```java
import java.util.*;

List<String> liste = new ArrayList<>();       // liste ordonnée, doublons OK
liste.add("a"); liste.add("b");

Set<String> ensemble = new HashSet<>();       // pas de doublons
Map<String, Integer> map = new HashMap<>();   // clé -> valeur
map.put("age", 30);
map.getOrDefault("taille", 0);                // valeur par défaut si absente

// Collections immuables (Java 9+)
List<String> fixe = List.of("x", "y", "z");
Map<String, Integer> config = Map.of("port", 8080);
```

### Generics

Les generics permettent de paramétrer une classe ou une méthode par un type (`<T>`) fixé au moment de l'utilisation. Le bénéfice : le compilateur vérifie la cohérence des types et l'on évite les transtypages (`cast`) manuels et leurs erreurs à l'exécution. C'est ce qui fait qu'une `List<String>` n'accepte que des chaînes.

```java
public class Boite<T> {
    private T contenu;
    public void ranger(T item) { this.contenu = item; }
    public T recuperer() { return contenu; }
}

Boite<String> b = new Boite<>();
b.ranger("secret");   // n'accepte que des String
```

### Streams et lambdas

Une **lambda** est une fonction anonyme écrite de façon concise (`e -> e.getNom()`), que l'on passe en argument comme une valeur. L'**API Stream** enchaîne des opérations sur une collection de manière déclarative : on décrit *quoi* faire (filtrer, transformer, agréger) plutôt que *comment* le faire avec des boucles. Les opérations intermédiaires (`filter`, `map`, `sorted`) sont paresseuses et ne s'exécutent qu'au déclenchement d'une opération terminale (`collect`, `sum`, `toList`).

```java
import java.util.stream.*;

List<Employe> employes = /* ... */;

// Filtrer, transformer, collecter
List<String> nomsCadres = employes.stream()
    .filter(e -> e.getSalaire() > 50000)     // lambda
    .map(Employe::getNom)                     // référence de méthode
    .sorted()
    .collect(Collectors.toList());

// Agrégations
double masseSalariale = employes.stream()
    .mapToDouble(Employe::getSalaire)
    .sum();

// Regrouper
Map<Role, List<Employe>> parRole = employes.stream()
    .collect(Collectors.groupingBy(Employe::getRole));
```

### `Optional` — éviter les `NullPointerException`

`Optional<T>` est un conteneur qui représente explicitement « une valeur ou son absence ». Plutôt que de renvoyer `null` (qu'un appelant risque d'oublier de tester, provoquant une `NullPointerException`), une méthode renvoie un `Optional` : le type force alors à traiter le cas « absent » de façon visible.

```java
Optional<Utilisateur> resultat = repository.findByEmail(email);

Utilisateur u = resultat.orElseThrow(
    () -> new UtilisateurIntrouvableException(email)
);

// Ou une valeur par défaut
String nom = resultat.map(Utilisateur::getNom).orElse("Anonyme");
```

### Gestion des exceptions

Une exception est un objet signalant une erreur qui interrompt le flux normal ; elle « remonte » la pile d'appels jusqu'à un bloc `catch` capable de la traiter. Java distingue deux familles : les exceptions **vérifiées** (checked), que le compilateur oblige à gérer ou à déclarer, et les **non vérifiées** (unchecked, issues de `RuntimeException`), qui traduisent le plus souvent un bug de programmation.

```java
try {
    var contenu = Files.readString(Path.of("config.txt"));
} catch (IOException e) {
    log.error("Lecture impossible", e);
    throw new RuntimeException(e);
} finally {
    // toujours exécuté
}

// try-with-resources : ferme automatiquement les ressources
try (var reader = Files.newBufferedReader(path)) {
    return reader.readLine();
}   // reader.close() appelé automatiquement
```

| Type d'exception | Vérifiée à la compilation ? | Exemple |
|---|---|---|
| **Checked** (`Exception`) | Oui — `throws` ou `try/catch` obligatoire | `IOException`, `SQLException` |
| **Unchecked** (`RuntimeException`) | Non | `NullPointerException`, `IllegalArgumentException` |

---

## Outillage & build

Un projet Java repose sur un **outil de build** qui compile le code, télécharge les dépendances depuis un dépôt central, exécute les tests et produit un artefact livrable (un `.jar`). Les deux standards sont **Maven** (configuration XML déclarative) et **Gradle** (scripts plus concis). Tous deux suivent une convention d'arborescence identique.

### Structure d'un projet Maven

```
mon-projet/
├── pom.xml                          # configuration du build
└── src/
    ├── main/
    │   ├── java/                    # code source
    │   │   └── com/exemple/app/
    │   │       ├── Application.java
    │   │       ├── controller/
    │   │       ├── service/
    │   │       ├── repository/
    │   │       └── model/
    │   └── resources/               # config, templates
    │       └── application.yml
    └── test/
        └── java/                    # tests
```

### Maven — `pom.xml` annoté

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
  <modelVersion>4.0.0</modelVersion>

  <!-- Hérite des versions gérées par Spring Boot -->
  <!-- Ligne courante en 2026 : 4.x (baseline Java 17). 3.x reste très répandue ;
       le code de cette fiche s'applique aux deux (namespace jakarta.*). -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.0</version>
  </parent>

  <!-- Coordonnées du projet -->
  <groupId>com.exemple</groupId>
  <artifactId>app</artifactId>
  <version>1.0.0</version>

  <properties>
    <java.version>21</java.version>
  </properties>

  <dependencies>
    <!-- Un "starter" tire toutes les dépendances cohérentes d'un besoin -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>          <!-- disponible en test uniquement -->
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

### Commandes Maven essentielles

```bash
mvn clean            # supprime le dossier target/
mvn compile          # compile le code
mvn test             # exécute les tests
mvn package          # produit le .jar (dans target/)
mvn install          # installe le .jar dans le dépôt local
mvn spring-boot:run  # lance l'application (dev)
```

### Maven vs Gradle

| Aspect | Maven (`pom.xml`) | Gradle (`build.gradle`) |
|---|---|---|
| **Format** | XML déclaratif | Groovy / Kotlin DSL |
| **Verbosité** | Élevée | Concise |
| **Performance** | Correcte | Meilleure (cache, build incrémental) |
| **Déclarer une dépendance** | `<dependency>…</dependency>` | `implementation 'groupe:artefact:version'` |
| **Lancer** | `mvn spring-boot:run` | `./gradlew bootRun` |
| **Écosystème** | Majoritaire, très stable | Populaire (Android, gros projets) |

```groovy
// build.gradle — équivalent Gradle du pom.xml ci-dessus
plugins {
    id 'org.springframework.boot' version '3.4.0'
    id 'io.spring.dependency-management' version '1.1.6'
    id 'java'
}

java { sourceCompatibility = '21' }

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

## Spring Boot — les bases

Spring Boot simplifie radicalement la configuration de Spring via l'**auto-configuration** (configure automatiquement selon les dépendances présentes) et les **starters** (regroupements de dépendances cohérentes).

### Point d'entrée

Une application Spring Boot est une classe Java classique avec un `main`. Au démarrage, `SpringApplication.run(...)` lance le **conteneur Spring** (l'_ApplicationContext_), scanne le projet à la recherche des composants, les instancie et démarre un serveur web embarqué (Tomcat par défaut) — aucun serveur externe à installer.

```java
package com.exemple.app;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication   // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### Injection de dépendances (IoC)

Le principe d'**inversion de contrôle** (IoC) consiste à confier à Spring la création et l'assemblage des objets plutôt que de les instancier soi-même avec `new`. Chaque classe annotée devient un **bean** géré par le conteneur ; quand une classe déclare avoir besoin d'une autre, Spring la lui **injecte** automatiquement. Résultat : les composants sont faiblement couplés, remplaçables et testables isolément.

On annote les classes selon leur rôle, et on injecte les dépendances via le **constructeur** (méthode recommandée).

| Annotation | Couche |
|---|---|
| `@RestController` | Exposition HTTP (API REST) |
| `@Service` | Logique métier |
| `@Repository` | Accès aux données |
| `@Component` | Composant générique |
| `@Configuration` | Classe de configuration / beans |

```java
@Service
public class ArticleService {
    private final ArticleRepository repository;   // dépendance

    // Injection par constructeur : pas besoin de @Autowired si un seul constructeur
    public ArticleService(ArticleRepository repository) {
        this.repository = repository;
    }

    public List<Article> lister() {
        return repository.findAll();
    }
}
```

> **Pourquoi le constructeur plutôt que `@Autowired` sur le champ ?** Il rend les dépendances explicites, permet de les déclarer `final` (immuables) et facilite les tests unitaires (on passe des mocks directement).

### Configuration — `application.yml`

Toute la configuration externe de l'application (port, base de données, secrets…) se déclare dans `application.yml` (ou `.properties`), au lieu d'être codée en dur. La syntaxe `${VARIABLE}` récupère une variable d'environnement : les valeurs sensibles restent ainsi hors du code source et du dépôt Git.

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/madb
    username: ${DB_USER}          # variable d'environnement
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: update            # validate | update | create | create-drop
    show-sql: true

app:
  jwt:
    secret: ${JWT_SECRET}
    expiration: 900000            # 15 min en ms
```

### Profils (environnements)

```yaml
# application.yml — profil actif
spring:
  profiles:
    active: dev
```

Chaque profil a son fichier : `application-dev.yml`, `application-prod.yml`. Activation au lancement :

```bash
java -jar app.jar --spring.profiles.active=prod
```

```java
// Lire une valeur de config dans le code
@Value("${app.jwt.expiration}")
private long expiration;
```

---

## API REST

Une API REST expose les ressources de l'application (articles, utilisateurs…) via des URL et les verbes HTTP standard (GET, POST, PUT, DELETE). Dans Spring, un **contrôleur** reçoit la requête, délègue le traitement à un service, et renvoie un objet que Spring sérialise automatiquement en JSON.

### Contrôleur REST

Annoté `@RestController`, chaque méthode est associée à une route et à un verbe. Les valeurs de retour sont converties en JSON (via Jackson), et les paramètres de la requête sont désérialisés en objets Java par les annotations `@PathVariable`, `@RequestParam` et `@RequestBody`.

```java
@RestController
@RequestMapping("/api/articles")    // préfixe commun à toutes les routes
public class ArticleController {

    private final ArticleService service;

    public ArticleController(ArticleService service) {
        this.service = service;
    }

    @GetMapping                                   // GET /api/articles
    public List<ArticleDTO> lister() {
        return service.lister();
    }

    @GetMapping("/{id}")                          // GET /api/articles/42
    public ArticleDTO obtenir(@PathVariable Long id) {
        return service.obtenir(id);
    }

    @PostMapping                                  // POST /api/articles
    @ResponseStatus(HttpStatus.CREATED)           // 201
    public ArticleDTO creer(@Valid @RequestBody CreerArticleDTO dto) {
        return service.creer(dto);
    }

    @PutMapping("/{id}")                          // PUT /api/articles/42
    public ArticleDTO modifier(@PathVariable Long id,
                               @Valid @RequestBody CreerArticleDTO dto) {
        return service.modifier(id, dto);
    }

    @DeleteMapping("/{id}")                       // DELETE /api/articles/42
    @ResponseStatus(HttpStatus.NO_CONTENT)        // 204
    public void supprimer(@PathVariable Long id) {
        service.supprimer(id);
    }

    // Paramètres de requête : GET /api/articles/recherche?q=clavier&page=0
    @GetMapping("/recherche")
    public List<ArticleDTO> rechercher(@RequestParam String q,
                                       @RequestParam(defaultValue = "0") int page) {
        return service.rechercher(q, page);
    }
}
```

### Mapping des annotations HTTP

| Annotation | Verbe HTTP | Usage typique |
|---|---|---|
| `@GetMapping` | GET | Lire |
| `@PostMapping` | POST | Créer |
| `@PutMapping` | PUT | Remplacer |
| `@PatchMapping` | PATCH | Modifier partiellement |
| `@DeleteMapping` | DELETE | Supprimer |
| `@PathVariable` | — | Variable dans l'URL (`/{id}`) |
| `@RequestParam` | — | Paramètre de requête (`?q=`) |
| `@RequestBody` | — | Corps JSON désérialisé en objet |

### Validation des entrées

Spring intègre **Jakarta Bean Validation**. On annote le DTO, et `@Valid` déclenche la vérification.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

```java
public record CreerArticleDTO(
    @NotBlank(message = "Le titre est obligatoire")
    String titre,

    @Positive(message = "Le prix doit être positif")
    double prix,

    @Email(message = "Email invalide")
    String contactEmail
) {}
```

| Annotation | Vérifie |
|---|---|
| `@NotNull` / `@NotBlank` / `@NotEmpty` | Non nul / chaîne non vide / collection non vide |
| `@Size(min=, max=)` | Longueur d'une chaîne ou collection |
| `@Min` / `@Max` / `@Positive` | Bornes numériques |
| `@Email` | Format e-mail |
| `@Pattern(regexp=)` | Expression régulière |

### Gestion centralisée des erreurs

Plutôt que de répéter des `try/catch` dans chaque contrôleur, on centralise le traitement des exceptions dans une classe `@RestControllerAdvice`. Elle intercepte les exceptions levées par n'importe quel contrôleur et les traduit en réponses HTTP cohérentes (bon code de statut, corps d'erreur uniforme). La logique métier peut ainsi lever une exception « métier » sans se soucier du format de la réponse.

```java
@RestControllerAdvice
public class GestionnaireErreurs {

    // Exception métier -> 404
    @ExceptionHandler(RessourceIntrouvableException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErreurReponse introuvable(RessourceIntrouvableException e) {
        return new ErreurReponse("NOT_FOUND", e.getMessage());
    }

    // Échec de validation -> 400 avec le détail des champs
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> validation(MethodArgumentNotValidException e) {
        Map<String, String> erreurs = new HashMap<>();
        e.getBindingResult().getFieldErrors().forEach(err ->
            erreurs.put(err.getField(), err.getDefaultMessage()));
        return erreurs;
    }
}

public record ErreurReponse(String code, String message) {}
```

### Codes HTTP courants

| Code | Signification | Quand |
|---|---|---|
| `200 OK` | Succès | GET, PUT réussis |
| `201 Created` | Créé | POST réussi |
| `204 No Content` | Succès sans corps | DELETE réussi |
| `400 Bad Request` | Requête invalide | Validation échouée |
| `401 Unauthorized` | Non authentifié | Token absent/invalide |
| `403 Forbidden` | Non autorisé | Droits insuffisants |
| `404 Not Found` | Introuvable | Ressource inexistante |
| `500 Internal Server Error` | Erreur serveur | Exception non gérée |

---

## Persistance — Spring Data JPA / Hibernate

**JPA** (Jakarta Persistence API) est la spécification de mapping objet-relationnel (ORM) ; **Hibernate** en est l'implémentation par défaut ; **Spring Data JPA** ajoute une couche qui génère les repositories automatiquement.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <scope>runtime</scope>
</dependency>
```

### Entité

Une **entité** est une classe Java dont chaque instance correspond à une ligne d'une table. Les annotations décrivent le mapping : `@Entity` marque la classe comme persistante, `@Id` désigne la clé primaire, `@Column` règle les détails de colonne. Hibernate se charge alors de traduire les opérations sur ces objets en SQL.

```java
import jakarta.persistence.*;

@Entity
@Table(name = "articles")
public class Article {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)   // auto-incrément
    private Long id;

    @Column(nullable = false, length = 200)
    private String titre;

    private double prix;

    @Enumerated(EnumType.STRING)     // stocke "DISPONIBLE" plutôt que 0
    private Statut statut;

    @Column(name = "cree_le", updatable = false)
    private Instant creeLe = Instant.now();

    // JPA exige un constructeur sans argument
    protected Article() {}

    // + constructeur, getters, setters
}
```

### Relations

Les associations entre tables (clés étrangères) se traduisent par des références entre entités. Les annotations `@ManyToOne`, `@OneToMany` et `@ManyToMany` décrivent la cardinalité ; `cascade` propage les opérations aux entités liées (ex. sauvegarder une commande sauvegarde ses lignes), et `orphanRemoval` supprime les enfants détachés de leur parent.

```java
@Entity
public class Commande {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Plusieurs commandes -> un client
    @ManyToOne(fetch = FetchType.LAZY)     // LAZY : chargé à la demande
    @JoinColumn(name = "client_id")
    private Client client;

    // Une commande -> plusieurs lignes
    @OneToMany(mappedBy = "commande", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<LigneCommande> lignes = new ArrayList<>();
}
```

| Annotation | Cardinalité |
|---|---|
| `@OneToOne` | 1 ↔ 1 |
| `@OneToMany` / `@ManyToOne` | 1 ↔ N |
| `@ManyToMany` | N ↔ N (table de jointure) |

> ⚠️ Préférer `FetchType.LAZY` par défaut pour ne charger une relation qu'au moment où on y accède réellement. Le chargement `EAGER` non maîtrisé cause le problème classique des **N+1 requêtes** : parcourir une liste de N entités déclenche 1 requête pour la liste, puis 1 requête par entité pour charger sa relation. On le résout avec une jointure explicite (`JOIN FETCH` ou `@EntityGraph`).

### Repository

Un repository est la couche d'accès aux données. Avec Spring Data JPA, il suffit de déclarer une **interface** étendant `JpaRepository` : Spring en génère l'implémentation au démarrage. Mieux, il déduit la requête du **nom de la méthode** (`findByStatut`, `existsByTitre`…) selon une convention de nommage — aucune ligne de SQL pour les cas courants.

```java
public interface ArticleRepository extends JpaRepository<Article, Long> {

    // Requêtes DÉRIVÉES : Spring déduit le SQL du nom de la méthode
    List<Article> findByStatut(Statut statut);
    List<Article> findByTitreContainingIgnoreCase(String fragment);
    Optional<Article> findByTitre(String titre);
    boolean existsByTitre(String titre);
    long countByStatut(Statut statut);

    // Requête explicite (JPQL) quand la dérivation ne suffit pas
    @Query("SELECT a FROM Article a WHERE a.prix BETWEEN :min AND :max")
    List<Article> trouverParFourchette(double min, double max);

    // SQL natif si besoin
    @Query(value = "SELECT * FROM articles WHERE prix > ?1", nativeQuery = true)
    List<Article> plusChersQue(double prix);
}
```

`JpaRepository` fournit déjà : `findAll()`, `findById()`, `save()`, `deleteById()`, `count()`, la pagination et le tri.

### DTO et mapping

Ne jamais exposer directement les entités JPA dans l'API (fuite de structure, cycles de sérialisation, couplage). On convertit vers des DTO.

```java
@Service
public class ArticleService {
    private final ArticleRepository repository;

    public ArticleService(ArticleRepository repository) {
        this.repository = repository;
    }

    @Transactional(readOnly = true)
    public List<ArticleDTO> lister() {
        return repository.findAll().stream()
            .map(this::versDTO)
            .toList();
    }

    @Transactional
    public ArticleDTO creer(CreerArticleDTO dto) {
        Article a = new Article(dto.titre(), dto.prix());
        return versDTO(repository.save(a));
    }

    private ArticleDTO versDTO(Article a) {
        return new ArticleDTO(a.getId(), a.getTitre(), a.getPrix());
    }
}
```

> `@Transactional` délimite la transaction : tout réussit, ou tout est annulé (rollback). Utiliser `readOnly = true` pour les lectures (optimisation).

---

## Authentification & sécurité

Spring Security intercepte les requêtes via une **chaîne de filtres** (filter chain) avant qu'elles n'atteignent les contrôleurs. On l'utilise ici pour une authentification **JWT stateless**, adaptée aux API et aux SPA.

> Cette section applique les concepts JWT (structure du token, access/refresh, stockage) détaillés dans la fiche [JWT — Authentification](../dev-securite/jwt-auth.md). On se concentre ici sur l'intégration Spring.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Hachage des mots de passe (BCrypt)

On ne stocke jamais un mot de passe en clair : on en conserve une **empreinte** produite par une fonction de hachage à sens unique, impossible à inverser. BCrypt est conçu spécifiquement pour cela — il est volontairement lent (pour résister aux attaques par force brute) et intègre un **sel** aléatoire, de sorte que deux mots de passe identiques donnent des empreintes différentes. La vérification se fait par `matches`, sans jamais reconstituer le mot de passe.

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

```java
// À l'inscription : hacher avant de sauvegarder
String hash = passwordEncoder.encode(motDePasseEnClair);

// À la connexion : comparer
boolean ok = passwordEncoder.matches(motDePasseEnClair, hashStocke);
```

### Configuration de la sécurité

La `SecurityFilterChain` définit les règles appliquées à chaque requête entrante : quelles routes sont publiques, lesquelles exigent une authentification, lesquelles réclament un rôle précis. Pour une API JWT, on désactive les sessions (`STATELESS`) — chaque requête se ré-authentifie via son token, le serveur ne conserve aucun état — et on désactive la protection CSRF, inutile en l'absence de cookies de session.

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity        // active @PreAuthorize sur les méthodes
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;

    public SecurityConfig(JwtAuthFilter jwtAuthFilter) {
        this.jwtAuthFilter = jwtAuthFilter;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // API stateless : pas de session, pas besoin de CSRF
            .csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()      // login/register publics
                .requestMatchers(HttpMethod.GET, "/api/articles/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // réservé aux admins
                .anyRequest().authenticated()                      // le reste : authentifié
            )
            // Insérer notre filtre JWT avant le filtre d'authentification standard
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

### Génération du token (service JWT)

Ce service encapsule la création et la vérification des JWT avec la bibliothèque `jjwt`. À la connexion, il signe un token contenant l'identité de l'utilisateur et ses claims (rôle, expiration) ; à chaque requête suivante, il vérifie la signature avec le secret pour garantir que le token n'a pas été falsifié.

```java
@Service
public class JwtService {

    @Value("${app.jwt.secret}")
    private String secret;

    @Value("${app.jwt.expiration}")
    private long expiration;

    public String genererToken(Utilisateur u) {
        return Jwts.builder()
            .subject(u.getEmail())
            .claim("role", u.getRole().name())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(cle())
            .compact();
    }

    public String extraireEmail(String token) {
        return parser(token).getSubject();
    }

    public boolean estValide(String token) {
        try {
            parser(token);
            return true;
        } catch (JwtException e) {
            return false;
        }
    }

    private Claims parser(String token) {
        return Jwts.parser().verifyWith(cle()).build()
            .parseSignedClaims(token).getPayload();
    }

    private SecretKey cle() {
        return Keys.hmacShaKeyFor(secret.getBytes());
    }
}
```

### Endpoint de connexion

La route de login est publique. Elle vérifie l'e-mail et compare le mot de passe fourni au hash stocké (via `matches`, sans jamais déchiffrer). Si les identifiants sont valides, elle renvoie un JWT que le client joindra à ses requêtes suivantes.

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final UtilisateurRepository repository;
    private final PasswordEncoder encoder;
    private final JwtService jwtService;

    // + constructeur

    @PostMapping("/login")
    public TokenDTO login(@Valid @RequestBody LoginDTO dto) {
        Utilisateur u = repository.findByEmail(dto.email())
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.UNAUTHORIZED));

        if (!encoder.matches(dto.motDePasse(), u.getMotDePasse())) {
            throw new ResponseStatusException(HttpStatus.UNAUTHORIZED, "Identifiants invalides");
        }

        return new TokenDTO(jwtService.genererToken(u), "Bearer");
    }
}

public record LoginDTO(@Email String email, @NotBlank String motDePasse) {}
public record TokenDTO(String accessToken, String tokenType) {}
```

### Filtre de validation JWT

Ce filtre s'exécute à chaque requête : il lit le header `Authorization`, valide le token et place l'utilisateur dans le contexte de sécurité.

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;

    public JwtAuthFilter(JwtService jwtService) {
        this.jwtService = jwtService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain)
            throws ServletException, IOException {

        String header = request.getHeader("Authorization");

        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);

            if (jwtService.estValide(token)) {
                String email = jwtService.extraireEmail(token);
                var auth = new UsernamePasswordAuthenticationToken(
                    email, null, List.of());   // + autorités/rôles
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }

        chain.doFilter(request, response);   // laisser passer la requête
    }
}
```

### Autorisation fine par méthode

Là où la `SecurityFilterChain` filtre par URL, `@PreAuthorize` sécurise au niveau de la méthode. Son expression (langage SpEL) est évaluée avant l'exécution : elle peut vérifier un rôle, mais aussi une condition dynamique — par exemple qu'un utilisateur n'accède qu'à sa propre ressource.

```java
@RestController
@RequestMapping("/api/admin")
public class AdminController {

    @GetMapping("/stats")
    @PreAuthorize("hasRole('ADMIN')")        // vérifié avant l'appel
    public StatsDTO stats() { /* ... */ }

    // Accès à sa propre ressource uniquement
    @GetMapping("/users/{id}")
    @PreAuthorize("#id == authentication.principal.id or hasRole('ADMIN')")
    public UserDTO utilisateur(@PathVariable Long id) { /* ... */ }
}
```

### CORS (appels depuis un front sur un autre domaine)

Par sécurité, un navigateur bloque par défaut les requêtes JavaScript vers une origine (domaine, port ou protocole) différente de celle de la page. Le **CORS** (Cross-Origin Resource Sharing) est le mécanisme par lequel le serveur autorise explicitement certaines origines — indispensable quand le front (ex. `localhost:5173`) et l'API (ex. `localhost:8080`) sont sur des origines distinctes.

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    var config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("https://mon-front.fr"));  // pas "*" en prod
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
    config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
    config.setAllowCredentials(true);

    var source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return source;
}
```

### Flux d'une requête authentifiée

```
Client                          Serveur (Spring Security)
  │── POST /api/auth/login ───────→│  vérifie identifiants (BCrypt)
  │←── { accessToken } ────────────│  génère le JWT
  │                                │
  │── GET /api/articles ───────────→│  ┌─ JwtAuthFilter
  │   Authorization: Bearer <jwt>  │  │   valide le token
  │                                │  │   place l'user dans le contexte
  │                                │  └─ authorizeHttpRequests / @PreAuthorize
  │←── 200 { données } ────────────│  contrôleur exécuté si autorisé
  │                                │
  │── GET /api/admin/stats ────────→│  403 si rôle insuffisant
```

---

## Tests

Spring Boot fournit un starter de test complet (JUnit 5, Mockito, AssertJ, MockMvc).

### Test unitaire (service isolé avec Mockito)

Un test unitaire vérifie une classe isolée de ses dépendances. **Mockito** fournit des **mocks** : de fausses implémentations dont on programme le comportement (`when(...).thenReturn(...)`) pour tester la logique de la classe sans base de données ni contexte Spring. Le patron **given / when / then** structure le test : contexte, action, vérification.

```java
@ExtendWith(MockitoExtension.class)
class ArticleServiceTest {

    @Mock
    private ArticleRepository repository;     // dépendance simulée

    @InjectMocks
    private ArticleService service;           // classe testée

    @Test
    void lister_retourne_les_articles() {
        // given
        when(repository.findAll()).thenReturn(List.of(
            new Article("Clavier", 49.9)
        ));

        // when
        var resultat = service.lister();

        // then
        assertThat(resultat).hasSize(1);
        assertThat(resultat.get(0).titre()).isEqualTo("Clavier");
    }
}
```

### Test de contrôleur (couche web isolée)

`@WebMvcTest` ne charge que la couche web (le contrôleur ciblé, la sérialisation, la sécurité), en simulant les services sous-jacents. `MockMvc` envoie des requêtes HTTP simulées sans démarrer de vrai serveur, ce qui rend ces tests rapides tout en vérifiant routes, codes de statut et JSON produit.

```java
@WebMvcTest(ArticleController.class)     // charge seulement la couche web
class ArticleControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockitoBean                          // remplace le service par un mock
    private ArticleService service;

    @Test
    void obtenir_retourne_200() throws Exception {
        when(service.obtenir(1L)).thenReturn(new ArticleDTO(1L, "Clavier", 49.9));

        mockMvc.perform(get("/api/articles/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.titre").value("Clavier"));
    }
}
```

### Test d'intégration (contexte complet)

```java
@SpringBootTest                          // charge tout le contexte Spring
@AutoConfigureMockMvc
class ApplicationIT {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void endpoint_protege_refuse_sans_token() throws Exception {
        mockMvc.perform(get("/api/admin/stats"))
            .andExpect(status().isUnauthorized());
    }
}
```

| Annotation | Portée | Vitesse |
|---|---|---|
| `@ExtendWith(MockitoExtension.class)` | Unitaire, sans Spring | Très rapide |
| `@WebMvcTest` | Couche web seule | Rapide |
| `@DataJpaTest` | Couche persistance seule | Rapide |
| `@SpringBootTest` | Application complète | Lente |

---

## Checklist — backend Java/Spring

| # | Vérification |
|---|---|
| 1 | Injection par constructeur, dépendances `final` |
| 2 | Entités JPA jamais exposées dans l'API — passer par des DTO (`record`) |
| 3 | Validation des entrées avec `@Valid` + annotations Jakarta |
| 4 | Gestion des erreurs centralisée (`@RestControllerAdvice`) |
| 5 | Relations JPA en `FetchType.LAZY` par défaut — attention au N+1 |
| 6 | `@Transactional(readOnly = true)` sur les lectures |
| 7 | Mots de passe hachés avec BCrypt — jamais en clair |
| 8 | Secrets (JWT, BDD) dans des variables d'environnement, pas dans le code |
| 9 | API stateless : sessions désactivées, JWT dans le header `Authorization` |
| 10 | CORS restreint aux origines connues (pas `*` en production) |
| 11 | Codes HTTP corrects (201 à la création, 204 sur DELETE, 401 vs 403) |
| 12 | Tests par couche : unitaire (Mockito) + web (`@WebMvcTest`) + intégration |
