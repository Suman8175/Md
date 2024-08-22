# Shadcn

# Step 1: Configure shadcn in the react application
- Create `react app` with `vite`.
- Configure `tailwind css`
- Create `jsconfig.json` file in app
    >Note if you are using typescript create `tsconfig.json`

    ```json
    {
        "compilerOptions": {
            "baseUrl": ".",
            "paths": {
                "@/*": [
                    "./src/*"
                ]
            }
        }
    }
    ```
- Navigate to `vite.config.js` file and add

    ```js
    import path from "path"
    resolve: {
        alias: {
        "@": path.resolve(__dirname, "./src"),
        },
    },
    ```
   So it looks like :

    ```js
    import { defineConfig } from 'vite'
    import path from "path"
    import react from '@vitejs/plugin-react'

    // https://vitejs.dev/config/
    export default defineConfig({
    plugins: [react()],
    resolve: {
        alias: {
        "@": path.resolve(__dirname, "./src"),
        },
    },
    })

    ```

- Run command `npx shadcn-ui@latest init`
- Answer following questions
```sh
Would you like to use TypeScript (recommended)? no 
Which style would you like to use? › Default
Which color would you like to use as base color? › Slate
Where is your global CSS file? › › src/index.css
Are you using a custom tailwind prefix eg. tw-? (Leave blank if not) ...
√ Where is your tailwind.config.js located? ... tailwind.config.js
Do you want to use CSS variables for colors? ›  yes
Where is your tailwind.config.js located? › //BLANK
Configure the import alias for components: › //BLANK
Configure the import alias for utils: › //BLANK
Are you using React Server Components? ›  yes 

```

## Step 2: Run commands to add components
- For button
    ```sh
    npx shadcn-ui@latest add button
    ```

## Example for form
- Add form dependency
    ```sh
    npx shadcn-ui@latest add form
    ```

- Add input dependency as input is used in form
    ```sh
    npx shadcn-ui@latest add input
    ```
- Create a `zod validation schema`
    ```js
    import { z } from "zod"

    export const SignupFormValidation = z.object({
        username: z.string().min(2, { message: "Username must be at least 2 characters." }),
        name: z.string().min(2,{message:"Name is required"}),
        email: z.string().email("Invalid email address"),
        password: z.string()
            .min(6, "Password must be at least 6 characters long")
            .regex(/[0-9]/, "Password must contain at least one number"),
        confirmPassword: z.string(),
        gender: z.enum(["male", "female", "others"], {
            errorMap: () => ({ message: "Please select a gender" }),
        }),
    }).refine((data) => data.password === data.confirmPassword, {
        path: ["confirmPassword"],
        message: "Passwords do not match",
    });
    ```
>Note: You can have multiple schema like:For signup,login,etc

- Create jsx maybe `SigninForm.jsx` using above zod schema
    ```js
    import { zodResolver } from "@hookform/resolvers/zod"
    import { useForm } from "react-hook-form"
    import { Button } from "@/components/ui/button"
    import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage, } from "@/components/ui/form"
    import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue, } from "@/components/ui/select"
    import { Input } from "@/components/ui/input"
    import { SignupFormValidation } from "@/lib/validation"

    

    const SigninForm = () => {

    // 1. Define your form.
    const form = useForm({
        resolver: zodResolver(SignupFormValidation),
        defaultValues: {
        username: '',
        name: '',
        email: '',
        password: '',
        confirmPassword: '',
        gender: "", 
        },
    })
    // 2. Define a submit handler.
    function onSubmit(values) {
        // Get all the form values
    const allValues = form.getValues();
    console.log(allValues);
    }
    return (
        <Form {...form}>
          <form onSubmit={(e)=>{
            form.handleSubmit(onSubmit)();}} 
          className="space-y-8 text-black">
            <FormField
              control={form.control}
              name="username"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Username</FormLabel>
                  <FormControl>
                    <Input placeholder="shadcn" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
             <FormField
              control={form.control}
              name="gender"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Gender</FormLabel>
                  <FormControl>
                    <Select onValueChange={field.onChange} value={field.value}>
                      <SelectTrigger className="w-[180px]">
                        <SelectValue placeholder="Select gender" />
                      </SelectTrigger>
                      <SelectContent className=' bg-white text-black'>
                        <SelectItem value="male">Male</SelectItem>
                        <SelectItem value="female">Female</SelectItem>
                        <SelectItem value="others">Others</SelectItem>
                      </SelectContent>
                    </Select>
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            {/* Name Field */}
            <FormField
              control={form.control}
              name="name"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Name</FormLabel>
                  <FormControl>
                    <Input type="text" placeholder="Enter name" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
            {/* Email Field */}
            <FormField
              control={form.control}
              name="email"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Email</FormLabel>
                  <FormControl>
                    <Input type="email" placeholder="Enter email" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
            {/* Password Field */}
            <FormField
              control={form.control}
              name="password"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Password</FormLabel>
                  <FormControl>
                    <Input type="password" placeholder="Enter password" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            {/* Confirm Password Field */}
            <FormField
              control={form.control}
              name="confirmPassword"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Confirm Password</FormLabel>
                  <FormControl>
                    <Input type="password" placeholder="Retype password" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />


            <Button type="submit">Submit</Button>
          </form>
        </Form>
    )
    }

    export default SigninForm
    ```
