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
