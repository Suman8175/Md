# React Query

### Fetching Data Using `fetch` with `async/await`
  ```js
    const Products = () => {
        const [products, setProducts] = useState([]);
        useEffect(() => {
            const fetchData = async () => {
                const response = await fetch('http://localhost:8081/products');
                const data = await response.json();
                setProducts(data);
            }; fetchData();

        }, [])
        console.log(products);
        return (
            <div>Products</div>
        )
    }

    export default Products
  ```
### Fetching Data Using `fetch` with Promises (`.then()`)
  ```js
    import React, { useEffect, useState } from 'react'
    const Products = () => {
        const [products, setProducts] = useState([]);
        useEffect(() => {
            fetch('http://localhost:8081/products')
                .then(res => res.json())
                .then(json => setProducts(json))
                
        }, [])
        console.log(products);
    return (
        <div>Products</div>
    )
    }

    export default Products
  ```

### Problem
1 ***Too much complexity***
  - Now suppose to add loading when the data is beign fetched
  ```js
    const [products, setProducts] = useState([]);
    const [loading, setLoading] = useState(false);//declare a new state for loading
    useEffect(() => {
          const datafetch=async()=>{
            setLoading(true);
            const response = await fetch('http://localhost:8081/products')
            const data = await response.json()
            setProducts(data)
            setLoading(false);
          };datafetch(); 
                
        }, [])

        if (loading) {
            return <p className='flex justify-center items-center'>Loading...</p>;
          }
  ```
  - Also same for error handling
    ```js
    const [products, setProducts] = useState([]);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);   
    useEffect(() => {
        const datafetch = async () => {
          try {
            setError(null);
            setLoading(true);
            const response = await fetch('http://localhost:8081/productss')
            const data = await response.json()
            //Fetch doesnot throw error directly for 404 and others so checking if not 200
            if (!response.ok) {
              throw new Error(`HTTP error! status: ${response.status}`);
            }
            setProducts(data)
            console.log(data)
            setLoading(false);
          } catch (error) {
            setError(error.message);
            setLoading(false);
          }
        }; datafetch();

      }, [])

      if (loading) {
        return <p className='flex justify-center items-center'>Loading...</p>;
      }

      if (error) {
        return <p className='flex justify-center items-center'>{error}</p>;
      } 
     ```
 - Also another important thing is missing...which is `Cache`.Suppose `user clicks on one product suppose Shirt to view shirt details and goes back to all product listing the request is sent again to server why not cache the data`
  ```js
  <Link to={`/products/${product.productId}`}>
                          <span aria-hidden="true" className="absolute inset-0" />
                          {product.productName}
                        </Link>
  ```
  >Note : this will navigate to certain product and if we go back to all product listing the request is sent again to server

 ***
 ***To overcome all these and many more we use react query***
 ***

 ### Installation

- To install react query use `npm i @tanstack/react-query`
- Next wrap main.jsx with `QueryClientProvider`
    ```js
    import {QueryClient,} from '@tanstack/react-query'

    const queryClient = new QueryClient()
    
    ReactDOM.createRoot(document.getElementById('root')).render(
      <StrictMode>
        <QueryClientProvider client={queryClient}>
        <RouterProvider router={router} />
        </QueryClientProvider>
      </StrictMode>,
    )

    ```

- Now simply use reactQuery as
  ```js
  import { useQuery } from '@tanstack/react-query';
  const datafetch = async () => {
      const response = await fetch('http://localhost:8081/products')
      const data = await response.json()
      return data;
  }
  export default Products = () => { //rafce
    const {isLoading,error,data:products} = useQuery({ queryKey: ['products'], queryFn: datafetch })

    if (isLoading) {
      return <p className='flex justify-center items-center'>Loading...</p>;
    }

    if (error) {
      return <p className='flex justify-center items-center'>{error}</p>;
    }
  }

  ```
  > `const {isLoading,error,data:products} = useQuery({ queryKey: ['products'], queryFn: datafetch })`
  >
  >`isLoading` , `error` , `data` should have same name no other name.also if we want other name for fetched data we can do `data:<otherName>` 
  >
  >`queryKey: ['<keyName']` is used for caching by react query
  >
  > `queryFn: datafetch` contains logic for fetching data

  or we can  `separate fetching logic` in separate file and `import it`
  ```js
  export const datafetch = async () => {
        const response = await fetch('http://localhost:8081/products')
        const data = await response.json()
        return data;
    }
  ```
  and simply import it

- For caching add `staleTime:<numberinMilli>` in useQueryHook
  ```js
  const { isLoading, error, data: products } = useQuery({ queryKey: ['products'], 
         queryFn: datafetch,
           staleTime:10000 })
  ```

### For creating/post products use `useMutation()` Hook

- First create hook
  ```js
   const mutation= useMutation({mutationFn: (productData)=>{
    return axios.post('http://localhost:8081/products',productData)
  }})
  //productData is the requestBody for post
  ```
- Implement `.mutate()` method wherever to call post api like:
  ```js
  formData(){
    const productData= //contains all requestBody value
  mutation.mutate(productData);
  }

  ```
- For showing loading,error or success message use
    ```js
    if (isLoading || mutation. isPending) {
    return <p className='flex items-center justify-center'>Loading </p>;
  }
    //inside body
    {mutation.isPending && <p className='mt-10 text-black bg-blue-400'>Submitting product data...</p>}
    {mutation.isError && <p>Error submitting product: {mutation.error.message}</p>}
    {mutation.isSuccess && <p>Product submitted successfully!</p>}
    ```

> # Note: If you want like UI update when data is posting in server before result comes and if response is fail show look for `Optimistic Updates Ui`
>
- Eg: nothing added in anywhere just checking
  ```js
  <div className={`mb-5 ${mutation.isPending ? "opacity-50" : "opacity-100"}`}>
  {mutation.isPending ? mutation.variables.productName: mutation.isSuccess ? mutation.variables.productName : data.productName }
  </div>
  ```
> Note: if our form UI is in other component see `optimistics updates cache` 


## Pagination
- Define `skip/offset` and `limit` in useState hook
  ```js
  const [limit] = useState(4) //no of items displayed in screen 4 items in each page
  const [skip, setSkip] = useState(0) //page 2 will have 4 value of skip,page 3 will have 8 and so on
  ```

- Define button for `next` and `prev`
  > Note:you can have two separate function in each button or same function with - value for prev and + value for next
  ```js
  <div className="flex justify-center gap-5 mt-5">
              <button className='px-3 py-1 text-white bg-blue-500' onClick={() => { handlePagination(-limit) }}>Previous</button>
              <button className='px-3 py-1 text-white bg-blue-500' onClick={() => { handlePagination(limit) }}>Next</button>
            </div>
  ```
- Now define the function which will either increase or decrease skip value for next page 
   ```js
  const handlePagination = (moveCount) => {
      setSkip((prev) => {
        return Math.max(prev + moveCount, 0)
      })
    }
    //seting Skip value to +4 for next page and -4 for prev page
   ```

- Now add the `limit` and `offset/skip` in `queryKey`
  ```js
  useQuery({ queryKey: ['products',limit,skip],
             queryFn: () => datafetch(limit, skip), 
             staleTime: 5000 ,
             placeholderData: keepPreviousData})
  ```

  >Notice that the pagination is done but it is not shown in url...like if we share url it will not show the paginated to do it use useSearchParams hook

- Remove useState hook and use useSearchParams hook
  ```js
   const [searchParam,setSearchParam] =useSearchParams({limit:4,skip:0});
   const skip=parseInt(searchParam.get('skip')) || 0;
   const limit=parseInt(searchParam.get('limit')) || 4;
  ```
- Modify `handlePagination` function
  ```js
   const handlePagination = (moveCount) => {
    setSearchParam((prev) => {
      prev.set('skip', Math.max(skip + moveCount, 0));
      return prev;
  });
  }
  ```
- Add `skip` and `limit` in `queryKey` if already not added:
  ```js
  useQuery({ queryKey: ['products',limit,skip],
             queryFn: () => datafetch(limit, skip), 
             staleTime: 5000 ,
             placeholderData: keepPreviousData})
  ```

## **Use debounce `new dependency` for searching data.**
- Use `lodash.debounce` for searching data.Like we want to fetch data after 1 seconds not like on every search..meaning
  - User wants to search Phone
  - Normally we do `GET` req on `P`
  - `GET` req on `Ph`
  - `GET` req on `Pho`
  - `GET` req on `Phon`
  - `GET` req on `Phone`
- This is not good as it loads up the server we want to debounce on `1 second` or `n second` the keyword change


