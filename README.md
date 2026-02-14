# WORDPRESS HOOK SYSTEM - COMPLETE GUIDE

A clear reference for understanding WordPress hooks.
Perfect for GitHub and quick reference.


TABLE OF CONTENTS

1. What Are Hooks?
2. Actions vs Filters
3. How It Works
4. Function Reference
5. Execution Flow
6. Priority System
7. Code Examples
8. Best Practices
9. Quick Cheat Sheet


---


1. WHAT ARE HOOKS?

Hooks let you inject custom code into WordPress at specific points.

Why use hooks?
• Modify WordPress behavior without editing core files
• Extend plugins and themes safely
• Run code at the right time in WordPress lifecycle


---


2. ACTIONS vs FILTERS

ACTIONS - Execute Code

• Purpose: DO something (send email, save data, output HTML)
• Return: Nothing
• Register: add_action()
• Execute: do_action()
• Examples: wp_head, wp_footer, init, save_post

FILTERS - Modify Data

• Purpose: CHANGE something (modify text, alter arrays)
• Return: Modified value (required!)
• Register: add_filter()
• Execute: apply_filters()
• Examples: the_title, the_content, excerpt_length

Quick Rule

If it RETURNS data → Filter
If it DOES something → Action


---


3. HOW IT WORKS

Step 1: REGISTER (add hook)

add_action('hook_name', 'my_function');
add_filter('hook_name', 'my_function');

WordPress stores this in global $wp_filter array.

Step 2: EXECUTE (run hook)

do_action('hook_name');
apply_filters('hook_name', $value);

WordPress runs all registered callbacks in priority order.

Global Storage

$wp_filter = array(
    'hook_name' => array(
        10 => array('function1'),
        20 => array('function2')
    )
);


---


4. FUNCTION REFERENCE

REGISTRATION

add_filter($hook, $callback, $priority, $accepted_args)
add_action($hook, $callback, $priority, $accepted_args)

Parameters:
$hook           - Hook name (string)
$callback       - Your function name
$priority       - Execution order (default: 10)
$accepted_args  - Number of arguments (default: 1)

EXECUTION

apply_filters($hook, $value, ...$args)  - Returns modified value
do_action($hook, ...$args)              - Returns nothing

REMOVAL

remove_filter($hook, $callback, $priority)
remove_action($hook, $callback, $priority)

CHECKING

has_filter($hook, $callback)
has_action($hook, $callback)


---


5. EXECUTION FLOW

FILTER FLOW

1. apply_filters('my_hook', 'hello') is called
2. WordPress finds callbacks in $wp_filter['my_hook']
3. Sorts by priority (10, 20, 30...)
4. Runs each callback:
   - callback_1('hello') → 'HELLO'
   - callback_2('HELLO') → 'HELLO WORLD'
5. Returns final value: 'HELLO WORLD'

ACTION FLOW

1. do_action('my_hook', $arg1, $arg2) is called
2. WordPress finds callbacks in $wp_filter['my_hook']
3. Sorts by priority
4. Runs each callback with arguments
5. Done (no return value)


---


6. PRIORITY SYSTEM

Priority = Execution Order

Lower Number = Runs First

1   ← Highest priority (runs first)
5
10  ← DEFAULT
15
20
100 ← Lowest priority (runs last)

Example

add_filter('the_title', 'uppercase', 5);   // Runs FIRST
add_filter('the_title', 'add_prefix', 10); // Runs SECOND
add_filter('the_title', 'add_suffix', 20); // Runs THIRD


---


7. CODE EXAMPLES

Example 1: Simple Filter

add_filter('the_title', 'make_uppercase');

function make_uppercase($title) {
    return strtoupper($title);
}

// Usage in WordPress:
$title = apply_filters('the_title', 'hello');
// Result: "HELLO"


Example 2: Filter with Multiple Callbacks

add_filter('the_title', 'make_uppercase', 10);
add_filter('the_title', 'add_emoji', 20);

function make_uppercase($title) {
    return strtoupper($title);
}

function add_emoji($title) {
    return '🎉 ' . $title;
}

// Execution:
$result = apply_filters('the_title', 'hello');
// Step 1: 'hello' → make_uppercase → 'HELLO'
// Step 2: 'HELLO' → add_emoji → '🎉 HELLO'
// Final: '🎉 HELLO'


Example 3: Simple Action

add_action('wp_footer', 'add_copyright');

function add_copyright() {
    echo '<p>© 2024 My Website</p>';
}

// WordPress runs this automatically in footer


Example 4: Action with Arguments

add_action('save_post', 'log_post_save', 10, 2);

function log_post_save($post_id, $post) {
    error_log("Post {$post_id} was saved");
}

// WordPress calls: do_action('save_post', $post_id, $post);


Example 5: Multiple Arguments in Filter

add_filter('custom_price', 'apply_discount', 10, 3);

function apply_discount($price, $user_id, $product_id) {
    if ($user_id === 123) {
        return $price * 0.9; // 10% off
    }
    return $price;
}

// Usage:
$final = apply_filters('custom_price', 100, 123, 456);
// Result: 90


Example 6: Class Method as Callback

class MyPlugin {
    public function __construct() {
        add_action('init', array($this, 'initialize'));
    }
    
    public function initialize() {
        // Setup code here
    }
}

new MyPlugin();


Example 7: Anonymous Function

add_filter('the_title', function($title) {
    return strtoupper($title);
}, 10, 1);


Example 8: Removing Hooks

// Remove specific callback
remove_action('wp_footer', 'unwanted_function', 10);

// Must match exact priority used when adding
remove_filter('the_title', 'unwanted_filter', 20);


---


8. BEST PRACTICES

1. Always Return in Filters

✓ Good:
function my_filter($value) {
    return strtoupper($value);
}

✗ Bad:
function my_filter($value) {
    echo strtoupper($value); // DON'T echo in filters!
}


2. Use Unique Function Names

✓ Good: mytheme_custom_title()
✗ Bad: custom_title() ← might conflict


3. Specify Arguments Count

If your function uses multiple arguments:

add_filter('my_hook', 'my_func', 10, 3);

function my_func($arg1, $arg2, $arg3) {
    return $arg1;
}


4. Use Appropriate Priority

• Use 10 (default) for most cases
• Use 5-9 to run early
• Use 11-20 to run late
• Use 1 only if absolutely must run first


5. Remove Hooks Carefully

// Must match exact parameters
remove_action('init', 'my_func', 10);


6. Check Before Removing

if (has_action('init', 'unwanted_func')) {
    remove_action('init', 'unwanted_func');
}


---


9. QUICK CHEAT SHEET

Common Functions

add_filter()       - Register filter
add_action()       - Register action
apply_filters()    - Run filter, return value
do_action()        - Run action, no return
remove_filter()    - Unregister filter
remove_action()    - Unregister action
has_filter()       - Check if filter exists
has_action()       - Check if action exists


Common Patterns

// Basic filter
add_filter('hook', 'function_name', 10, 1);

// Basic action
add_action('hook', 'function_name', 10);

// Multiple arguments
add_filter('hook', 'function_name', 10, 3);

// Class method
add_action('hook', array($this, 'method'));

// Anonymous function
add_filter('hook', function($val) { return $val; });

// Remove hook
remove_action('hook', 'function_name', 10);


Common WordPress Hooks

ACTIONS:
init              - After WordPress loads
wp_head           - In <head> section
wp_footer         - Before </body>
admin_init        - Admin area initialization
save_post         - When post is saved
wp_enqueue_scripts - Load CSS/JS

FILTERS:
the_title         - Post/page title
the_content       - Post content
the_excerpt       - Post excerpt
body_class        - <body> CSS classes
excerpt_length    - Excerpt word count


Debugging Hooks

// See all registered hooks
var_dump($GLOBALS['wp_filter']);

// Check specific hook
var_dump($GLOBALS['wp_filter']['the_title']);

// Get current hook name
$current = current_filter();


---


CORE IMPLEMENTATION (SIMPLIFIED)

How WordPress Actually Does It

<?php
// Global storage
global $wp_filter;

// Adding hooks
function add_filter($hook, $callback, $priority = 10, $args = 1) {
    global $wp_filter;
    $wp_filter[$hook][$priority][] = array(
        'function' => $callback,
        'accepted_args' => $args
    );
}

// add_action is just an alias
function add_action($hook, $callback, $priority = 10, $args = 1) {
    return add_filter($hook, $callback, $priority, $args);
}

// Running filters
function apply_filters($hook, $value, ...$args) {
    global $wp_filter;
    
    if (!isset($wp_filter[$hook])) {
        return $value;
    }
    
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

// Running actions (same as filters, but no return)
function do_action($hook, ...$args) {
    global $wp_filter;
    
    if (!isset($wp_filter[$hook])) {
        return;
    }
    
    ksort($wp_filter[$hook]);
    
    foreach ($wp_filter[$hook] as $callbacks) {
        foreach ($callbacks as $callback) {
            call_user_func_array($callback['function'], $args);
        }
    }
}
?>


---


KEY TAKEAWAYS

1. Actions DO things, Filters CHANGE things
2. add_action() is just add_filter() with a different name
3. All hooks stored in $wp_filter global array
4. Lower priority number = runs earlier
5. Filters MUST return a value
6. Actions don't return anything
7. Specify accepted_args when using multiple parameters


---


REMEMBER

Basic Syntax

add_filter('hook_name', 'my_function', 10, 1);
add_action('hook_name', 'my_function', 10, 1);

Filter Function

function my_function($value) {
    // Modify $value
    return $value; // MUST return!
}

Action Function

function my_function() {
    // Do something
    // No return needed
}


---


END OF GUIDE

Save this for quick reference!
Keep learning and building awesome WordPress sites!
