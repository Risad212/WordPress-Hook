# WordPress Hook System – Behind the Scenes

A simple and clear explanation of how `add_action()` and `add_filter()` work internally in WordPress.

No long theory. Just how the engine works.

---

## 1️⃣ Important Truth

```
add_action() === add_filter()
```

In WordPress core:

```php
function add_action($hook, $callback, $priority = 10, $accepted_args = 1) {
    return add_filter($hook, $callback, $priority, $accepted_args);
}
```

Actions and filters use the same internal system.

---

## 2️⃣ Global Storage: `$wp_filter`

All hooks are stored inside a global variable:

```php
global $wp_filter;
```

Structure:

```php
$wp_filter = [
    'hook_name' => WP_Hook Object
];
```

Each hook name stores a `WP_Hook` object.

---

## 3️⃣ What Happens When You Call `add_action()`

Example:

```php
add_action('init', 'my_function', 10, 1);
```

Internally WordPress does something like:

```php
global $wp_filter;

if (!isset($wp_filter['init'])) {
    $wp_filter['init'] = new WP_Hook();
}

$wp_filter['init']->add_filter(
    'init',
    'my_function',
    10,
    1
);
```

### Step-by-step:

1. Check if hook exists  
2. If not → create `WP_Hook` instance  
3. Store callback inside the object  

---

## 4️⃣ Full Internal Structure Example

After:

```php
add_action('init', 'funcA', 10);
add_action('init', 'funcB', 20);
```

The structure becomes:

```php
$wp_filter = [

  'init' => WP_Hook Object
    (
      [callbacks] => [
        10 => [
          'unique_id_1' => [
            'function'      => 'funcA',
            'accepted_args' => 1
          ]
        ],
        20 => [
          'unique_id_2' => [
            'function'      => 'funcB',
            'accepted_args' => 1
          ]
        ]
      ]
    )

];
```

### Structure Explained

- Level 1 → Hook name  
- Level 2 → Priority number  
- Level 3 → Callback data  

---

## 5️⃣ Simplified `WP_Hook` Class

Very simplified version of the real class:

```php
class WP_Hook {

    public $callbacks = [];

    public function add_filter($hook, $callback, $priority, $accepted_args) {

        $this->callbacks[$priority][] = [
            'function'      => $callback,
            'accepted_args' => $accepted_args,
        ];
    }

    public function do_action($args) {

        ksort($this->callbacks);

        foreach ($this->callbacks as $priority => $functions) {
            foreach ($functions as $data) {

                call_user_func_array(
                    $data['function'],
                    $args
                );
            }
        }
    }
}
```

Hooks are just:

- Arrays  
- Objects  
- Loops  
- `call_user_func_array()`  

Nothing magical.

---

## 6️⃣ Execution Flow

When WordPress runs:

```php
do_action('init');
```

Internally:

```
1. Check $wp_filter['init']
2. Call WP_Hook->do_action()
3. Sort priorities
4. Loop through callbacks
5. Execute each function
```

### Visual Flow

```
do_action('init')
        ↓
$wp_filter['init']
        ↓
WP_Hook object
        ↓
callbacks grouped by priority
        ↓
Loop & execute
```

---

## 7️⃣ Why Unique IDs Exist

WordPress creates a unique ID for each callback.

Why?

So this works correctly:

```php
remove_action('init', 'my_function', 10);
```

Without unique IDs, removing callbacks would fail.

---

## 8️⃣ Key Takeaways

- `add_action()` is an alias of `add_filter()`
- Hooks are stored in `$wp_filter`
- Each hook is a `WP_Hook` object
- Callbacks are grouped by priority
- Lower priority runs first
- Internally → arrays + objects + loops

---

## 🧠 Mental Model

```
Event Name → Object → Priority Buckets → Callback List → Execute
```

It’s basically an Event Dispatcher pattern.
