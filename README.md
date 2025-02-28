# Exercise: Implement JWT Auth in the Time Capsule Social Network

## Learning Goals

Upon completing this exercise, you will be able to:

- Implement JWT-based authentication in an Express application
- Secure API endpoints using JWT middleware
- Handle user authentication and authorization
- Protect routes based on resource ownership
- Manage JWT tokens securely

## Introduction

In this exercise, you'll enhance your Time Capsule Social Network by adding JWT authentication.

Building upon your existing implementation that already has MongoDB integration and relationships between users, time capsules, and comments, you'll implement secure authentication to ensure users can only access and modify their own resources.


> [!TIP]
> Please remember, we currently have two versions of the Time Capsule Social Network:
> 
> 1. The original exercise
> 2. The refactored version that follows the Controller-Service-Repository (CSP) architecture pattern.
>
> For simplicity, we will build upon the original exercise as continuing with the CSP pattern would add unnecessary complexity for this exercise. You'll focus on implementing JWT authentication within the existing structure. 🙂


## Getting Started

1. Start here by [forking this repository](https://github.com/codevergehq/time-capsule-social-network-with-jwt)
2. Now clone your forked repository: 
   
```bash
git clone https://github.com/codevergehq/time-capsule-social-network-with-jwt.git
```

3. Navigate into your newly cloned repository:

```bash
cd time-capsule-social-network-with-jwt
```

> [!TIP]
> What you've forked and cloned is essentially an empty project (except for the `README.md` file). We'll now copy your previous project files into this repository.


4. While you’re in the new local directory, run the following command to copy your previous project files into the new directory:

```bash
# Assuming your previous project is in the same parent directory
cp -r ../time-capsule-social-network/* .
```

> [!TIP]
> The command above assumes both projects are in the same parent directory. If your `time-capsule-social-network` is located elsewhere, adjust the path accordingly.

5. Install all dependencies:

```bash
# Install existing dependencies
npm install

# Install additional dependencies for JWT authentication
npm install jsonwebtoken bcrypt dotenv
```

## Instructions

### Task 1: Set up Environment Variables

1. First, create a `.env` file in your project root:

```bash
touch .env
```

2. Add the following to your `.env` file:

```bash
MONGODB_URI=mongodb://localhost:27017/time-capsule-social-network
JWT_SECRET=your-jwt-secret-here
PORT=3000
```

Don't forget to overwrite the default value with a secure random string. The important thing is to use a long, random string that would be difficult to guess.

You can generate a secure random string using Node's `crypto` module:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Just copy the resulting value from the terminal and add it to your `.env` file as your `JWT_SECRET`.

This will be used to sign your JWT tokens, so keep it secure.

> [!TIP]
> The `.gitignore` file already ensures your `.env` file won't be committed to version control, keeping your sensitive information secure.

### Task 2: Configure JWT in Express Application

1. Update your `app.js` to use environment variables and configure JWT:

```jsx
// Load environment variables
require('dotenv').config();

// Import required packages
const express = require('express');
const mongoose = require('mongoose');
const jwt = require('jsonwebtoken');

const app = express();

// Middleware to parse JSON bodies
app.use(express.json());

// Verify JWT configuration by testing token generation
try {
    const testToken = jwt.sign({ test: true }, process.env.JWT_SECRET);
    console.log('JWT Configuration successful');
} catch (error) {
    console.error('JWT Configuration failed:', error.message);
    process.exit(1);
}

// MongoDB connection
mongoose.connect(process.env.MONGODB_URI)
    .then(() => console.log('Connected to MongoDB...'))
    .catch(err => console.error('MongoDB connection error:', err));

// Your routes will be mounted here later...

module.exports = app;

```

> [!TIP]
> The test token generation helps verify that your `JWT_SECRET` is properly configured. 
> If there's any issue with the environment variable or JWT setup, you'll know immediately when starting the application.

After configuring JWT in your `app.js` file, start your server to verify the JWT configuration:

```bash
node app.js
```

You should see these success messages in your console:

```bash
JWT Configuration successful
Connected to MongoDB...
Server is running on port 3000
```

> [!TIP]
> In case you don't see these messages, or if you see any errors, double-check your `.env` file values, package installations or MongoDB connection

Once you've confirmed everything is working, proceed to the next task.

### Task 3: Update User Model

Currently, your `User` model in `models/User.model.js` looks like this:

```jsx
const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    email: { type: String, required: true, unique: true },
    timeCapsules: [{ type: mongoose.Schema.Types.ObjectId, ref: 'TimeCapsule' }]
});
```

We need to add a `password` field for authentication.

Update your `User` schema to include a `password` field.

Finally, add timestamps too, as this is useful for tracking when users registered and when they last updated their profile.

### Task 4: Implement Authentication Routes

1. Create a new file `routes/auth.routes.js`:

```bash
touch routes/auth.routes.js
```

2. Set up the basic structure for authentication routes:

```jsx
const express = require('express');
const router = express.Router();
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const User = require('../models/User.model');

// Routes will go here

module.exports = router;
```

3. Implement the registration route (`POST /register`):
    1. Create a route handler for `POST /register`
    2. Extract username, email, and password from request body
    3. Check if a user with the same email or username already exists
    4. If user exists, return a 400 status with appropriate message
    5. If user doesn't exist, hash the password using bcrypt
    6. Create a new user with the hashed password
    7. Generate a JWT token containing the user's ID
    8. Return the token and user information (excluding password)
    
4. Implement the login route (`POST /login`):
    1. Create a route handler for `POST /login`
    2. Extract email and password from request body
    3. Find the user by email
    4. If user not found, return 401 with "Invalid credentials"
    5. Compare the provided password with the stored hash using bcrypt
    6. If password doesn't match, return 401 with "Invalid credentials"
    7. Generate a JWT token containing the user's ID
    8. Return the token and user information (excluding password)

5. Mount the authentication routes in `app.js` :

```jsx
// In app.js, add this after your middleware setup
const authRoutes = require('./routes/auth.routes');
app.use('/api/auth', authRoutes);
```

6. Test your authentication routes using Bruno before proceeding.

> [!TIP]
> Remember to use the same error message for both "user not found" and "incorrect password" scenarios during login. This is a **security best practice** to prevent attackers from knowing which part of the credentials was incorrect.

### Task 5: Create Authentication Middleware

1. Create a new file `middleware/auth.middleware.js`:

```bash
mkdir -p middleware
touch middleware/auth.middleware.js
```

2. Implement the authentication middleware:
    1. Import the JWT library and User model
    2. Create a middleware function that:
        1. Extracts the token from the Authorization header
        2. Verifies the token using your J`WT_SECRET`
        3. Finds the user by ID from the decoded token
        4. Attaches the user to the request object
        5. Handles errors for missing/invalid tokens
    3. Export the middleware function

### Task 6: Secure Time Capsule Routes

1. Update your time capsule routes in `routes/timeCapsule.routes.js` :
    1. Import the auth middleware
    2. Apply the middleware to protected routes:

```jsx
const authMiddleware = require('../middleware/auth.middleware');

// Protected routes
router.post('/timeCapsules', authMiddleware, createTimeCapsule);
router.put('/timeCapsules/:id', authMiddleware, updateTimeCapsule);
router.delete('/timeCapsules/:id', authMiddleware, deleteTimeCapsule);
```

2. Modify your route handlers to:
    1. Use `req.user._id` as the creator ID for new time capsules
    2. Verify ownership before updates/deletes by comparing `timeCapsule.creator` with `req.user._id` 
    3. Return `403 Forbidden` if a user attempts to modify another user's time capsule

3. For `GET` routes, implement appropriate access control:
    1. Public time capsules should be accessible to all
    2. Private time capsules should only be accessible to their creator or recipients

### Task 7: Secure Comment Routes

1. Update your comment routes in `routes/comment.routes.js`:
    1. Import the auth middleware
    2. Apply the middleware to all comment creation, update, and deletion routes
    3. Leave comment retrieval routes public
2. Modify your comment handlers to:
    1. Use `req.user._id` as the author ID for new comments
    2. Verify ownership before updates / deletes
    3. Return `403 Forbidden` if a user attempts to modify another user's comment

### Task 8: Test Authentication Flow

1. Authentication flow:
    1. Register a new user
    2. Login with the user’s credentials
    3. Save the token for subsequent request
2. Protected routes:
    1. Create a time capsule (include token in Authorization header)
    2. Try to update another user's time capsule (should fail)
    3. Create a comment on a time capsule (with token)
    4. Delete your own comment (with token)
3. Test error cases:
    1. Try accessing protected routes without a token
    2. Try using an invalid token
    3. Try modifying resources you don’t own
4. Use `req.user._id` as the author ID for new comments
    1. Verify ownership before updates/deletes
    2. Return `403 Forbidden` if a user attempts to modify another user’s comment

## Submission

When you’ve completed all tasks:

```bash
git add .
git commit -m "Add JWT authentication to Time Capsule Social Network"
git push origin main
```

Create a Pull Request and Submit your assignment.

## Frequently Asked Questions (FAQ)

<details>
<summary>How do I verify token ownership in protected routes?</summary>
Compare the authenticated user's ID (from token) with the resource's creator ID:
  
```jsx
    if (timeCapsule.creator.toString() !== req.user._id.toString()) {
    
        return res.status(403).json({ error: 'Not authorized' });
    
    }
```
</details>

<details>
<summary>How should I handle expired tokens?</summary>
The auth middleware should check token expiration and return appropriate error responses:

```jsx
    if (error.name === 'TokenExpiredError') {
    
        return res.status(401).json({ error: 'Token expired' });
    
    }
```
</details>
    
<details>
<summary>What's the difference between `401` and `403` status codes?</summary>

- `401 (Unauthorized)`: The user is not authenticated (missing or invalid token)
- `403 (Forbidden)`: The user is authenticated but doesn't have permission to access the resource
</details>
    
