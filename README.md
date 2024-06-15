# Spyne
Software Engineer I - Backend
To create the backend API in Java Spring Boot for the given assignment, we need to follow these steps:

1. **Project Setup:**
   - Initialize a new Spring Boot project.
   - Add necessary dependencies for Spring Web, Spring Data JPA, Spring Security, and a database like H2 or MySQL.

2. **Database Schema:**
   - Create entities for `User`, `Discussion`, `Comment`, and `Like`.
   - Define relationships between these entities (e.g., one-to-many, many-to-many).

3. **Service Layer:**
   - Implement services for user management, discussions, comments, and likes.

4. **Controller Layer:**
   - Implement REST controllers for each of the functionalities mentioned in the assignment.

5. **Security:**
   - Implement authentication and authorization using Spring Security.

6. **Postman Collection:**
   - Create a Postman collection to test the API endpoints.

7. **Documentation:**
   - Provide API documentation and create a README file for the GitHub repository.

Below is a high-level implementation plan:

### 1. Project Setup
Use Spring Initializr to create a new Spring Boot project with dependencies:
- Spring Web
- Spring Data JPA
- Spring Security
- H2 Database (for simplicity)

### 2. Database Schema
Define the entities and their relationships.

#### `User` Entity
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String mobileNo;
    private String email;
    
    @OneToMany(mappedBy = "user")
    private List<Discussion> discussions;

    // Getters and setters
}
```

#### `Discussion` Entity
```java
@Entity
public class Discussion {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String text;
    private String image;
    private LocalDateTime createdOn;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
    
    @ManyToMany
    @JoinTable(
        name = "discussion_hashtag",
        joinColumns = @JoinColumn(name = "discussion_id"),
        inverseJoinColumns = @JoinColumn(name = "hashtag_id")
    )
    private List<Hashtag> hashtags;

    @OneToMany(mappedBy = "discussion")
    private List<Comment> comments;

    @OneToMany(mappedBy = "discussion")
    private List<Like> likes;

    // Getters and setters
}
```

#### `Comment` Entity
```java
@Entity
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String text;
    private LocalDateTime createdOn;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    @ManyToOne
    @JoinColumn(name = "discussion_id")
    private Discussion discussion;

    // Getters and setters
}
```

#### `Like` Entity
```java
@Entity
public class Like {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    @ManyToOne
    @JoinColumn(name = "discussion_id")
    private Discussion discussion;

    // Getters and setters
}
```

#### `Hashtag` Entity
```java
@Entity
public class Hashtag {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToMany(mappedBy = "hashtags")
    private List<Discussion> discussions;

    // Getters and setters
}
```

### 3. Service Layer
Create service classes for each entity. For example:

#### UserService
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public User updateUser(Long id, User userDetails) {
        User user = userRepository.findById(id).orElseThrow(() -> new ResourceNotFoundException("User", "id", id));
        user.setName(userDetails.getName());
        user.setMobileNo(userDetails.getMobileNo());
        user.setEmail(userDetails.getEmail());
        return userRepository.save(user);
    }

    public void deleteUser(Long id) {
        User user = userRepository.findById(id).orElseThrow(() -> new ResourceNotFoundException("User", "id", id));
        userRepository.delete(user);
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public List<User> searchUsersByName(String name) {
        return userRepository.findByNameContaining(name);
    }
}
```

### 4. Controller Layer
Create REST controllers to handle HTTP requests.

#### UserController
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        return ResponseEntity.ok(userService.createUser(user));
    }

    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
        return ResponseEntity.ok(userService.updateUser(id, user));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.ok().build();
    }

    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        return ResponseEntity.ok(userService.getAllUsers());
    }

    @GetMapping("/search")
    public ResponseEntity<List<User>> searchUsersByName(@RequestParam String name) {
        return ResponseEntity.ok(userService.searchUsersByName(name));
    }
}
```

### 5. Security
Implement user authentication using Spring Security.

#### Security Configuration
```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()
            .antMatchers("/api/users/**").permitAll()
            .anyRequest().authenticated()
            .and()
            .httpBasic();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user").password("{noop}password").roles("USER")
            .and()
            .withUser("admin").password("{noop}admin").roles("ADMIN");
    }
}
```

### 6. Postman Collection
Create a Postman collection to test the API endpoints. Export the collection and provide the link in the README file.

### 7. Documentation
Provide detailed documentation in the README file of the GitHub repository.

### Complete Example Structure
Here's a complete structure for your project:

```
src
└── main
    ├── java
    │   └── com
    │       └── example
    │           └── backend
    │               ├── controller
    │               │   └── UserController.java
    │               ├── entity
    │               │   ├── User.java
    │               │   ├── Discussion.java
    │               │   ├── Comment.java
    │               │   ├── Like.java
    │               │   └── Hashtag.java
    │               ├── repository
    │               │   ├── UserRepository.java
    │               │   ├── DiscussionRepository.java
    │               │   ├── CommentRepository.java
    │               │   └── LikeRepository.java
    │               ├── service
    │               │   ├── UserService.java
    │               │   ├── DiscussionService.java
    │               │   ├── CommentService.java
    │               │   └── LikeService.java
    │               └── BackendApplication.java
    └── resources
        └── application.properties
```

### Submission
- Upload the project to a GitHub repository.
- Create a README file with instructions on how to run the project, API documentation, and the Postman collection link.
- Submit the GitHub repository link through the provided Google form. 

This setup provides a solid foundation for the backend API. Additional features like following users, viewing post counts, and more can be incrementally added following similar patterns.
