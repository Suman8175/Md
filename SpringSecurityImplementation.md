# Step 1 : OverView

* **Spring Security**: A powerful and highly customizable authentication and access-control framework.
* **OAuth JWT**: A secure and efficient way to handle authentication and authorization between different parties.
* **HttpOnly Cookie**: A cookie attribute that prevents client-side scripts from accessing the cookie.
* **AuthFilter**: A filter that intercepts requests and performs authentication and authorization checks.
* **Login Logout**: A mechanism to authenticate and de-authenticate users.
* **RefreshToken Access Token**: A technique to refresh the access token without requiring the user to re-authenticate.

## Step2 : Add dependency 
 1.`web`,  `lombock`, `validation`, `h2`, `jpa`, `oauth2 resource server`, `configuration-processor`

 2.`application.properties` :database
 ```yaml
spring.datasource.url=jdbc:postgresql://localhost:5432/auth
spring.datasource.username=suman
spring.datasource.password=password
spring.jpa.database=POSTGRESQL
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
 ```
 >Note give url,username,password of your own...it is not same for you

 ## Step :3  Store User using JPA

 1. Create a `UserInfoEntity` to store User details. 

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Entity
    @Table(name="users")
    public class User {
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        private Long userId;

        @Column(name = "USER_NAME")
        private String userName;

        @Column(nullable = false, name = "EMAIL_ID", unique = true)
        private String emailId;

        @Column(nullable = false, name = "PASSWORD")
        private String password;

        @Column(name = "MOBILE_NUMBER")
        private String mobileNumber;

        @Column(nullable = false, name = "ROLES")
        private String roles;
    }
    ```

2. Create an interface  `UserRepository` in `repository` package, to create `jpa-mapping` using hibernate which extends JPARepository. 

    ```java
    @Repository
    public interface UserRepository extends JpaRepository<User,Long> {
    }
    //<User,Long> means typeClass=User and Long is userId data type
    ```
3. Create a `UserConfig` class which implements `UserDetails` interface, which **provides core user information which is later encapsulated into Authentication objects.** or you can directly implement `UserDetails` in `User` entity both are same but we create seperate UserConfig class for separtaion on concern.

    ```java
        
    @RequiredArgsConstructor
    public class UserConfig implements UserDetails {

        private final User user;
        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            return Arrays
                    .stream(user
                            .getRoles()
                            .split(","))
                    .map(SimpleGrantedAuthority::new)
                    .toList();
        }

        @Override
        public String getPassword() {
            return user.getPassword();
        }

        @Override
        public String getUsername() {
            return user.getEmailId();
        }

        @Override
        public boolean isAccountNonExpired() {
            return true;
        }

        @Override
        public boolean isAccountNonLocked() {
            return true;
        }

        @Override
        public boolean isCredentialsNonExpired() {
            return true;
        }

        @Override
        public boolean isEnabled() {
            return true;
        }
    }
    ```
    > Note:Remember to import User as entity which you made..not others your import should have `import {your package name}.user`

4. Create a `UserManagerConfig` class that implements the `UserDetailsService` interface, used to **retrieve user-related data, using loadUserByUsername(), and returns `UserDetails`**. 

   ```java
   @Service
   @RequiredArgsConstructor
   public class UserInfoManagerConfig implements UserDetailsService {
   
        private final UserRepository userRepository;
        @Override
        public UserDetails loadUserByUsername(String emailId) throws UsernameNotFoundException {
            return userRepository
                    .findByEmailId(emailId)
                    .map(UserConfig::new)
                    .orElseThrow(()-> new UsernameNotFoundException("UserEmail: "+emailId+" does not exist"));
        }
   }
   ```
   > Note:it finds user and I will convert it in form of Authentication Object

    - Add the missing method findByEmailId in `UserRepository` which we created in `Step:3 point 2`
        ```java
        @Repository
        public interface UserRepository extends JpaRepository<UserInfoEntity,Long> {
            Optional<UserInfoEntity> findByEmailId(String emailId);
        }
        ```
        <a id="point-5"></a>
5. Let's add  our Security config, to let it access the API using our User. Create a `SecurityConfig` file in config package. 
    ```java
    @Configuration
    @EnableWebSecurity
    @EnableMethodSecurity
    @RequiredArgsConstructor
    public class SecurityConfig  {

        private final UserManagerConfig userManagerConfig;

        @Order(1)
        @Bean
        public SecurityFilterChain apiSecurityFilterChain(HttpSecurity httpSecurity) throws Exception{
            return httpSecurity
                    .securityMatcher(new AntPathRequestMatcher("/api/**"))
                    .csrf(AbstractHttpConfigurer::disable)
                    .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
                    .userDetailsService(userManagerConfig)
                    .httpBasic(withDefaults())
                    .build();
        }

        // Public APIs under /public/** 
        @Order(2)
        @Bean
        public SecurityFilterChain publicSecurityFilterChain(HttpSecurity httpSecurity) throws Exception {
            return httpSecurity
                    .securityMatcher(new AntPathRequestMatcher("/public/**"))
                    .authorizeHttpRequests(auth -> auth
                            .anyRequest().permitAll()  
                    )
                    .csrf(AbstractHttpConfigurer::disable)
                    .build();
        }

    }

    ```
6. Now lets check by adding users in our database.
    >Note we will be using CLI (Command Line Interface) for now..but later will use api to create/save users

    ```java

    @RequiredArgsConstructor
    @Component
    @Slf4j
    public class UserAddUsingCLI implements CommandLineRunner {
        private final UserRepository userRepository;
        private final PasswordEncoder passwordEncoder;

        @Override
        public void run(String... args) throws Exception {
            User manager = new User();
            manager.setUserName("Manager");
            manager.setPassword(passwordEncoder.encode("password"));
            manager.setRoles("ROLE_MANAGER");
            manager.setEmailId("manager@manager.com");

            User admin = new User();
            admin.setUserName("Admin");
            admin.setPassword(passwordEncoder.encode("password"));
            admin.setRoles("ROLE_ADMIN");
            admin.setEmailId("admin@admin.com");

            User user = new User();
            user.setUserName("User");
            user.setPassword(passwordEncoder.encode("password"));
            user.setRoles("ROLE_USER");
            user.setEmailId("user@user.com");

            userRepository.saveAll(List.of(manager, admin, user));
        }
    }

    ```
    - After import all, you might still be seeing error `Could not autowire. No beans of 'PasswordEncoder' type found.` So,lets create bean for `PasswordEncoder` in `SecurityConfig` class which we created at `Step:3 point 5` [Step 3](#point-5)

         ```java
         @Bean
         PasswordEncoder passwordEncoder() {
         return new BCryptPasswordEncoder();
         }
         ```
7. Now lets create a dummy controller to check if we can access api role based or not
    ```java
    @RestController
    @RequestMapping("/api")
    public class TestController {
        @PreAuthorize("hasAnyRole('ROLE_MANAGER','ROLE_ADMIN','ROLE_USER')")
        @GetMapping("/welcome-message")
        public ResponseEntity<String> getFirstWelcomeMessage(Authentication authentication){
            return ResponseEntity.ok("Welcome to the JWT Tutorial:"+authentication.getName()+"with scope:"+authentication.getAuthorities());
        }

        @PreAuthorize("hasRole('ROLE_MANAGER')")
        @GetMapping("/manager-message")
        public ResponseEntity<String> getManagerData(Principal principal){
            return ResponseEntity.ok("Manager::"+principal.getName());

        }

        @PreAuthorize("hasRole('ROLE_ADMIN')")
        @PostMapping("/admin-message")
        public ResponseEntity<String> getAdminData(@RequestParam("message") String message, Principal principal){
            return ResponseEntity.ok("Admin::"+principal.getName()+" has this message:"+message);

        }
    }   
    ```

    - Run the application and also postMan(API testing tool)
    
      - Hit `GET` request in api <http://localhost:8080/api/welcome-message> with Authorization as  `Basic Auth ` with following(choose any one userName and password):
      >
      - UserName for `Manager:` `manager@manager.com`
      - UserName for `Admin:` `admin@admin.com`
      - UserName for `User:` `user@user.com`
      - Password for `all:` `password`
      - Hit `GET` request in api <http://localhost:8080/api/manager-message> with Authorization as  `Basic Auth ` as manager credintials...to get `200` response and hit api again with any other credinitals like (`Admin` or `User`) to get `403 Forbidden` HttpStatus.
      - Similarly hit `POST ` request in api <http://localhost:8080/api/admin-message?message=AdminIsGod> using admin credintials.
      > Note we can hit API on role based...like user and manager cannot hit admin api request (3rd request) and user and admin cannot hit manager api request (2nd request) and we can also specify a api in which all can request (1st request) its done through `@PreAuthorize` annotation.
