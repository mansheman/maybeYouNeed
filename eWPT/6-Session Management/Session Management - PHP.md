2025-08-18 14:13

Status:

Tags: [[eWPT]] [[Session Management]]
###### Prerequisites: 
# Session Management - PHP

## Overview
- PHP provides built-in functions for session handling.  
- Sessions are identified by a unique **session ID**.  

## Steps

### 1. Start a Session
```php
session_start();
````

- Initializes the session.
    
- Generates a unique session ID for the user.
    

### 2. Store and Retrieve Session Data

```php
$_SESSION['username'] = 'john_doe';
echo $_SESSION['username'];
```

- Data is stored in the `$_SESSION` superglobal array.
    

### 3. Session Timeout

- Defined in **php.ini** with `session.gc_maxlifetime`.
    
- Determines how long a session remains active without activity.
    

### 4. Session ID Management

- PHP automatically generates and manages session IDs.
    
- IDs are linked to users for session tracking.
    
