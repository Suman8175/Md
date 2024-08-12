# Steps to use redux-toolkit in react application

 ## 1. ***Run commands to install react-redux and redux-toolkit***:
```sh
   npm install @reduxjs/toolkit
```
```sh
    npm install react-redux
```
##
## 2. ***Create Store***

- Create a seperate folder inside src/app/store.js.

    (Store.js is naming convention for store)
    #

- Import configStore

    ```js
    import { configureStore } from '@reduxjs/toolkit'
    ```
    #

- Export the store
    ```js
    export const store=configureStore({})
    ```
##

## 3. ***Create Slice***

- Create a folder src/feature/todo/todoSlice.js

    (Todo is any name you want)
    #
- Import Slice
    ```js
    import { createSlice } from '@reduxjs/toolkit'
    ```
    #
- Create inital state which will further be used in slice

    ```js
    const initialState={
        todos:[
            {
                id:1,
                title:'Title1'
            }
            ]
    }
    ```
    or something like userAuthenticate part
    ```js
    const initialState2={
    user: null,
    isAuthenticated: false,
  }

    ```
- Create a slice
    ```js
    export const todoSlice=createSlice({
        name:'todo',
        initialState,
        reducers:{
            addToDo:(state,action)=>{
                const todo={
                    id:Date.now(),
                    title:action.payload.text
                }
                state.todos.push(todo);
            },
            removeToDo:(state,action)=>{
                state.todos=state.todos.filter((todo)=>todo.id!==action.payload)
                //simple remove logic where state.todos is override by taking all values except the one in which id matches
            },
            updateToDo:(state,action)=>{
                //perform logic here match id and if match change its title to action.payload
            }
        }
    })
    ```
     or something like userAuthenticate part
     ```js
     export const authenticateSlice=createSlice({
        name:'auth',
        initialState2,
        reducers:{
            setUser:(state,action)=>{
                state.user = action.payload;
                state.state.isAuthenticated = true;
            },
            logout: (state) => {
            state.user = null;
            state.isAuthenticated = false;
            }
        }
    })
     ```
  > <span style="color: #f54263; font-weight: bold;"><mark>[NOTE]<mark></span>

    > <span style="background-color: #f54263; color: #000000"><mark>name</mark></span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- reserved keyword for createSlice.What you give name in value that will be the sliceName.

    ><span style="background-color: #f54263; color: #000000"><mark>reducers</mark></span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-takes all function which can change /modify the value in a slice because redux ensures there is only single source of truth.

    >Each function inside reducers can have access to  two value:<span style="background-color: #a1d6a1; color: #000000"> <mark>state</mark> </span> and <span style="background-color: #a1d6a1; color: #000000"><mark>action</mark> </span>.
    >
    >   <span style="background-color: #a1d6a1; color: #000000"><mark> State contains all the current value access inside initialState...meaning suppose we added 10 more items in todoSlice (todos...inititalState key) we get access of it </mark></span>
    >
    ><span style="background-color: #a1d6a1; color: #000000"><mark> Action contains value which is needed for function to run...like we need id inside function to remove anything...we need data to add..so action holds the data coming from whoever uses it..like they pass the data and action catches it</mark></span>



- Now export the slice functions like:
    ```js
    export const {addToDo,removeToDo,updateToDo}=todoSlice.actions
    
    export default todoSlice.reducer 
    ```
    or for authenticate slice
    ```js
    export const {setUser,logout}=authenticateSlice.actions

    export default authenticateSlice.reducer
    ```

##

## 4. ***Update store we made earlier***
 Now remember we had store with ``` export const store=configureStore({}) ``` where no slice were registered.Now lets register.Now update it with below:

```js
    import todoReducer from './todoSlice';//Remember to import


    export const store=configureStore({
        reducer:todoReducer
    })
```

or if you have multiple reducer 
```js
    import authReducer from './authSlice'; //Remember to import 
    import todoReducer from './todoSlice';//Remember to import


    export const store = configureStore({
        reducer: {
            auth: authReducer,   // The 'authenticateSlice' slice will be managed by authReducer
            todos: todoReducer,  // The 'todoSlice' slice will be managed by todoReducer
        },
    });
```
##

## 5. ***Use useDispatch()***

- > <span style="color: #f54263; font-weight: bold;"><mark>[NOTE]<mark></span>

    > useDispatch() is used whenever we want to save value in store...Dispatch uses reducer to save/update/remove value in store...

- Lets assume you want to save when save() function is called 
    ```js
    import {useDispatch} from 'react-redux'
    import {addTodo} from '../todoSlice' 


    const dispatch=useDispatch();
    const [value,setValue]=useState({});
    //Assuming you did setValue() when form is submitted so that value have now data
    const save = (e) => {
        e.preventDefault() //Preventing form for submitting to url
        dispatch(addTodo(value))  //addToDo is a function which we made in toDoreducers
    }
    ```
##

## 6. ***Use useSelector()***
- > <span style="color: #f54263; font-weight: bold;"><mark>[NOTE]<mark></span>

    > useSelector() is used whenever we want to fetch/extract value from store...useSelector is just like DQL or select query in database...
    >
    ><mark>When you use custom keys, you’ll access those parts of the state with the corresponding key names. </mark>

- Now let's fetch data using useSelector()

    ```js
    import { useSelector} from 'react-redux'


     const user = useSelector((state) => state.auth.user);  // Accessing user via 'auth'
     const todos = useSelector((state) => state.todos);  //todos because we defined key as todos in store

    ```
- Now user and todos contains all data store in <mark> todoSlice</mark> and <mark>authSlice.</mark>





