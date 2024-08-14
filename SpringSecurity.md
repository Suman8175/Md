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
 Decides whether the authentication is successful or not and what to do with the authentication object based on that.
- <mark> Authentication Provider: </mark> Is someone who validates whether the given credintial are correct or not

- For  Authentication Provider to validate credintials it needs two things : <mark>PasswordEncoder </mark> and <mark>UserDetailsService </mark>

