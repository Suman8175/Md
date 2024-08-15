# Spring Security


<img title="a title" alt="Spring Security Structure" src="https://res.cloudinary.com/dhmdgbhby/image/upload/v1723628537/remknrxzdmtwacuzkhor.png">

> Fig: Spring-Security

## Overview 

- <mark> Your Application Controllers: </mark> Controllers like /create-users,/products

- <mark> Request/ Response </mark>
- <mark>Filter Chain: </mark> Series of filter upon which we configured
- <mark> Authentication Filter: </mark> Is available only when we use spring security. 
     Intercepting HTTP requests to extract authentication details (such as a username and password)  and create authentication object from it
- <mark> AuthenticationManager: </mark> 
 Decides whether the authentication is successful or not and what to do with the authentication object based on that.s
- <mark> Authentication Provider: </mark> Is someone who validates whether the given credintial are correct or not..like it validates google sign in,credintials for db,etc

- For  Authentication Provider to validate credintials it needs two things : <mark>PasswordEncoder </mark> and <mark>UserDetailsService </mark>


## To permit certain request
>Like signup doesnot need user to have authentication...like in ecommerce /products for get request doesn't need user to be authenticated

```java
 @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/v1/app/signup").permitAll()
                .anyRequest().authenticated())
                .sessionManagement(session -> session.sessionCreationPolicy(STATELESS))
                .addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

```
The line **`.requestMatchers("/v1/app/signup").permitAll()`** allows unauthenticated access to the signup endpoint.

## To enable logging level to deubg of spring security
- Below unlocks all logging level for spring security which is very handy in case for debugging spring security problems/errors.

```properties
logging.level.org.springframework.security=DEBUG
```

## Basic authentication

```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests(request -> request
                        .anyRequest().authenticated());
                http.sessionManagement(session -> session.sessionCreationPolicy(STATELESS));
                http.httpBasic(withDefaults());
        http.csrf(AbstractHttpConfigurer::disable);
        return http.build();
    }
```
> ```http.httpBasic(withDefaults());``` is defines that seccurity is basic_auth like: bearer,basic_auth,etc


