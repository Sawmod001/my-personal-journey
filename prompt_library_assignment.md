# Prompt Library for AI Coding Assistants

## Introduction

This prompt library contains three prompts designed for software development tasks: code generation, debugging, and refactoring. Each prompt was tested using an AI coding assistant (ChatGPT).

---

# 1. Code Generation Prompt

## Prompt

```
Act as a senior backend developer.

Create a REST API using Node.js, Express.js, and MongoDB for managing job applications.

Requirements:
- POST /jobs → Create a new job application
- GET /jobs → Retrieve all job applications
- GET /jobs/:id → Retrieve a single job application
- PUT /jobs/:id → Update a job application
- DELETE /jobs/:id → Delete a job application

Each job application should contain:
- Company Name
- Job Role
- Date Applied
- Application Status

Use MVC architecture and include error handling.
```

## Test Result

**AI Assistant Used:** ChatGPT

### Outcome

The AI generated:

- Express server setup
- MongoDB connection
- Job model schema
- Controller functions
- Route definitions
- CRUD operations
- Basic error handling

### Evaluation

✅ Correct CRUD implementation

✅ Followed MVC structure

✅ Included MongoDB integration

---

# 2. Debugging Prompt

## Prompt

```
Act as a software debugging expert.

Analyze the following code and identify:
1. The cause of the error
2. Why it occurs
3. The corrected version of the code
4. Best practices to prevent similar issues

Code:

function divide(a, b) {
    return a / b;
}

console.log(divide(10));
```

## Test Result

**AI Assistant Used:** ChatGPT

### Outcome

The AI identified that:

- The second parameter (b) is missing.
- Division by undefined returns NaN.
- Input validation should be added.

### Corrected Code

```javascript
function divide(a, b) {
    if (b === undefined) {
        throw new Error("Second parameter is required");
    }

    return a / b;
}

console.log(divide(10, 2));
```

### Evaluation

✅ Correctly identified the issue

✅ Provided a working solution

✅ Suggested validation techniques

---

# 3. Refactoring Prompt

## Prompt

```
Act as a senior software engineer.

Refactor the following code to improve:
- Readability
- Maintainability
- Performance (where applicable)
- Adherence to JavaScript best practices

Explain all changes made.

Code:

function getUserData(users){
for(let i=0;i<users.length;i++){
if(users[i].active == true){
console.log(users[i].name);
}
}
}
```

## Test Result

**AI Assistant Used:** ChatGPT

### Refactored Code

```javascript
function getActiveUserNames(users) {
    users
        .filter(user => user.active)
        .forEach(user => console.log(user.name));
}
```

### Changes Made

1. Renamed function to better describe its purpose.
2. Improved formatting and indentation.
3. Used array methods (`filter` and `forEach`) for readability.
4. Replaced `== true` with a direct boolean check.
5. Improved code maintainability.

### Evaluation

✅ Cleaner and easier to understand

✅ Follows modern JavaScript practices

✅ Maintains original functionality

---

# Conclusion

This prompt library demonstrates how AI coding assistants can support software development through code generation, debugging, and refactoring. Properly structured prompts produce more accurate, maintainable, and efficient code solutions.
