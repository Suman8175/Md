# Single Responsibility

- It states that one class should have only one job.A class cannot have more than one job as it will voilate SRP.

## Example
What happen if we dont use SRP

### Case 1:
Suppose our UserServiceImpls class handles
 - User CRUD
 - Auth
 - Notification

Now problem lies if we want to change logic to auth then we have to do regression testing and all other testing to whole class.But auth is not related to user CRUD. So,changing in auth might even break other function in same class.


What if we want to change logic in Notification ? As notification is enforced into same UserServiceImpls class...We need to be cautious when changing logic of notification even though It might not even be realted to any way for userCRUD.


### Solution

- We create seperate class for UserCrud,Notification and Auth.
  - UserServiceImpls -> UserCrud by talking to repo.
  - Notification Class -> Manages notification task
  - Auth Class -> Manages all auth related task

So now even if we need to change notification logic we dont even need to worry about UserServiceImpls or Auth Class.
