 #  WORDPRESS HOOK SYSTEM - COMPLETE GUIDE

Author: Study Reference for WordPress Developers
Purpose: Understanding WordPress Hook System Core Architecture
Last Updated: 2024

================================================================================
TABLE OF CONTENTS
================================================================================
1. What Are Hooks?
2. Hook Types
3. Core Architecture
4. Global Storage Structure
5. Function Reference
6. Execution Flow
7. Priority System
8. Real Examples
9. Quick Reference

================================================================================
1. WHAT ARE HOOKS?
================================================================================
Hooks allow you to "hook into" WordPress code at specific points to execute
your own custom code without modifying WordPress core files.

Think of hooks as:
- Event listeners that trigger at specific moments
- Extension points where you can inject custom functionality
- A way to modify data or execute actions

================================================================================
2. HOOK TYPES
================================================================================

┌─────────────────────────────────────────────────────────────────┐
│                          HOOKS                                   │
│                            │                                     │
│              ┌─────────────┴─────────────┐                      │
│              │                           │                      │
│         ┌────▼─────┐              ┌──────▼──────┐              │
│         │ ACTIONS  │              │   FILTERS   │              │
│         │          │              │             │              │
│         │ Execute  │              │   Modify    │              │
│         │ code     │              │   & return  │              │
│         │ (no      │              │   data      │              │
│         │ return)  │              │             │              │
│         └──────────┘              └─────────────┘              │
└─────────────────────────────────────────────────────────────────┘

ACTIONS:
- Do something (send email, save data, echo HTML)
- Don't return values
- Examples: wp_head, wp_footer, init, save_post

FILTERS:
- Modify data and return it
- Always return a value
- Examples: the_title, the_content, excerpt_length

================================================================================
3. CORE ARCHITECTURE
================================================================================

┌──────────────────────────────────────────────────────────────────┐
│  REGISTRATION PHASE                                               │
│  ─────────────────                                                │
│                                                                   │
│  add_action('hook_name', 'callback')                             │
│  add_filter('hook_name', 'callback')                             │
│           │                                                       │
│           └─────────► Stores in $wp_filter global array          │
│                                                                   │
├──────────────────────────────────────────────────────────────────┤
│  EXECUTION PHASE                                                  │
│  ───────────────                                                  │
│                                                                   │
│  do_action('hook_name', $args)                                   │
│  apply_filters('hook_name', $value, $args)                       │
│           │                                                       │
│           └─────────► Retrieves callbacks from $wp_filter        │
│                       Executes them in priority order            │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘

================================================================================
4. GLOBAL STORAGE STRUCTURE: $wp_filter
================================================================================

$wp_filter = array(
    'hook_name' => array(
        PRIORITY => array(
            'unique_id' => array(
                'function' => callable,
                'accepted_args' => int
            )
        )
    )
);

EXAMPLE:
--------
$wp_filter = array(
    'the_title' => array(
        10 => array(
            'uppercase_title' => array(
                'function' => 'uppercase_title',
                'accepted_args' => 1
            )
        ),
        20 => array(
            'add_prefix' => array(
                'function' => 'add_prefix',
                'accepted_args' => 1
            )
        )
    )
);

================================================================================
5. FUNCTION REFERENCE
================================================================================

┌──────────────────────────────────────────────────────────────────┐
│ REGISTRATION FUNCTIONS                                            │
└──────────────────────────────────────────────────────────────────┘

add_filter($hook_name, $callback, $priority = 10, $accepted_args = 1)
  ├─ Registers a callback to a filter hook
  ├─ Returns modified data
  └─ Example: add_filter('the_title', 'my_function', 10, 1);

add_action($hook_name, $callback, $priority = 10, $accepted_args = 1)
  ├─ Registers a callback to an action hook
  ├─ Just executes code (no return)
  ├─ Internally calls add_filter()
  └─ Example: add_action('init', 'my_function', 10);

┌──────────────────────────────────────────────────────────────────┐
│ EXECUTION FUNCTIONS                                               │
└──────────────────────────────────────────────────────────────────┘

apply_filters($hook_name, $value, ...$args)
  ├─ Executes all callbacks for a filter
  ├─ Passes $value through each callback
  ├─ Returns the final modified value
  └─ Example: $title = apply_filters('the_title', $title);

do_action($hook_name, ...$args)
  ├─ Executes all callbacks for an action
  ├─ No return value
  └─ Example: do_action('wp_footer');

┌──────────────────────────────────────────────────────────────────┐
│ REMOVAL FUNCTIONS                                                 │
└──────────────────────────────────────────────────────────────────┘

remove_filter($hook_name, $callback, $priority = 10)
  └─ Removes a callback from a filter

remove_action($hook_name, $callback, $priority = 10)
  └─ Removes a callback from an action

has_filter($hook_name, $callback = false)
  └─ Check if filter has callbacks

has_action($hook_name, $callback = false)
  └─ Check if action has callbacks

================================================================================
6. EXECUTION FLOW
================================================================================

FILTER FLOW:
───────────

Input → apply_filters('my_filter', $value)
  │
  ├─► Get callbacks from $wp_filter['my_filter']
  ├─► Sort by priority (10, 20, 30...)
  │
  ├─► Priority 10:
  │     ├─ callback_1($value) → returns $new_value
  │     └─ callback_2($new_value) → returns $newer_value
  │
  ├─► Priority 20:
  │     └─ callback_3($newer_value) → returns $final_value
  │
  └─► Return $final_value


ACTION FLOW:
───────────

do_action('my_action', $arg1, $arg2)
  │
  ├─► Get callbacks from $wp_filter['my_action']
  ├─► Sort by priority
  │
  ├─► Priority 10:
  │     ├─ callback_1($arg1, $arg2) → [executes]
  │     └─ callback_2($arg1, $arg2) → [executes]
  │
  ├─► Priority 20:
  │     └─ callback_3($arg1, $arg2) → [executes]
  │
  └─► Complete (no return)

================================================================================
7. PRIORITY SYSTEM
================================================================================

Priority = Order of execution (lower number = earlier execution)

Priority 1    ──► Executes FIRST (highest priority)
Priority 5    ──► 
Priority 10   ──► DEFAULT priority
Priority 15   ──► 
Priority 20   ──► 
Priority 100  ──► Executes LAST (lowest priority)

EXAMPLE:
--------
add_filter('the_title', 'add_suffix', 20);     // Runs second
add_filter('the_title', 'add_prefix', 10);     // Runs first
add_filter('the_title', 'uppercase', 5);       // Runs before all

Result: uppercase → add_prefix → add_suffix

================================================================================
8. REAL EXAMPLES
================================================================================

┌──────────────────────────────────────────────────────────────────┐
│ FILTER EXAMPLE                                                    │
└──────────────────────────────────────────────────────────────────┘

// Register callbacks
add_filter('the_title', 'make_uppercase', 10, 1);
add_filter('the_title', 'add_emoji', 20, 1);

function make_uppercase($title) {
    return strtoupper($title);
}

function add_emoji($title) {
    return '🎉 ' . $title;
}

// Execute
$title = 'hello world';
$result = apply_filters('the_title', $title);
// Result: "🎉 HELLO WORLD"

FLOW:
"hello world" → make_uppercase → "HELLO WORLD" → add_emoji → "🎉 HELLO WORLD"


┌──────────────────────────────────────────────────────────────────┐
│ ACTION EXAMPLE                                                    │
└──────────────────────────────────────────────────────────────────┘

// Register callbacks
add_action('wp_footer', 'add_copyright', 10);
add_action('wp_footer', 'add_analytics', 20);

function add_copyright() {
    echo '<p>© 2024 My Site</p>';
}

function add_analytics() {
    echo '<script>console.log("Tracked");</script>';
}

// Execute
do_action('wp_footer');

OUTPUT:
<p>© 2024 My Site</p>
<script>console.log("Tracked");</script>


┌──────────────────────────────────────────────────────────────────┐
│ MULTIPLE ARGUMENTS EXAMPLE                                        │
└──────────────────────────────────────────────────────────────────┘

// Register with 3 arguments
add_filter('custom_price', 'apply_discount', 10, 3);

function apply_discount($price, $user_id, $product_id) {
    if ($user_id === 123) {
        return $price * 0.9; // 10% discount
    }
    return $price;
}

// Execute
$final_price = apply_filters('custom_price', 100, 123, 456);
// Result: 90


┌──────────────────────────────────────────────────────────────────┐
│ REMOVING HOOKS EXAMPLE                                            │
└──────────────────────────────────────────────────────────────────┘

// Remove a specific callback
remove_action('wp_footer', 'add_analytics', 20);

// Remove all callbacks from a hook
remove_all_actions('wp_footer');

================================================================================
9. QUICK REFERENCE CHEAT SHEET
================================================================================

┌─────────────────────┬──────────────────────────────────────────────┐
│ FUNCTION            │ PURPOSE                                      │
├─────────────────────┼──────────────────────────────────────────────┤
│ add_filter()        │ Register filter callback                     │
│ add_action()        │ Register action callback                     │
│ apply_filters()     │ Execute filter, return modified value        │
│ do_action()         │ Execute action, no return                    │
│ remove_filter()     │ Unregister filter callback                   │
│ remove_action()     │ Unregister action callback                   │
│ has_filter()        │ Check if filter exists                       │
│ has_action()        │ Check if action exists                       │
│ current_filter()    │ Get name of current filter being executed    │
│ current_action()    │ Get name of current action being executed    │
│ doing_filter()      │ Check if specific filter is being executed   │
│ doing_action()      │ Check if specific action is being executed   │
└─────────────────────┴──────────────────────────────────────────────┘

================================================================================
COMMON PATTERNS
================================================================================

1. CONDITIONAL HOOK EXECUTION
   if (!has_action('my_hook', 'my_callback')) {
       add_action('my_hook', 'my_callback');
   }

2. PRIORITY MANAGEMENT
   // Run before other callbacks
   add_filter('the_title', 'my_filter', 5);
   
   // Run after other callbacks
   add_filter('the_title', 'my_filter', 999);

3. CLASS METHODS AS CALLBACKS
   class MyClass {
       public function __construct() {
           add_action('init', array($this, 'my_method'));
       }
       
       public function my_method() {
           // Code here
       }
   }

4. ANONYMOUS FUNCTIONS
   add_filter('the_title', function($title) {
       return strtoupper($title);
   }, 10, 1);

5. ACCEPT MULTIPLE ARGUMENTS
   add_filter('my_hook', 'my_callback', 10, 3);
   
   function my_callback($arg1, $arg2, $arg3) {
       // All 3 arguments available
       return $arg1;
   }

================================================================================
KEY DIFFERENCES: ACTIONS vs FILTERS
================================================================================

┌─────────────────────┬─────────────────┬─────────────────────────────┐
│ FEATURE             │ ACTIONS         │ FILTERS                     │
├─────────────────────┼─────────────────┼─────────────────────────────┤
│ Purpose             │ Execute code    │ Modify data                 │
│ Return value        │ None (void)     │ Required                    │
│ Registration        │ add_action()    │ add_filter()                │
│ Execution           │ do_action()     │ apply_filters()             │
│ Use case            │ Send email,     │ Change title, modify        │
│                     │ save data       │ content, alter arrays       │
│ Example             │ wp_footer       │ the_title                   │
└─────────────────────┴─────────────────┴─────────────────────────────┘

================================================================================
BEST PRACTICES
================================================================================

1. Always return a value in filters
   ✓ return $value;
   ✗ echo $value;

2. Use unique function names to avoid conflicts
   ✓ mytheme_uppercase_title()
   ✗ uppercase_title()

3. Use appropriate priority
   - Use 10 (default) for most cases
   - Use 5-9 to run early
   - Use 11-99 to run late

4. Specify accepted_args when using multiple parameters
   add_filter('my_hook', 'my_callback', 10, 3);

5. Remove hooks in the same priority they were added
   remove_action('init', 'my_function', 10);

6. Check if hook exists before removing
   if (has_action('init', 'unwanted_function')) {
       remove_action('init', 'unwanted_function');
   }

================================================================================
CORE IMPLEMENTATION (SIMPLIFIED)
================================================================================

<?php
// Global storage for all hooks
global $wp_filter;

// Adding a hook
function add_filter($hook, $callback, $priority = 10, $args = 1) {
    global $wp_filter;
    $wp_filter[$hook][$priority]['id'] = [
        'function' => $callback,
        'accepted_args' => $args
    ];
}

// Executing a filter
function apply_filters($hook, $value, ...$args) {
    global $wp_filter;
    if (!isset($wp_filter[$hook])) return $value;
    
    ksort($wp_filter[$hook]); // Sort by priority
    
    foreach ($wp_filter[$hook] as $priority => $callbacks) {
        foreach ($callbacks as $callback) {
            $value = call_user_func_array(
                $callback['function'],
                array_merge([$value], $args)
            );
        }
    }
    return $value;
}

// Executing an action (same as filter, no return)
function do_action($hook, ...$args) {
    global $wp_filter;
    if (!isset($wp_filter[$hook])) return;
    
    ksort($wp_filter[$hook]);
    
    foreach ($wp_filter[$hook] as $callbacks) {
        foreach ($callbacks as $callback) {
            call_user_func_array($callback['function'], $args);
        }
    }
}
?>

================================================================================
REMEMBER
================================================================================

1. Actions and Filters use THE SAME underlying system
2. add_action() is just an alias for add_filter()
3. ALL hooks stored in global $wp_filter array
4. Priority determines execution order (lower = earlier)
5. Filters MUST return a value
6. Actions execute code without returning

================================================================================
USEFUL DEBUGGING
================================================================================

// See all registered hooks
var_dump($GLOBALS['wp_filter']);

// Check specific hook
var_dump($GLOBALS['wp_filter']['the_title']);

// Get current filter name
$current = current_filter();

// Count callbacks on a hook
$count = count($GLOBALS['wp_filter']['init'][10]);

================================================================================
END OF GUIDE
================================================================================

Keep this file handy for quick reference!
Star this repo if it helped you understand WordPress hooks!

Repository: [Your GitHub URL Here]
License: MIT
Contributions: Welcome!

================================================================================
