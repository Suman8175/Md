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
<a id="point-2"></a>
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

    - Add the missing method findByEmailId in `UserRepository` which we created in `Step:3 point 2` [Step:3(2)](#point-2)
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
    - After import all, you might still be seeing error `Could not autowire. No beans of 'PasswordEncoder' type found.` So,lets create bean for `PasswordEncoder` in `SecurityConfig` class which we created at `Step:3 point 5` [Step:3(5)](#point-5)

         ```java
         @Bean
         PasswordEncoder passwordEncoder() {
         return new BCryptPasswordEncoder();
         }
         ```
         <a id="point-3.7"></a>
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

## Step 4: Return _Jwt Access Token_ while authenticating, and add `Roles` and `Permissions` 

1. **Generating Asymmetric Keys with OpenSSL** :
   You have the option to create asymmetric keys (public and private keys) using OpenSSL or utilize the provided files in the repository located at resources/certs.

   Using OpenSSL (Optional)
   If you choose to generate your own keys, follow these steps:

   - Create a `certs` folder in the resources directory and navigate to it:
      ```
      cd src/main/resources/certs
      ```
   
   - Generate a KeyPair :
      This line generates an RSA private key with a length of 2048 bits using OpenSSL (openssl genrsa). 
      It then specifies the output file (-out keypair.pem) where the generated private key will be saved. 
      The significance lies in creating a private key that can be used for encryption, decryption, and digital signatures in asymmetric cryptography.
      ```
      openssl genrsa -out keypair.pem 2048   
      ```
   - Generate a Public Key from the Private Key: 
       This command extracts the public key from the previously generated private key (openssl rsa). 
       It reads the private key from the file specified by -in keypair.pem and outputs the corresponding public key (-pubout) to a file named publicKey.pem. 
       The significance is in obtaining the public key from the private key, which can be shared openly for encryption and verification purposes while keeping 
       the private key secure.
      ```
       openssl rsa -in keypair.pem -pubout -out publicKey.pem 
      
      ```
   - Format the Private Key (keypair.pem) in Supported Format (PKCS8 format):
       This line converts the private key generated in the first step (keypair.pem) into PKCS#8 format, a widely used standard for private key encoding (openssl pkcs8). 
       It specifies that the input key format is PEM (-inform PEM), the output key format is also PEM (-outform PEM), and there is no encryption applied (-nocrypt). 
       The resulting private key is saved in a file named private.pem. 
       The significance is in converting the private key into a standard format that is interoperable across different cryptographic systems and applications.
      ```
      openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in keypair.pem -out privateKey.pem
      ```
     
      **If you want to apply encryption to the private key while exporting it using OpenSSL, you can simply omit the -nocrypt option in the openssl pkcs8 command. By doing so, OpenSSL will prompt you to enter a passphrase that will be used to encrypt the private key**
      **Note:  encrypting the private key adds an extra layer of security, but it also means that you'll need to provide the passphrase whenever you want to use the private key for cryptographic operations.**
 
 2. Add the reference of those keys, from the properties file to be used in RSAKeyRecord. [Externalise the private and public key]
     Inside `RSAKeyRecord.class` which holds, both public and private key that will be used by JWT
      ```java
       @ConfigurationProperties(prefix = "jwt")
       public record RSAKeyRecord (RSAPublicKey rsaPublicKey, RSAPrivateKey rsaPrivateKey){
      
       }
      ```
    > Note:Even after importing all you will see the error in `@ConfigurationProperties(prefix = "jwt")` as
    >
    > `Not registered via @EnableConfigurationProperties, marked as Spring component, or scanned via @ConfigurationPropertiesScan `
    - To fix it do `EnableConfiguraitonProperties` to enable it to be found in properties file.
    ```java
    @EnableConfigurationProperties(RSAKeyRecord.class)
    @SpringBootApplication
    public class OAuthApplication {

        public static void main(String[] args) {
            SpringApplication.run(OAuthApplication.class, args);
        }

    }
    ```
3. Add the path to public and private key in `application.properties`
    ```properties
    #jwt
    jwt.rsa-public-key=classpath:certs/privateKey.pem
    jwt.rsa-private-key=classpath:certs/publicKey.pem
    ```
4. Let's now create a new `filterChain` similar to  `/api`  as  `sign-in` api, which will return **accessToken**  which we created in [Step:3(5)](#point-5)
    - **STATELESS** : A stateless architecture is one in which the server does not store any session data for a client. Instead, each request from the client contains all the information necessary to complete the request
   ```java
   @Bean
    public SecurityFilterChain signInSecurityFilterChain(HttpSecurity httpSecurity) throws Exception{
        return httpSecurity
                .securityMatcher(new AntPathRequestMatcher("/sign-in/**"))
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
                .userDetailsService(userManagerConfig)
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .exceptionHandling(ex -> {
                    ex.authenticationEntryPoint((request, response, authException) ->
                            response.sendError(HttpServletResponse.SC_UNAUTHORIZED, authException.getMessage()));
                })
                .httpBasic(withDefaults())
                .build();
    }
 
    ```
5. Now lets create a `AuthController` , to receive the `sign-in` api and test the filter we created

   ```java
   @RestController
   @RequiredArgsConstructor
   @Slf4j
   public class AuthController {
   
       private final AuthService authService;
       @PostMapping("/sign-in")
       public ResponseEntity<?> authenticateUser(Authentication authentication){
   
           return ResponseEntity.ok(authService.getJwtTokensAfterAuthentication(authentication));
       }
   }
   ```
6. Now add the business logic to return the accessToken

    - What we want to return ? **`AuthResponseDTO`**
   ```java
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class AuthResponseDTO {
        @JsonProperty("access_token")
        private String accessToken;

        @JsonProperty("access_token_expiry")
        private int accessTokenExpiry;

        @JsonProperty("token_type")
        private TokenType tokenType;

        @JsonProperty("user_name")
        private String userName;

    }
   ```
   > Note:Remeber to import TokenType of below created enum not of any other
   - Token Type
   ```java
   public enum TokenType {
       Bearer
   }
   ```

7. Add `AuthService` Class which will contain all business logic for `sign-in` for now
    ```java
    @Service
    @RequiredArgsConstructor
    @Slf4j
    public class AuthService {

        private final UserRepository userRepository;
        private final JwtTokenGenerator jwtTokenGenerator;
        

        public AuthResponseDTO getJwtTokensAfterAuthentication(Authentication authentication) {
            try
            {
                User userInfoEntity = userRepository.findByEmailId(authentication.getName())
                        .orElseThrow(()->{
                            log.error("[AuthService:userSignInAuth] User :{} not found",authentication.getName());
                            return new ResponseStatusException(HttpStatus.NOT_FOUND,"USER NOT FOUND ");});


                String accessToken = jwtTokenGenerator.generateAccessToken(authentication);

                log.info("[AuthService:userSignInAuth] Access token for user:{}, has been generated",userInfoEntity.getUserName());
                return  AuthResponseDTO.builder()
                        .accessToken(accessToken)
                        .accessTokenExpiry(15*60)
                        .userName(userInfoEntity.getUserName())
                        .tokenType(TokenType.Bearer)
                        .build();


            }catch (Exception e){
                log.error("[AuthService:userSignInAuth]Exception while authenticating the user due to :"+e.getMessage());
                throw new ResponseStatusException(HttpStatus.INTERNAL_SERVER_ERROR,"Please Try Again");
            }
        }
    }

    ```
8. Now add `JwtTokenGenerator` in **config/jwtconfig** package
     This builder is used to create a new **JwtClaimsSet object**, which represents the **claims conveyed by a JSON Web Token (JWT)**.
   ```java

    @Service
    @RequiredArgsConstructor
    @Slf4j
    public class JwtTokenGenerator {


        private final JwtEncoder jwtEncoder;

        public String generateAccessToken(Authentication authentication) {

            log.info("[JwtTokenGenerator:generateAccessToken] Token Creation Started for:{}", authentication.getName());

            String roles = getRolesOfUser(authentication);
            List<String> rolesInJWT = getRolesOfUserAsList(authentication);

            String permissions = getPermissionsFromRoles(roles);

            JwtClaimsSet claims = JwtClaimsSet.builder()
                    .issuer("atquil")
                    .issuedAt(Instant.now())
                    .expiresAt(Instant.now().plus(15 , ChronoUnit.MINUTES))
                    .subject(authentication.getName())
                    .claim("scope", permissions)
                    .claim("roles", rolesInJWT)
                    .build();

            return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
        }

        private static String getRolesOfUser(Authentication authentication) {
            return authentication.getAuthorities().stream()
                    .map(GrantedAuthority::getAuthority)
                    .collect(Collectors.joining(" "));
        }

        private static List<String> getRolesOfUserAsList(Authentication authentication) {
            return authentication.getAuthorities().stream()
                  .map(GrantedAuthority::getAuthority)
                  .collect(Collectors.toList());  
        }

        private String getPermissionsFromRoles(String roles) {
            Set<String> permissions = new HashSet<>();

            if (roles.contains("ROLE_ADMIN")) {
            permissions.addAll(List.of("SCOPE_READ", "SCOPE_WRITE","SCOPE_DELETE"));
            }
            if (roles.contains("ROLE_MANAGER")) {
                permissions.add("SCOPE_READ");
            }
            if (roles.contains("ROLE_USER")) {
                permissions.add("SCOPE_READ");
            }

            return String.join(" ", permissions);
        }

    }

   ```
9. Let's add `token encoder` and `decoder`
   **decoder **:  The JwtEncoder takes a **JwtClaimsSet** object as an argument and **returns a JWT as a string**. The JwtEncoder bean is created using the NimbusJwtEncoder class, which is an implementation of the JwtEncoder interface. The NimbusJwtEncoder class uses a **JWKSource object to obtain the key used to sign the JWT**. In this case, the key is obtained from an RSAKey object that is built using the rsaPublicKey() and rsaPrivateKey() methods of a rsaKeyRecord object
   >Note:add following code in securityConfig which we created  at `Step:3 point 5` [Step:3(5)](#point-5)
   
   ```java
    private final RSAKeyRecord rsaKeyRecord;
    @Bean
    JwtDecoder jwtDecoder(){
        return NimbusJwtDecoder.withPublicKey(rsaKeyRecord.rsaPublicKey()).build();
    }

    @Bean
    JwtEncoder jwtEncoder(){
        JWK jwk = new RSAKey.Builder(rsaKeyRecord.rsaPublicKey()).privateKey(rsaKeyRecord.rsaPrivateKey()).build();
        JWKSource<SecurityContext> jwkSource = new ImmutableJWKSet<>(new JWKSet(jwk));
        return new NimbusJwtEncoder(jwkSource);
    }
   ```
10. Lets modify test controller which we created at `Step:3 point 7` [Step:3(7)](#point-3.7)
    ```java
    @RestController
    @RequestMapping("/api")
    public class TestController {
    //    @PreAuthorize("hasAnyRole('ROLE_MANAGER','ROLE_ADMIN','ROLE_USER')")
        @PreAuthorize("hasAuthority('SCOPE_READ')")
        @GetMapping("/welcome-message")
        public ResponseEntity<String> getFirstWelcomeMessage(Authentication authentication){
            return ResponseEntity.ok("Welcome to the JWT Tutorial:"+authentication.getName()+"with scope:"+authentication.getAuthorities());
        }

    //    @PreAuthorize("hasRole('ROLE_MANAGER')")
        @PreAuthorize("hasAuthority('SCOPE_READ')")
        @GetMapping("/manager-message")
        public ResponseEntity<String> getManagerData(Principal principal){
            return ResponseEntity.ok("Manager::"+principal.getName());

        }

    //    @PreAuthorize("hasRole('ROLE_ADMIN')")
        @PreAuthorize("hasAuthority('SCOPE_WRITE')")
        @PostMapping("/admin-message")
        public ResponseEntity<String> getAdminData(@RequestParam("message") String message, Principal principal){
            return ResponseEntity.ok("Admin::"+principal.getName()+" has this message:"+message);

        }
    }
    ```
11. Now modify `/api` config which will use `ouath2` for authentication which we created in `Step:3 point 5` [Step:3(5)](#point-5)
    ```java
        @Bean
        public SecurityFilterChain apiSecurityFilterChain(HttpSecurity httpSecurity) throws Exception{
        return httpSecurity
                .securityMatcher(new AntPathRequestMatcher("/api/**"))
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
                .oauth2ResourceServer(oauth2 -> oauth2
                        .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter()))  // Use the converter
                )
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .exceptionHandling(ex -> {
                    log.error("[SecurityConfig:apiSecurityFilterChain] Exception due to :{}",ex);
                    ex.authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint());
                    ex.accessDeniedHandler(new BearerTokenAccessDeniedHandler());
                })
                .httpBasic(withDefaults())
                .build();
    }
    ```
    also add `jwtAuthenticationConverter()` method in same config file
    ```java
     @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("scope");  // Set claim name for scopes
        grantedAuthoritiesConverter.setAuthorityPrefix("");  // No prefix for scopes

        JwtAuthenticationConverter jwtAuthenticationConverter = new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(jwt -> {
            // Convert scopes
            Collection<GrantedAuthority> authorities = grantedAuthoritiesConverter.convert(jwt);

            // Convert roles
            Collection<GrantedAuthority> roles = jwt.getClaimAsStringList("roles").stream()
                    .map(SimpleGrantedAuthority::new)  // Convert each role to a GrantedAuthority
                    .collect(Collectors.toList());

            // Combine scopes and roles
            Set<GrantedAuthority> combinedAuthorities = new HashSet<>(authorities);
            combinedAuthorities.addAll(roles);

            return combinedAuthorities;
        });

        return jwtAuthenticationConverter;
    }
    ```
    > Now you can also use combination of `hasRole()` and `hasAuthority()` in `@PreAuthorize` annotation 
    >
    >eg: `@PreAuthorize("(hasRole('ROLE_MANAGER') and hasAuthority('SCOPE_READ')) or (hasRole('ROLE_USER'))")`

12. As till now we are doing signup using `basic form` but we may want `userName` and `password` in `requestBody`.
Crate a dto class 
    ```java
    @Data
    public class LoginRequestDTO {
        private String username;
        private String password;
    }
    ```
    and now modify the `sign-in` controller
    ```java
    private final AuthService authService;
    @PostMapping("/sign-in")
    public ResponseEntity<?> authenticateUser(@RequestBody LoginRequestDTO loginRequestDTO){
        Authentication authentication = authService.authenticate(loginRequestDTO.getUserName(), loginRequestDTO.getPassword());

        return ResponseEntity.ok(authService.getJwtTokensAfterAuthentication(authentication));
    }
    ``` 
    also modify the AuthService class by adding following
    ```java
    private final AuthenticationManager authenticationManager;
    public Authentication authenticate(String username, String password) {
        return authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(username, password)
        );
    }

    ```
    now you see a error as `Could not autowire. No beans of 'AuthenticationManager' type found.`Add following in securityConfig file [Step:3(5)](#point-5)
    ```java
    private final AuthenticationConfiguration authConfiguration;
    @Bean
    public AuthenticationManager authenticationManager() throws Exception {
        return authConfiguration.getAuthenticationManager();
    }
    ```
    also be sure to `premitAll()` for `sign-in api` in same class `signInSecurityFilterChain()` method,modify method as
    ```java
     @Bean
    public SecurityFilterChain signInSecurityFilterChain(HttpSecurity httpSecurity) throws Exception{
        return httpSecurity
                .securityMatcher(new AntPathRequestMatcher("/sign-in/**"))
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth ->
                        auth
                                .requestMatchers("/sign-in").permitAll()
                                .anyRequest().authenticated())
                .userDetailsService(userManagerConfig)
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .exceptionHandling(ex -> {
                    ex.authenticationEntryPoint((request, response, authException) ->
                            response.sendError(HttpServletResponse.SC_UNAUTHORIZED, authException.getMessage()));
                })
                .httpBasic(withDefaults())
                .build();
    }
    ```
    Now you should be able to sign up(login) using userName and password in `requestBody`

## Step 5: Adding Custom JwtFilter to `validate` JWTs that are included in the Authorization header of HTTP Request. 
    UseCase: User is removed, then also jwtAccessToken will work, so prevent it. 
1. Let's create the Filter 
    - **OncePerRequestFilter**: The filter is implemented as a subclass of OncePerRequestFilter, which ensures that the filter is only applied once per request.
    - The filter uses the **rsaKeyRecord** object to **obtain the RSA public and private keys used to sign and verify the JWTs**.
    - JWT 
      - Valid: The filter creates an Authentication object and sets it in the SecurityContextHolder. The Authentication object contains the **user details** and **authorities extracted from the JWT**.
      - In-valid: If the JWT is not valid, the filter throws a ResponseStatusException with an HTTP 406 Not Acceptable status code
    
    ```java 
    @RequiredArgsConstructor
    @Slf4j
    public class JwtAccessTokenFilter extends OncePerRequestFilter {

        private final RSAKeyRecord rsaKeyRecord;
        private final JwtTokenUtils jwtTokenUtils;
        @Override
        protected void doFilterInternal(HttpServletRequest request,
                                        HttpServletResponse response,
                                        FilterChain filterChain) throws ServletException, IOException {

            try{
                log.info("[JwtAccessTokenFilter:doFilterInternal] :: Started ");

                log.info("[JwtAccessTokenFilter:doFilterInternal]Filtering the Http Request:{}",request.getRequestURI());

                final String authHeader = request.getHeader(HttpHeaders.AUTHORIZATION);

                JwtDecoder jwtDecoder =  NimbusJwtDecoder.withPublicKey(rsaKeyRecord.rsaPublicKey()).build();

                if(!authHeader.startsWith(TokenType.Bearer.name())){
                    filterChain.doFilter(request,response);
                    return;
                }

                final String token = authHeader.substring(7);
                final Jwt jwtToken = jwtDecoder.decode(token);


                final String userName = jwtTokenUtils.getUserName(jwtToken);

                if(!userName.isEmpty() && SecurityContextHolder.getContext().getAuthentication() == null){

                    UserDetails userDetails = jwtTokenUtils.userDetails(userName);
                    if(jwtTokenUtils.isTokenValid(jwtToken,userDetails)){
                        SecurityContext securityContext = SecurityContextHolder.createEmptyContext();

                        UsernamePasswordAuthenticationToken createdToken = new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities()
                        );
                        createdToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                        securityContext.setAuthentication(createdToken);
                        SecurityContextHolder.setContext(securityContext);
                    }
                }
                log.info("[JwtAccessTokenFilter:doFilterInternal] Completed");

                filterChain.doFilter(request,response);
            }catch (JwtValidationException jwtValidationException){
                log.error("[JwtAccessTokenFilter:doFilterInternal] Exception due to :{}",jwtValidationException.getMessage());
                throw new ResponseStatusException(HttpStatus.NOT_ACCEPTABLE,jwtValidationException.getMessage());
            }
        }
    }

    ```
2. Token Utils
    - Jwt from **Oauth**
    ```java
    @Component
    @RequiredArgsConstructor
    public class JwtTokenUtils {

        public String getUserName(Jwt jwtToken){
            return jwtToken.getSubject();
        }

        public boolean isTokenValid(Jwt jwtToken, UserDetails userDetails){
            final String userName = getUserName(jwtToken);
            boolean isTokenExpired = getIfTokenIsExpired(jwtToken);
            boolean isTokenUserSameAsDatabase = userName.equals(userDetails.getUsername());
            return !isTokenExpired  && isTokenUserSameAsDatabase;

        }

        private boolean getIfTokenIsExpired(Jwt jwtToken) {
            return Objects.requireNonNull(jwtToken.getExpiresAt()).isBefore(Instant.now());
        }

        private final UserRepository userRepository;
        public UserDetails userDetails(String emailId){
            return userRepository
                    .findByEmailId(emailId)
                    .map(UserConfig::new)
                    .orElseThrow(()-> new UsernameNotFoundException("UserEmail: "+emailId+" does not exist"));
        }
    }
    ```
3. Call the tokenFilter from the `SecurityConfig` and modify it as:
    ```java
    private final JwtTokenUtils jwtTokenUtils;
     @Bean
    public SecurityFilterChain apiSecurityFilterChain(HttpSecurity httpSecurity) throws Exception{
        return httpSecurity
                .securityMatcher(new AntPathRequestMatcher("/api/**"))
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
                .addFilterBefore(new JWTAccessTokenFilter(rsaKeyRecord, jwtTokenUtils), UsernamePasswordAuthenticationFilter.class)

                .oauth2ResourceServer(oauth2 -> oauth2
                        .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter()))  // Use the converter
                )
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .exceptionHandling(ex -> {
                    log.error("[SecurityConfig:apiSecurityFilterChain] Exception due to :{}",ex);
                    ex.authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint());
                    ex.accessDeniedHandler(new BearerTokenAccessDeniedHandler());
                })
                .httpBasic(withDefaults())
                .build();
    }
    ```
4. Testing:
   - [Success] Test with the same API  
   - [Failure] After creating the token , **delete the user or Wait for expiry**. 

## Step 6 : `Refresh token ` using `HttpOnly` Cookie and store it in database
1. Let's understand difference between `Access token` and `RefreshToken`

| Topics   | Access Token                                                              | Refresh Token                                                        |
|----------|---------------------------------------------------------------------------|----------------------------------------------------------------------|
| Purpose  | Used to access protected resources on behalf of a user. **Authorization** | Used to obtain a new access token after the previous one has expired |
| Duration | Short-lived (typically minutes to hours).                                 | Long-lived (typically days to weeks)                                 |
| Storage  | Generally returned as Response Object                                     | Must be secured, thus mostly using **HTTPOnly Cookie**               |


2. RefreshToken must be **saved in the database**, to verify and return the access token: ** `RefreshTokenEntity`**
   ```java
    @Entity
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    @Table(name="REFRESH_TOKENS")
    public class RefreshTokenEntity {

        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        private Long refreshId;;
        // Increase the length to a value that can accommodate your actual token lengths
        @Column(name = "REFRESH_TOKEN", nullable = false, length = 10000)
        private String refreshToken;

        @Column(name = "REVOKED")
        private boolean revoked;

        @ManyToOne
        @JoinColumn(name = "user_id",referencedColumnName = "userId")
        private User user;
    }
   ```

   - Let's add the relation `RefreshTokenEntity` to  `User` also
   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @Entity
   @Table(name="users")
   public class User {
      //....
   
       // Many-to-One relationship with RefreshTokenEntity
      @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
      private List<RefreshTokenEntity> refreshTokens;
   }
   
   ```
   - Now map Entity to `RefreshTokenRepository`
   ```java
   @Repository
   public interface RefreshTokenRepository extends JpaRepository<RefreshTokenEntity, Long> {
   }

   ```
3. Create a `refreshTokenGenerator` method in `JwtTokenGenerator`. Remember, no **API access scope should be added**

   ```java
   @Service
   @RequiredArgsConstructor
   @Slf4j
   public class JwtTokenGenerator {
   
   
      //....
       public String generateRefreshToken(Authentication authentication) {

        log.info("[JwtTokenGenerator:generateRefreshToken] Token Creation Started for:{}", authentication.getName());
        
        JwtClaimsSet claims = JwtClaimsSet.builder()
                .issuer("my-app")
                .issuedAt(Instant.now())
                .expiresAt(Instant.now().plus(15 , ChronoUnit.DAYS))
                .subject(authentication.getName())
                .claim("scope", "SCOPE_REFRESH_TOKEN")
                .claim("roles","SCOPE_REFRESH_TOKEN")
                .build();

        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }
       
       // ....
   }
   ```

4. **Modify** `getJwtTokensAfterAuthentication` so that when user `sign-in`, he will receive the **refresh-token** as well, so that when **access token** expires , it get's a new access token using refresh token. 

   ```java
   @Service
   @RequiredArgsConstructor
   @Slf4j
   public class AuthService {
   
       //...
       private final RefreshTokenRepository refreshTokenRepository;
       public AuthResponseDto getJwtTokensAfterAuthentication(Authentication authentication) {
           try
           {
              //....
               String refreshToken = jwtTokenGenerator.generateRefreshToken(authentication);
   
               //Let's save the refreshToken as well
               saveUserRefreshToken(userInfoEntity,refreshToken);
               log.info("[AuthService:userSignInAuth] Access token for user:{}, has been generated",userInfoEntity.getUserName());
                return  AuthResponseDTO.builder()
                    .accessToken(accessToken)
                    .accessTokenExpiry(jwtExpirationMs)
                    .userName(userInfoEntity.getUserName())
                    .tokenType(TokenType.Bearer)
                    .build();
   
   
           }catch (Exception e){
               log.error("[AuthService:userSignInAuth]Exception while authenticating the user due to :"+e.getMessage());
               throw new ResponseStatusException(HttpStatus.INTERNAL_SERVER_ERROR,"Please Try Again");
           }
       }
   
      private void saveUserRefreshToken(User user, String refreshToken) {
        var refreshTokenEntity = RefreshTokenEntity.builder()
                .user(user)
                .refreshToken(refreshToken)
                .revoked(false)
                .build();
        refreshTokenRepository.save(refreshTokenEntity);

      }
   }
   
   ```
5. We will be returning the `refresh-token` using **`HttpOnlyCookie`** so will need **HttpServletResponseObject**

   - Add `HttpServletReponse` in the `/sign-in` api. 
   ```java
   @RestController
   @RequiredArgsConstructor
   @Slf4j
   public class AuthController {
   
      private final AuthService authService;
      @PostMapping("/sign-in")
      public ResponseEntity<?> authenticateUser(@RequestBody LoginRequestDTO loginRequestDTO, HttpServletResponse response){
        Authentication authentication = authService.authenticate(loginRequestDTO.getUserName(), loginRequestDTO.getPassword());

        return ResponseEntity.ok(authService.getJwtTokensAfterAuthentication(authentication,response));
    }
   }
   ```
   - Modify the method to `create a cookie`. 
   ```java
   @Service
   @RequiredArgsConstructor
   @Slf4j
   public class AuthService {
   
    
       public AuthResponseDto getJwtTokensAfterAuthentication(Authentication authentication, HttpServletResponse response) {
               //..
               String accessToken = jwtTokenGenerator.generateAccessToken(authentication);
               String refreshToken = jwtTokenGenerator.generateRefreshToken(authentication);
               //..
               createRefreshTokenCookie(response,refreshToken);
   
               //....
       }
   
       private Cookie createRefreshTokenCookie(HttpServletResponse response, String refreshToken) {
           Cookie refreshTokenCookie = new Cookie("refresh_token",refreshToken);
           refreshTokenCookie.setHttpOnly(true);
           refreshTokenCookie.setSecure(true);
           refreshTokenCookie.setMaxAge(15 * 24 * 60 * 60 ); // in seconds
           response.addCookie(refreshTokenCookie);
           return refreshTokenCookie;
       }
   
   }
   
   ```

   > Note: This will add refreshToken in HttpOnlyCookie...HttpOnlyCookie is safe in web but it is not supported in web but not in mobile...For httpOnly in web and as responseBody in web when hitting request in signup api ask device as another another parameter like:userName,password,device...and do if ..else `if device=web {HttpCookie} else {responseBody}`


6. Add the `/refresh-token` api , so that we can get **new accessToken**

   - Create the Api
   ```java
   @RestController
    @RequiredArgsConstructor
    @Slf4j
    public class AuthController {
    
        //...
        @PreAuthorize("hasAuthority('SCOPE_REFRESH_TOKEN')")
        @PostMapping ("/refresh-token")
        public ResponseEntity<?> getAccessToken(@RequestHeader(HttpHeaders.AUTHORIZATION) String authorizationHeader){
            return ResponseEntity.ok(authService.getAccessTokenUsingRefreshToken(authorizationHeader));
        }
    }

   ```
7. Now write the businessLogic 
   ```java
   @Service
   @RequiredArgsConstructor
   @Slf4j
    public class AuthService {
    
        //...
    
         public Object getAccessTokenUsingRefreshToken(String authorizationHeader) {
            if(!validateRefreshToken(authorizationHeader)){
            return new ResponseStatusException(HttpStatus.INTERNAL_SERVER_ERROR,"Please verify your token type");
            }
            final String refreshToken = authorizationHeader.substring(7);
            //Find refreshToken from database and should not be revoked : Same thing can be done through filter.
            RefreshTokenEntity refreshTokenEntity = refreshTokenRepository.findByRefreshToken(refreshToken)
                .filter(tokens-> !tokens.isRevoked())
                .orElseThrow(()-> new ResponseStatusException(HttpStatus.INTERNAL_SERVER_ERROR,"Refresh token revoked"));
            User userInfoEntity = refreshTokenEntity.getUser();
            //Now create the Authentication object
            Authentication authentication =  createAuthenticationObject(userInfoEntity);
            //Use the authentication object to generate new accessToken as the Authentication object that we will have may not contain correct role.
            String accessToken = jwtTokenGenerator.generateAccessToken(authentication);

         return  AuthResponseDTO.builder()
                .accessToken(accessToken)
                .accessTokenExpiry(jwtExpirationMs)
                .userName(userInfoEntity.getUserName())
                .tokenType(TokenType.Bearer)
                .build();
         }
                private static Authentication createAuthenticationObject(User userInfoEntity) {
                        // Extract user details from UserDetailsEntity
                        String username = userInfoEntity.getEmailId();
                        String password = userInfoEntity.getPassword();
                        String roles = userInfoEntity.getRoles();

                        // Extract authorities from roles (comma-separated)
                        String[] roleArray = roles.split(",");
                        GrantedAuthority[] authorities = Arrays.stream(roleArray)
                                .map(role -> (GrantedAuthority) role::trim)
                                .toArray(GrantedAuthority[]::new);

                        return new UsernamePasswordAuthenticationToken(username, password, Arrays.asList(authorities));
                    }

                private boolean validateRefreshToken(String authorizationHeader){
                    return authorizationHeader.startsWith(TokenType.Bearer.name());
                }


    }
   ```
   
    - Also add the helper method in the repo
    ```java
    @Repository
    public interface RefreshTokenRepository extends JpaRepository<RefreshTokenEntity, Long> {
    
        Optional<RefreshTokenEntity> findByRefreshToken(String refreshToken);
        
    }
    
    ```
8. Now, add the `/refresh-token` in **securityConfig**. 

    ```java
    @Configuration
    @EnableWebSecurity
    @EnableMethodSecurity
    @RequiredArgsConstructor
    @Slf4j
    public class SecurityConfig {
        
        //..
        
         @Order(3)
         @Bean
            public SecurityFilterChain refreshTokenSecurityFilterChain(HttpSecurity httpSecurity) throws Exception{
                return httpSecurity
                        .securityMatcher(new AntPathRequestMatcher("/refresh-token/**"))
                        .csrf(AbstractHttpConfigurer::disable)
                        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
                        .oauth2ResourceServer(oauth2 -> oauth2.jwt(withDefaults()))
                        .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                        .addFilterBefore(new JWTRefreshTokenFilter(rsaKeyRecord,jwtTokenUtils,refreshTokenRepository), UsernamePasswordAuthenticationFilter.class)
                        .exceptionHandling(ex -> {
                            log.error("[SecurityConfig:refreshTokenSecurityFilterChain] Exception due to :{}",ex);
                            ex.authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint());
                            ex.accessDeniedHandler(new BearerTokenAccessDeniedHandler());
                        })
                        .httpBasic(withDefaults())
                        .build();
            }

    }
    
    ```
9. We can also create a **custom filter** to check the validity for `refreshToken`

   - Create a **JwtRefreshTokenFilter**: 
   ```java
    @RequiredArgsConstructor
    @Slf4j
    public class JWTRefreshTokenFilter extends OncePerRequestFilter {
        private  final RSAKeyRecord rsaKeyRecord;
        private final JwtTokenUtils jwtTokenUtils;
        private final RefreshTokenRepository refreshTokenRepo;

        @Override
        protected void doFilterInternal(HttpServletRequest request,
                                        HttpServletResponse response,
                                        FilterChain filterChain) throws ServletException, IOException {

            try {
                log.info("[JwtRefreshTokenFilter:doFilterInternal] :: Started ");

                log.info("[JwtRefreshTokenFilter:doFilterInternal]Filtering the Http Request:{}", request.getRequestURI());


                final String authHeader = request.getHeader(HttpHeaders.AUTHORIZATION);

                JwtDecoder jwtDecoder = NimbusJwtDecoder.withPublicKey(rsaKeyRecord.rsaPublicKey()).build();

                if (!authHeader.startsWith("Bearer ")) {
                    filterChain.doFilter(request, response);
                    return;
                }

                final String token = authHeader.substring(7);
                final Jwt jwtRefreshToken = jwtDecoder.decode(token);


                final String userName = jwtTokenUtils.getUserName(jwtRefreshToken);


                if (!userName.isEmpty() && SecurityContextHolder.getContext().getAuthentication() == null) {
                    //Check if refreshToken isPresent in database and is valid
                    var isRefreshTokenValidInDatabase = refreshTokenRepo.findByRefreshToken(jwtRefreshToken.getTokenValue())
                            .map(refreshTokenEntity -> !refreshTokenEntity.isRevoked())
                            .orElse(false);

                    UserDetails userDetails = jwtTokenUtils.userDetails(userName);
                    if (jwtTokenUtils.isTokenValid(jwtRefreshToken, userDetails) && isRefreshTokenValidInDatabase) {
                        SecurityContext securityContext = SecurityContextHolder.createEmptyContext();

                        UsernamePasswordAuthenticationToken createdToken = new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities()
                        );

                        createdToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                        securityContext.setAuthentication(createdToken);
                        SecurityContextHolder.setContext(securityContext);
                    }
                }
                log.info("[JwtRefreshTokenFilter:doFilterInternal] Completed");
                filterChain.doFilter(request, response);
            }catch (JwtValidationException jwtValidationException){
                log.error("[JwtRefreshTokenFilter:doFilterInternal] Exception due to :{}",jwtValidationException.getMessage());
                throw new ResponseStatusException(HttpStatus.NOT_ACCEPTABLE,jwtValidationException.getMessage());
            }
        }
    }
    ```
10. Test the api : 
   - Sign-in using admin : http://localhost:8080/sign-in
   - Copy the `refresh-token` from `cookie`
   - Use the `refresh-token` to get new `access-token`: http://localhost:8080/refresh-token
   - Access any of the `admin-api` using it : http://localhost:8080/api/admin-message

## Step 7: `Sign-out` and `Revoke` the token

1. Spring Security provide inbuilt api `/logout` to manage **revoking**. 

   ```java
   @Configuration
   @EnableWebSecurity
   @EnableMethodSecurity
   @RequiredArgsConstructor
   @Slf4j
   public class SecurityConfig extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {
   
       //..
       private final LogoutHandlerService logoutHandlerService;
        //...
       @Order(4)
       @Bean
       public SecurityFilterChain logoutSecurityFilterChain(HttpSecurity httpSecurity) throws Exception {
           return httpSecurity
                   .securityMatcher(new AntPathRequestMatcher("/logout/**"))
                   .csrf(AbstractHttpConfigurer::disable)
                   .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
                   .oauth2ResourceServer(oauth2 -> oauth2.jwt(withDefaults()))
                   .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                   .addFilterBefore(new JwtAccessTokenFilter(rsaKeyRecord,jwtTokenUtils), UsernamePasswordAuthenticationFilter.class)
                   .logout(logout -> logout
                           .logoutUrl("/logout")
                           .addLogoutHandler(logoutHandlerService)
                           .logoutSuccessHandler(((request, response, authentication) -> SecurityContextHolder.clearContext()))
                   )
                   .exceptionHandling(ex -> {
                       log.error("[SecurityConfig:logoutSecurityFilterChain] Exception due to :{}",ex);
                       ex.authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint());
                       ex.accessDeniedHandler(new BearerTokenAccessDeniedHandler());
                   })
                   .build();
            }
    //..
    }
   ```
2. Add Logic for revoking access
   ```java

    @Service
    @Slf4j
    @RequiredArgsConstructor
    public class LogoutHandlerService implements LogoutHandler {

        private final RefreshTokenRepository refreshTokenRepo;

        @Override
        public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {

            final String authHeader = request.getHeader(HttpHeaders.AUTHORIZATION);

            if(!authHeader.startsWith(TokenType.Bearer.name())){
                return;
            }

            final String refreshToken = authHeader.substring(7);

            var storedRefreshToken = refreshTokenRepo.findByRefreshToken(refreshToken)
                    .map(token->{
                        token.setRevoked(true);
                        refreshTokenRepo.save(token);
                        return token;
                    })
                    .orElse(null);
        }
    }

   ```

3. Now test the api using `refreshToken` : http://localhost:8080/logout

4. **Note: If you want to revoke/logout from all the places/devices, you can get the userName**

    ```java
    @Repository
    public interface RefreshTokenRepository  extends JpaRepository<RefreshTokenEntity,Long> {
        Optional<RefreshTokenEntity> findByRefreshToken(String refreshToken);
        @Query(value = "SELECT rt.* FROM REFRESH_TOKENS rt " +
                "INNER JOIN USER_DETAILS ud ON rt.user_id = ud.id " +
                "WHERE ud.EMAIL = :userEmail and rt.revoked = false ", nativeQuery = true)
        List<RefreshTokenEntity> findAllRefreshTokenByUserEmailId(String userEmail);
    }

    ```
