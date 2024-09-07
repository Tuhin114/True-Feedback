# True Feedback

-`npm i mongoose`

## User Model

This schema defines a `User` and `Message` structure using Mongoose with TypeScript integration. The `Message` interface includes basic properties like `content` (required string) and `createdAt` (a timestamp that defaults to the current date). The `User` interface extends the `Document` model and contains typical user data such as `username`, `email`, `password`, and additional fields like `verifyCode` and `verifyCodeExpiry` for email verification. It also includes `isVerified` and `isAcceptingMessages` boolean flags, and an array of `messages` using the `MessageSchema`.

### Key Concepts

- **Schemas and Models**: The `MessageSchema` is nested within the `UserSchema`, which means that each user document can have an array of messages, fully integrated as subdocuments.
- **Validation and Constraints**: The schema enforces several validations such as requiring certain fields (`username`, `email`, etc.), using unique constraints (`username`, `email`), and adding default values (e.g., `isVerified` defaults to `false`).
- **TypeScript Integration**: Interfaces (`Message`, `User`) ensure that the schema strictly adheres to the defined types, offering strong typing for both schemas and document interactions.

This approach provides a robust structure for user data, including message handling, email verification, and general user authentication workflows.

This line is responsible for creating or reusing the `UserModel` in Mongoose. Here’s a detailed breakdown:

### Concept

In Mongoose, models are reusable objects that provide an interface to interact with the database for a particular collection (in this case, `User`). When defining models in a modularized application or with TypeScript, you need to ensure that you don’t redefine a model if it already exists—this avoids conflicts and errors when the application runs multiple times (e.g., during development with hot-reloading).

### Code Breakdown

```typescript
const UserModel =
  (mongoose.models.User as mongoose.Model<User>) ||
  mongoose.model<User>("User", UserSchema);
```

- **`mongoose.models.User`**:
  - `mongoose.models` is an object where Mongoose stores cached models that have already been compiled. If the `User` model has already been defined, it will exist in this object.
  - This part checks whether a model named `"User"` has already been created. If it exists, it avoids recreating it.
  - The `as mongoose.Model<User>` cast ensures that TypeScript understands `mongoose.models.User` is of type `mongoose.Model<User>`, making it compatible with TypeScript type checking.

- **Logical OR (`||`)**:
  - This operator ensures that if `mongoose.models.User` is not found (i.e., it hasn't been created yet), Mongoose will proceed to define the model using `mongoose.model<User>("User", UserSchema)`.
  - In simple terms, it checks for an existing model and, if not found, defines a new one.

- **`mongoose.model<User>("User", UserSchema)`**:
  - This is how you define a new model in Mongoose if it doesn't exist. It takes two arguments:
    - `"User"`: the name of the model.
    - `UserSchema`: the schema definition that describes the structure of documents in the `User` collection.
  - The `<User>` part is a TypeScript generic that tells the `mongoose.model` function that the model it creates will conform to the `User` interface, enforcing type safety.

### Summary

The purpose of this line is to either reuse the existing `User` model (if it has already been defined) or create a new one using the provided schema. This prevents errors from model redefinition, especially when the code is run multiple times, while ensuring strong TypeScript type checking.

## Schemas

-`npm i zod`

In this code, you're using the `zod` library to create validation schemas for user input. Here’s a detailed explanation:

### Zod Overview

Zod is a TypeScript-first schema validation library that helps in validating and parsing data. It ensures that the data conforms to the desired structure and type, offering type-safe validation.

### `usernameValidation` Schema

This block defines the validation rules for a username:

```typescript
export const usernameValidation = z
  .string() // The value must be a string.
  .min(2, "Username must be at least 2 characters") // Minimum of 2 characters.
  .max(20, "Username must not be more than 20 characters") // Maximum of 20 characters (there's a typo in the original code with max(2)).
  .regex(/^[a-zA-Z0-9_]+$/, "Username must not contain special characters"); // Ensures the username contains only alphanumeric characters and underscores.
```

- **`.string()`**: Specifies that the value should be a string.
- **`.min(2)`**: Ensures that the username has at least 2 characters. The second argument provides a custom error message if the condition fails.
- **`.max(20)`**: Limits the username to a maximum of 20 characters.
- **`.regex()`**: Ensures the username only contains letters (both uppercase and lowercase), numbers, or underscores. It disallows special characters like `!`, `@`, etc.

### `signUpSchema`

This defines the validation rules for the entire sign-up form, which includes the username, email, and password fields:

```typescript
export const signUpSchema = z.object({
  username: usernameValidation, // Uses the predefined username validation.
  
  email: z.string().email({ message: "Invalid email address" }), // Ensures a valid email format.
  
  password: z
    .string()
    .min(6, { message: "Password must be at least 6 characters" }), // Ensures the password is a string and is at least 6 characters long.
});
```

- **`z.object({ ... })`**: Creates an object schema to validate multiple fields at once.
- **`email: z.string().email()`**: Validates that the email field is a string and conforms to a valid email format.
- **`password: z.string().min(6)`**: Ensures the password is a string with a minimum length of 6 characters.

### Summary2

This schema helps validate user input for the signup form by enforcing rules on the username, email, and password fields. It ensures that usernames follow specific character requirements, emails are valid, and passwords are sufficiently secure with a minimum length.

## SignUp - Resend Email

-`npm install resend`
-`@react-email/components`

This function handles sending a verification email using the `resend` service and a React-based email template. Here’s a detailed explanation:

### Function Purpose

`sendVerificationEmail` is an asynchronous function that sends an email to a user to verify their email address. It uses a custom React component (`VerificationEmail`) to render the email content dynamically with the user's `username` and a `verifyCode`. If the email is sent successfully, it returns a success response; otherwise, it returns an error message.

### Breakdown

1. **Function Signature**:

   ```typescript
   export async function sendVerificationEmail(
     email: string,
     username: string,
     verifyCode: string
   ): Promise<ApiResponse>
   ```

   - **Parameters**:
     - `email`: The recipient’s email address.
     - `username`: The user's name (used in the email content).
     - `verifyCode`: A unique code for verifying the email.
   - **Return Type**:
     - The function returns a `Promise<ApiResponse>`, indicating an asynchronous operation that resolves to an object of type `ApiResponse`. This type likely contains a success flag and a message.

2. **Using the Resend Email API**:

   ```typescript
   await resend.emails.send({
     from: "onboarding@resend.dev",
     to: email,
     subject: "True feedback Verification Code",
     react: VerificationEmail({ username, otp: verifyCode }),
   });
   ```

   - **resend.emails.send()**:
     - This sends the email using the `resend` email service.
   - **from**: The sender's email address, here `"onboarding@resend.dev"`.
   - **to**: The recipient’s email address, passed as an argument.
   - **subject**: The subject line of the email.
   - **react**: This takes a React component (`VerificationEmail`) that dynamically generates the content of the email. The component receives `username` and `otp` (the verification code) as props.

3. **React-based Email Template**:

   ```typescript
   react: VerificationEmail({ username, otp: verifyCode })
   ```

   - The `VerificationEmail` component is a React email template that formats the email content. This allows for more flexible and reusable email designs, as React can render components even for emails.
   - The function passes `username` and `verifyCode` as props to this component to personalize the email.

4. **Error Handling**:

   ```typescript
   } catch (emailError) {
     console.error("Error sending verification email:", emailError);
     return { success: false, message: "Failed to send verification email." };
   }
   ```

   - If an error occurs during the email sending process, the function catches it and logs the error to the console.
   - The function then returns an object with `success: false` and an appropriate failure message.

5. **Success Response**:

   ```typescript
   return { success: true, message: "Verification email sent successfully." };
   ```

   - If the email is successfully sent, the function returns an object with `success: true` and a success message.

### Summary3

This function sends a verification email using the `resend` email service and a React-based email template. It accepts `email`, `username`, and `verifyCode` as parameters, attempts to send the email, and returns a success or error response depending on the outcome. This structure ensures that the email content is both dynamic and reusable, with clear error handling.

This `VerificationEmail` component uses `@react-email/components` to structure an HTML email. It personalizes the email by receiving `username` and `otp` as props and generates the content dynamically. The email uses `<Html>`, `<Head>`, and `<Font>` to set metadata and font styling, with a fallback font for compatibility. The body includes a greeting with the `username`, a verification message, and the `otp`. There's also a commented-out `<Button>` that could link to a verification URL, though it's not currently active. This template ensures cross-client compatibility while offering a clean, responsive design for verification emails.

## Sign-up Functionality

-`npm i bcrypt.js`

### User Registration API Handler

This function handles user registration, ensuring that the user provides a unique `username` and `email`, secures the password, and sends a verification email to the user.

#### 1. **Database Connection**

- Before processing the request, the function ensures that a connection to the MongoDB database is established using `dbConnect()`. This is essential for performing any operations on the `UserModel`.

   ```typescript
   await dbConnect();
   ```

#### 2. **Request Parsing**

- The `POST` function expects a `username`, `email`, and `password` in the request body. It parses the incoming request as JSON:

   ```typescript
   const { username, email, password } = await request.json();
   ```

#### 3. **Username Validation**

- The function checks if a verified user with the same `username` already exists in the database:

   ```typescript
   const existingVerifiedUserByUsername = await UserModel.findOne({
     username,
     isVerified: true,
   });

   if (existingVerifiedUserByUsername) {
     return Response.json(
       {
         success: false,
         message: 'Username is already taken',
       },
       { status: 400 }
     );
   }
   ```

- If such a user exists, it responds with a `400` status and an error message indicating that the username is already taken.

#### 4. **Email Validation**

- The function checks if the `email` is already associated with a user in the system:

   ```typescript
   const existingUserByEmail = await UserModel.findOne({ email });
   ```

- **If the email is verified**:
  - If the user is already verified, the function returns a `400` status with a message stating that the email is already registered.

     ```typescript
     if (existingUserByEmail.isVerified) {
       return Response.json(
         {
           success: false,
           message: 'User already exists with this email',
         },
         { status: 400 }
       );
     }
     ```

- **If the email is not verified**:
  - If the email exists but is unverified, the password is updated by hashing the new password, and the `verifyCode` is regenerated.
  - The `verifyCodeExpiry` is updated to an hour from the current time.

     ```typescript
     const hashedPassword = await bcrypt.hash(password, 10);
     existingUserByEmail.password = hashedPassword;
     existingUserByEmail.verifyCode = verifyCode;
     existingUserByEmail.verifyCodeExpiry = new Date(Date.now() + 3600000);
     await existingUserByEmail.save();
     ```

#### 5. **New User Registration**

- If the email is not found in the database, a new user is created with the provided details:

- **Password Hashing**:
  - The password is hashed securely using `bcrypt` to ensure it is not stored in plain text.

     ```typescript
     const hashedPassword = await bcrypt.hash(password, 10);
     ```

- **Verification Code Generation**:
  - A 6-digit `verifyCode` is generated, valid for 1 hour (`verifyCodeExpiry`).

     ```typescript
     let verifyCode = Math.floor(100000 + Math.random() * 900000).toString();
     ```

- **Creating a New User**:
  - A new user is created with properties like `username`, `email`, `password`, `verifyCode`, and `isVerified: false`. The default value for `isVerified` is `false` because the user has yet to verify their account.
  - The `messages` array is initialized as empty, and `isAcceptingMessages` is set to `true`.

     ```typescript
     const newUser = new UserModel({
       username,
       email,
       password: hashedPassword,
       verifyCode,
       verifyCodeExpiry: expiryDate,
       isVerified: false,
       isAcceptingMessages: true,
       messages: [],
     });

     await newUser.save();
     ```

#### 6. **Sending Verification Email**

- After successfully creating or updating the user, the function sends a verification email containing the OTP using the `sendVerificationEmail()` helper:

   ```typescript
   const emailResponse = await sendVerificationEmail(
     email,
     username,
     verifyCode
   );
   ```

- If the email fails to send, the function responds with a `500` status and an appropriate error message:

   ```typescript
   if (!emailResponse.success) {
     return Response.json(
       {
         success: false,
         message: emailResponse.message,
       },
       { status: 500 }
     );
   }
   ```

#### 7. **Success Response**

- If everything succeeds (user creation, password update, and email sending), the function responds with a success message and a `201` status, prompting the user to verify their account.

   ```typescript
   return Response.json(
     {
       success: true,
       message: 'User registered successfully. Please verify your account.',
     },
     { status: 201 }
   );
   ```

#### 8. **Error Handling**

- If any error occurs during the registration process, it is caught and logged. The function then responds with a `500` status and a generic error message:

   ```typescript
   catch (error) {
     console.error('Error registering user:', error);
     return Response.json(
       {
         success: false,
         message: 'Error registering user',
       },
       { status: 500 }
     );
   }
   ```

### Key Features

- **Unique Validation**: Ensures both `username` and `email` are unique and appropriately validated.
- **Password Security**: Uses `bcrypt` for secure password hashing.
- **Email Verification**: Sends a 6-digit OTP to the user's email for account verification.
- **Error Handling**: Catches and logs errors during registration, ensuring that the system remains stable.

This organized flow ensures that the registration process is robust, secure, and user-friendly.

## SignIn via NextAuth.js

-`npm install next-auth`

### Detailed Explanation of NextAuth Configuration with Credentials Provider

This configuration is for setting up **NextAuth** in a Next.js application, using **credentials-based authentication** (email/username and password) with MongoDB as the database for managing users. The implementation focuses on secure user login, user verification, and session management using JSON Web Tokens (JWT).

---

### 1. **Providers: CredentialsProvider**

The **`CredentialsProvider`** is one of the built-in NextAuth providers that enables authentication using a user's credentials (email/username and password). This provider is customizable and allows you to define how the login process should work.

#### **Structure:**

```typescript
providers: [
  CredentialsProvider({
    id: 'credentials',
    name: 'Credentials',
    credentials: {
      email: { label: 'Email', type: 'text' },
      password: { label: 'Password', type: 'password' },
    },
    async authorize(credentials: any): Promise<any> {
      await dbConnect();
      ...
    },
  }),
],
```

- **`id`**: A unique identifier for the provider. Here, it's set as `'credentials'`.
- **`name`**: The display name of the provider, which will be shown on the sign-in page.
- **`credentials`**: This object defines the fields needed for authentication. In this case, we require `email` (or username) and `password`.

---

### 2. **Authorize Function**

The `authorize` function handles the core authentication logic. It performs the following tasks:

#### **Database Connection:**

```typescript
await dbConnect();
```

- **`dbConnect()`**: Establishes a connection to the MongoDB database using Mongoose. This ensures that the database is available for querying user data.

#### **User Lookup:**

```typescript
const user = await UserModel.findOne({
  $or: [
    { email: credentials.identifier },
    { username: credentials.identifier },
  ],
});
```

- **`credentials.identifier`**: The user can input either an email or a username to log in. The query checks for a user document where the email or username matches the provided identifier.
- **`$or`**: This MongoDB operator allows us to query for documents that match either of the specified conditions (email or username).

#### **Error Handling for Missing Users:**

```typescript
if (!user) {
  throw new Error('No user found with this email');
}
```

- If no user is found in the database, an error is thrown. This message is used to notify the user that the account does not exist.

#### **Verification Status Check:**

```typescript
if (!user.isVerified) {
  throw new Error('Please verify your account before logging in');
}
```

- If the user exists but has not yet verified their account (i.e., `isVerified` is `false`), the function throws an error, prompting the user to verify their account first.

#### **Password Validation:**

```typescript
const isPasswordCorrect = await bcrypt.compare(
  credentials.password,
  user.password
);
if (isPasswordCorrect) {
  return user;
} else {
  throw new Error('Incorrect password');
}
```

- The provided password is compared to the hashed password stored in the database using **`bcrypt.compare()`**.
- If the passwords match, the user object is returned to complete the login process.
- If the password is incorrect, an error is thrown.

#### **Error Handling:**

```typescript
catch (err: any) {
  throw new Error(err);
}
```

- If any other errors occur during the authorization process (e.g., database issues), the error is caught and thrown.

---

### 3. **JWT Callbacks**

NextAuth uses JWT tokens to manage user sessions. The **callbacks** object allows us to modify the behavior of token generation and session management.

#### **JWT Callback:**

```typescript
async jwt({ token, user }) {
  if (user) {
    token._id = user._id?.toString();
    token.isVerified = user.isVerified;
    token.isAcceptingMessages = user.isAcceptingMessages;
    token.username = user.username;
  }
  return token;
}
```

- **Purpose**: This callback is triggered when a JWT is being created or updated.
- **Storing User Information**: If the `user` object is available (which happens after successful login), additional user details such as `_id`, `isVerified`, `isAcceptingMessages`, and `username` are added to the token.
- **Token Conversion**: The MongoDB `ObjectId` is converted to a string (`user._id?.toString()`), since tokens only handle string data.

#### **Session Callback:**

```typescript
async session({ session, token }) {
  if (token) {
    session.user._id = token._id;
    session.user.isVerified = token.isVerified;
    session.user.isAcceptingMessages = token.isAcceptingMessages;
    session.user.username = token.username;
  }
  return session;
}
```

- **Purpose**: This callback is invoked when the session object is created. The session object is sent to the frontend and contains user-specific information.
- **Storing Token Data in Session**: The JWT token data is transferred to the session object. This makes fields like `_id`, `isVerified`, and `username` available in the frontend session, allowing client-side apps to easily access these details.

---

### 4. **Session Strategy**

```typescript
session: {
  strategy: 'jwt',
},
```

- **Strategy**: The session strategy is set to `'jwt'`, meaning that the session is entirely token-based. This avoids the need for server-side session storage, as everything is handled client-side using JWT tokens.

---

### 5. **Secret and Pages**

#### **Secret:**

```typescript
secret: process.env.NEXTAUTH_SECRET,
```

- **Purpose**: The secret is used to sign the JWT tokens. It is stored in the environment variable **`NEXTAUTH_SECRET`** and should be a strong, secure string to ensure token integrity.

#### **Custom Pages:**

```typescript
pages: {
  signIn: '/sign-in',
},
```

- **Custom Sign-in Page**: This specifies the route for a custom sign-in page (`/sign-in`). If a user is unauthenticated and tries to access a protected route, they will be redirected to this page.

---

### Summary of Key Features

- **Credentials-Based Authentication**: Users can log in with their email or username and password.
- **User Verification**: Only verified users can log in. Unverified users are prompted to verify their account first.
- **Password Security**: Passwords are securely hashed with **bcrypt** and validated during login.
- **JWT-Based Session Management**: Sessions are lightweight and secure, using JSON Web Tokens (JWT). This strategy eliminates the need for server-side session storage.
- **Custom Sign-in Page**: You can define a custom page for user login, allowing for better user experience and branding.

This setup provides a secure, scalable, and flexible authentication solution using **NextAuth** and **MongoDB** in a Next.js application.

This code extends the type definitions of `next-auth` for both the **Session** and **JWT** interfaces. Here's a detailed breakdown:

### 1. **Purpose of Type Declaration Extension**

NextAuth provides some default types for `Session`, `User`, and `JWT`. However, you may want to include additional fields like `_id`, `isVerified`, `isAcceptingMessages`, and `username` to manage specific user properties in the authentication flow. This TypeScript declaration file augments the default types, enabling you to store and access custom properties in the user session and JWT tokens.

---

### 2. **Extending `Session` Interface**

```typescript
interface Session {
  user: {
    _id?: string;
    isVerified?: boolean;
    isAcceptingMessages?: boolean;
    username?: string;
  } & DefaultSession["user"];
}
```

- **`Session`**: The session object holds the information that is passed between the server and the client. Here, you add the following custom fields:
  - **`_id`**: The MongoDB ObjectId of the user, stored as a string.
  - **`isVerified`**: A boolean indicating whether the user has verified their email.
  - **`isAcceptingMessages`**: A boolean showing whether the user has allowed incoming messages.
  - **`username`**: The username of the user.
  
  The `& DefaultSession["user"]` ensures that the default `user` properties (like `email`, `name`, etc.) are still available alongside the custom fields.

---

### 3. **Extending `User` Interface**

```typescript
interface User {
  _id?: string;
  isVerified?: boolean;
  isAcceptingMessages?: boolean;
  username?: string;
}
```

- **`User`**: Represents the user object in NextAuth. The custom fields added here mirror the ones added to the `Session` interface (`_id`, `isVerified`, `isAcceptingMessages`, and `username`).

This interface ensures that these fields are present when you retrieve the user data directly (for instance, after the `authorize` function in your NextAuth options).

---

### 4. **Extending `JWT` Interface**

```typescript
interface JWT {
  _id?: string;
  isVerified?: boolean;
  isAcceptingMessages?: boolean;
  username?: string;
}
```

- **`JWT`**: The JWT interface is responsible for defining the token's payload that is generated during authentication. This type extension adds the same fields (`_id`, `isVerified`, `isAcceptingMessages`, `username`) to the token, ensuring that this information is stored and accessible in the JWT.

---

### 5. **Why Use `declare module`?**

`declare module` allows you to extend the types of external libraries—in this case, **NextAuth**. The `declare module "next-auth"` and `declare module "next-auth/jwt"` sections ensure that these type extensions are globally available within your project when working with NextAuth and its JWT handling.

---

### 6. **Key Points**

- This type extension ensures that your session and JWT can store additional fields (`_id`, `isVerified`, etc.) alongside the default properties.
- You can access and modify these fields when handling authentication logic in your NextAuth callbacks (e.g., `session`, `jwt`, and `authorize` functions).
- These extended fields make it easy to implement features like user verification, message handling, or any other custom logic specific to your application.

This structure enhances type safety in your Next.js app by making sure TypeScript knows about the custom fields you're working with when using NextAuth.

### Middleware

This middleware file is designed to handle authentication and route redirection for specific paths in a Next.js application using **NextAuth**. Here's a detailed explanation of how this middleware works:

### Key Concept

1. **`NextRequest` and `NextResponse`**: These are used to interact with the request and response objects in the middleware. `NextRequest` gives access to details about the incoming request (like headers, URL, etc.), and `NextResponse` allows sending a response, redirect, or continuing the request flow.
  
2. **`getToken` from `next-auth/jwt`**: This function is used to extract the authentication token from the request. The token indicates if a user is logged in, and it contains the session details.

3. **URL Matching**:
   - The `config` section defines which paths this middleware applies to, using the `matcher` property:

     ```js
     matcher: ['/dashboard/:path*', '/sign-in', '/sign-up', '/', '/verify/:path*']
     ```

     This means that the middleware will run for:
     - `/dashboard/*` paths
     - `/sign-in`, `/sign-up`, `/` (home page), and `/verify/*` paths

### How the Middleware Works

1. **Checking User Authentication (`token`)**:
   The middleware first attempts to get the authentication token using `getToken`. If the token exists, it indicates that the user is authenticated.

2. **Redirect Logic**:

   - **Redirect authenticated users away from public pages**:
     If the user is authenticated (`token` exists) and is trying to access **public routes** such as:
     - `/sign-in`
     - `/sign-up`
     - `/verify/*`
     - `/` (home page)

     The user will be **redirected to the dashboard** (`/dashboard`). This ensures that authenticated users don't access sign-up or sign-in pages again.

     ```ts
     if (
       token &&
       (url.pathname.startsWith('/sign-in') ||
         url.pathname.startsWith('/sign-up') ||
         url.pathname.startsWith('/verify') ||
         url.pathname === '/')
     ) {
       return NextResponse.redirect(new URL('/dashboard', request.url));
     }
     ```

   - **Redirect unauthenticated users away from protected pages**:
     If the user is **not authenticated** (no token) and tries to access a **protected route** like `/dashboard/*`, they are **redirected to the sign-in page** (`/sign-in`).

     ```ts
     if (!token && url.pathname.startsWith('/dashboard')) {
       return NextResponse.redirect(new URL('/sign-in', request.url));
     }
     ```

3. **Allow Access if Conditions Are Met**:
   If none of the redirection conditions are met, the middleware allows the request to continue to the requested page by returning `NextResponse.next()`.

   ```ts
   return NextResponse.next();
   ```

### Summary of Middleware Flow

- **Authenticated users** trying to access **sign-in**, **sign-up**, **verify**, or the **home page** will be **redirected to the dashboard**.
- **Unauthenticated users** trying to access **dashboard** routes will be **redirected to the sign-in page**.
- Other requests proceed normally.

### Next Steps

To use this, ensure you have the following:

- Proper setup of **NextAuth** with JWT tokens.
- Correct routes defined for **sign-in**, **sign-up**, **dashboard**, and **verify** pages.

This approach provides simple route protection and redirection based on authentication status.

## Check username schema via zod and uniqueness

This code defines a `GET` request handler that checks if a username is unique and available in the database. It utilizes `zod` for schema validation and Mongoose to interact with a MongoDB database.

### Explanation

1. **Database Connection**:
   - The `dbConnect` function is called at the start to establish a connection with the MongoDB database. This ensures that the database is ready to handle queries.

   ```ts
   await dbConnect();
   ```

2. **Schema Validation**:
   - The `zod` library is used to define a schema, `UsernameQuerySchema`, which validates the `username` query parameter. It uses `usernameValidation` from the `signUpSchema` file to ensure that the username follows predefined validation rules.

   ```ts
   const result = UsernameQuerySchema.safeParse(queryParams);
   ```

   - If the validation fails, the errors are formatted and returned with a `400` status code, signaling that the input is invalid.

   ```ts
   if (!result.success) {
      const usernameErrors = result.error.format().username?._errors || [];
      return Response.json(
        {
          success: false,
          message: usernameErrors.length > 0
              ? usernameErrors.join(', ')
              : 'Invalid query parameters',
        },
        { status: 400 }
      );
    }
   ```

3. **Username Query**:
   - The `username` is extracted from the query parameters using `searchParams.get()`, and then validated using the schema.
   - If the validation succeeds, the code checks if the username is already taken by querying the database for an existing, verified user.

   ```ts
   const existingVerifiedUser = await UserModel.findOne({
      username,
      isVerified: true,
    });
   ```

   - If a verified user with the given username exists, a response is sent indicating that the username is already taken.

   ```ts
   if (existingVerifiedUser) {
      return Response.json(
        {
          success: false,
          message: 'Username is already taken',
        },
        { status: 200 }
      );
    }
   ```

4. **Response**:
   - If the username is not taken, the response indicates that the username is available and unique.

   ```ts
   return Response.json(
     {
       success: true,
       message: 'Username is unique',
     },
     { status: 200 }
   );
   ```

5. **Error Handling**:
   - In case of any unexpected errors (like database or server issues), a `500` status code is returned with an appropriate error message.

   ```ts
   return Response.json(
     {
       success: false,
       message: 'Error checking username',
     },
     { status: 500 }
   );
   ```

### Summary1

- The handler checks if a username is already taken by querying the database for verified users.
- It uses `zod` for query validation to ensure the username meets the required format.
- It returns a JSON response indicating whether the username is unique or already taken.

## OTP Verification

Here’s a more detailed explanation of the core parts of the verification flow:

### 1. **Database Connection (`dbConnect()`)**

- **Purpose**: This function ensures that the app connects to the MongoDB database before executing any logic.
- **Importance**: MongoDB operations require an active connection, and by ensuring the connection first, we avoid errors related to trying to access the database without a connection.
- **Implementation**: This method is called at the start of the `POST` function to make sure all database-related queries (finding users, saving updates) have access to the database.

### 2. **Extracting Data from the Request**

- **Purpose**: The `POST` method is expecting JSON data in the request body, specifically the `username` and `code`.
- **Details**:
  - `const { username, code } = await request.json();` extracts the `username` and `code` properties from the incoming JSON body.
  - `decodeURIComponent(username);` decodes the `username` in case it's URL-encoded (i.e., contains special characters like `%20` for spaces).
- **Why URL-Decoding?**: If the username was passed via a URL and has encoded characters, `decodeURIComponent` converts those into human-readable characters (e.g., `%40` becomes `@`). This ensures that the database can correctly match the username as stored.

### 3. **Finding the User**

- **Purpose**: The code attempts to find a user with the provided `username` in the database.
- **Query**: `await UserModel.findOne({ username: decodedUsername });` looks for a single user document where the `username` matches the provided `username`.
- **Error Handling**:
  - If no user is found (`if (!user)`), the function immediately returns a response with `404 Not Found` indicating that the username doesn't exist.

### 4. **Checking the Verification Code**

- **Purpose**: The app needs to verify if the user’s provided code matches the one stored in the database and check if it’s still valid (i.e., hasn’t expired).
- **Logic**:
  - `isCodeValid`: Checks whether the code in the request matches the one stored in the user’s record (`user.verifyCode === code`).
  - `isCodeNotExpired`: Compares the expiration time of the code (`verifyCodeExpiry`) with the current time to ensure it hasn’t expired. This is done by comparing the stored expiry date to the current time (`new Date()`).
- **Expiration Logic**: The verification code has a time limit (in this case, it's assumed to be one hour, based on the stored `verifyCodeExpiry`), and the code is only valid before this expiration time.

### 5. **Handling Various Verification Cases**

- **Case 1: Successful Verification**:
  - If both `isCodeValid` and `isCodeNotExpired` are true, the user's verification status is updated:
    - `user.isVerified = true;` marks the user as verified.
    - `await user.save();` saves this change to the database.
  - The server then responds with a `200 OK` status and a message indicating that the account has been successfully verified.

- **Case 2: Expired Code**:
  - If the verification code has expired (`!isCodeNotExpired`), the user is prompted to sign up again to receive a new code:
    - Returns a `400 Bad Request` response with a message that the code has expired and a new one is required.

- **Case 3: Incorrect Code**:
  - If the code is incorrect but has not expired, a response with a `400 Bad Request` status is sent, indicating the verification code is incorrect.

- **Importance of These Conditions**: Handling both expired and incorrect codes separately provides clear and specific feedback to the user, helping them understand why the verification failed.

### 6. **Error Handling (`try-catch`)**

- **Purpose**: The `try-catch` block ensures that if any unexpected error occurs during the execution of the code (e.g., database connection fails or some internal error), it is caught, and an appropriate response is returned.
- **In the Catch Block**:
  - `console.error('Error verifying user:', error);` logs the actual error to the console for debugging purposes.
  - The response sent to the user includes a generic message: `"Error verifying user"` with a `500 Internal Server Error` status.
- **Why Log the Error?**: This ensures that while the user doesn’t see sensitive details, the developers can still debug the issue from the server logs.

### Response Structure

- The response in each case (`success: true/false`) follows a standard format, returning:
     1. **Success**: Indicates whether the operation (verification) was successful.
     2. **Message**: Provides a clear message describing the outcome (e.g., account verified, incorrect code, expired code).
- **Why This Structure?**: Having a consistent response format allows the frontend to handle these responses uniformly, showing appropriate messages to the user depending on the situation.

### Summary of Key Points

1. **Database Connection**: Ensures MongoDB connection before proceeding with user operations.
2. **Request Parsing**: Extracts and decodes `username` and `code` from the request body.
3. **User Lookup**: Searches the database for the user by their `username`.
4. **Verification Logic**: Checks if the provided verification code matches the one stored in the database and if it hasn’t expired.
5. **Verification Outcome**:
   - Updates the user’s status if the code is valid and not expired.
   - Handles expired or incorrect codes with appropriate responses.
6. **Error Handling**: Logs errors and returns a generic response to the user if any unexpected issues arise.

This approach effectively handles the entire verification process while ensuring that the user’s actions (submitting a verification code) are validated thoroughly and errors are properly managed.
