Duktape Programmer's Guide
Introduction §

Version: 2.1.0 (2017-04-15)

Document scope §

This guide provides an introduction to using Duktape in your programs. Once you're familiar with the basics, there is a concise API reference for looking up API details. The Duktape Wiki provides more detailed examples and best practices.

This document doesn't cover Duktape internals, see the Duktape repo if you wish to tinker with them.

What is Duktape? §

Duktape is an embeddable Ecmascript engine with a focus on portability and compact footprint. By integrating Duktape into your C/C++ program you can easily extend its functionality through scripting. You can also build the main control flow of your program in Ecmascript and use fast C code functions to do heavy lifting.

Embeddability means that Duktape makes minimal assumptions about the underlying platform and its capabilities; instead, the application which embeds Duktape is in control of which native bindings are provided and in what manner. For example, there are no default bindings for printing text to the console or to interact with the file system. Duktape distributable package includes example providers which you can easily integrate if they suit your needs.

The terms Ecmascript and Javascript are often considered more or less equivalent, although Javascript and its variants are technically just one environment where the Ecmascript language is used. The line between the two is not very clear in practice: even non-browser Ecmascript environments often provide some browser-specific built-ins. Even so, we use the term Ecmascript throughout to refer to the language implemented by Duktape.

Conformance §

Duktape conforms to ES5.0/ES5.1 with semantics updated from ES2015 or later where appropriate:

ECMAScript Language Specification 5th Edition
ECMAScript® Language Specification 5.1 Edition
Duktape tracks the latest Ecmascript specification for semantics and built-ins (however support for ES2015 and later is still incomplete), see:

ECMAScript® 2015 Language Specification
ECMAScript® 2016 Language Specification
In some specific cases Duktape may follow a specification draft, see work in progress in TC39/ecma262. This is done when a feature in the latest specifications conflicts with real world code (see e.g. RegExp.prototype issues).

TypedArray support is based on ES2016 TypedArray; initial implementation was based on Khronos TypedArray specification:

Typed Array Specification (Editor's Draft 18 July 2013) (Note: link is to latest)
svn co -r 30720 https://cvs.khronos.org/svn/repos/registry/trunk/public/typedarray
Node.js Buffer support is based on:

Buffer Node.js v6.9.1
Features §

Besides standard Ecmascript features, Duktape has the following additional features (some are visible to applications, while others are internal):

ES2015 TypedArray and Node.js Buffer bindings, plain buffer type (lightweight Uint8Array)
From Ecmascript 2015 (ES6): setPrototypeOf/__proto__, a subset of Proxy objects, Reflect, computed property names, and minimal const support
From Ecmascript 2016 (ES7): exponentiation operator (**, **=)
Encoding API bindings based on the WHATWG Encoding Living Standard
Duktape specific built-ins: provided by the Duktape global object
Extended types: custom "buffer" and "pointer" types, extended string type which supports arbitary binary strings and non-BMP strings (standard Ecmascript only supports 16-bit codepoints)
Combined reference counting and mark-and-sweep garbage collection, with finalizers and emergency garbage collection (you can also build with just mark-and-sweep)
Coroutine support
Tail call support
Built-in debugger framework based on a debugger protocol
Built-in regular expression engine with no platform dependencies
Built-in Unicode support with no platform dependencies
Built-in number parsing and formatting with no platform dependencies
Additional custom JSON formats (JX and JC)
Bytecode dump/load mechanism for caching compiled functions
Goals §

Compliance. Ecmascript E5/E5.1 and real world compliance. Ecmascript compliance requires regular expression and Unicode support. When possible, implement features from latest or draft Ecmascript specifications to minimize Duktape custom features.

Portability. Minimal system dependencies are nice when porting, so Duktape depends on very few system libraries. For example, number formatting and parsing, regular expressions, and Unicode are all implemented internally by Duktape. One of the few dependencies that cannot be fully eliminated is system date/time integration in the Date built-in. Duktape supports major platforms directly but you can also use an external Date provider on exotic platforms.

Easy C interface. The interface between Duktape and C programs should be natural and error-tolerant. As a particular issue, string representation should be UTF-8 with automatic NUL terminators to match common C use.

Small footprint. Code and data footprint should be as small as possible, even for small programs. Duktape is portable even to "bare metal" targets with no standard libraries. This is more important than performance, as there are already several very fast engines but fewer very compact, portable engines.

Reasonable performance. Small footprint (and portability, to some extent) probably eliminates the possibility of a competitive JIT-based engine, so there is no practical way of competing with very advanced JIT-based engines like SpiderMonkey (and its optimized variants) or Google V8. Performance should still be reasonable for typical embedded programs. Lua is a good benchmark in this respect. (Adding optional, modular support for JITing or perhaps off-line compilation would be nice.)

ASCII string performance. It's important that operations dealing with plain ASCII strings be very fast: ASCII dominates most embedded use. Operations dealing with non-ASCII strings need to perform reasonably but are not critical. This is a necessary trade-off: using C-compatible strings means essentially using UTF-8 string representation which makes string indexing and many other operations slower than with fixed size character representations. It's still important to support common idioms like iterating strings sequentially (in either direction) efficiently.

Document organization §

Getting started guides you through downloading, compiling, and integrating Duktape into your program. It also provides concrete examples of how you can integrate scripting capabilities into your program.

Programming model, Stack types, and C types discuss core Duktape concepts such as heap, context, value stacks, Duktape API, and Duktape/C functions. Duktape stack types and C type wrappers are discussed in detail.

Duktape specific Ecmascript features are discussed in multiple sections: Type algorithms (for custom types), Duktape built-ins (additional built-ins), Post-ES5 features (features implemented from ES2016 and beyond), Custom behavior (behavior differing from standard), Custom JSON formats, Custom directives, Buffer objects, Error objects (properties and traceback support), Function objects (properties), Date and time, Random numbers, Debugger, Modules, Finalization, Coroutines, Virtual properties, Symbols, Bytecode dump/load, Threading, Sandboxing.

Performance provides a few Duktape-specific tips for improving performance and avoiding performance pitfalls. Memory usage summarizes Duktape memory usage and gives pointers for minimizing it. Compiling describes how to configure and compile Duktape as part of your application. Portability covers platform and compiler specific issues and other portability issues. Compatibility discusses Duktape's compatibility with Ecmascript dialects, extensions, and frameworks. Versioning describes Duktape versioning and what version compatibility to expect. Limitations summarizes currently known limitations and provides possible workarounds.

Comparison to Lua discusses some differences between Lua and Duktape; it may be useful reading if you're already familiar with Lua.

Getting started §

Downloading §

Download the source distributable from the Download page.

Command line tool for testing §

Unpack the distributable:

$ cd /tmp
$ tar xvfJ duktape-<version>.tar.xz
Compile the command line tool using the provided Makefile:

$ cd /tmp/duktape-<version>/
$ make -f Makefile.cmdline
The Makefile assumes you have gcc installed. If you don't, you can just edit the Makefile to match your compiler (the Makefile is quite simple).

Duktape doesn't provide built-in bindings for file or console I/O to avoid portability issues (for example, some platforms don't have I/O APIs at all). The command line utility provides print() and alert() bindings using extras/print-alert to make it easier to play with. There are useful "extras" in the distributable providing useful (optional) bindings such as:
print() and alert()
console object, e.g. console.log()
Throughout the guide examples will assume a print() binding for illustration.
The command line tool avoids platform dependencies by default. You can add line editing support via linenoise by editing the Makefile:
Add -DDUK_CMDLINE_FANCY
Add -Ipath/to/linenoise for the linenoise.h header
Add path/to/linenoise.c to the source list
Linenoise only works in POSIX environments and requires a C compiler (not C++)
You can now run Ecmascript code interactively:

$ ./duk
((o) Duktape 2.1.0 (v2.1.0)
duk> print('Hello world!')
Hello world!
= undefined
You can also run Ecmascript code from a file which is useful for playing with features and algorithms. As an example, create fib.js:

// fib.js
function fib(n) {
    if (n == 0) { return 0; }
    if (n == 1) { return 1; }
    return fib(n-1) + fib(n-2);
}

function test() {
    var res = [];
    for (i = 0; i < 20; i++) {
        res.push(fib(i));
    }
    print(res.join(' '));
}

test();
Test the script from the command line:

$ ./duk fib.js
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181
Integrating Duktape into your program §

The command line tool is simply an example of a program which embeds Duktape. Embedding Duktape into your program is very simple:

Run duktape-N.N.N/tools/configure.py to configure Duktape for build. The result is a directory containing duktape.c, duktape.h, and duk_config.h.
Add duktape.c, duktape.h, and duk_config.h to your build, and call the Duktape API from elsewhere in your program.
The Duktape distributable (duktape-N.N.N.tar.xz) src/ directory contains preconfigured header and source files for the Duktape default configuration which can usually be used as is. If needed, the configuration tool allows you to customize Duktape options, such as optimizing Duktape for low memory targets and enable/disable features. See Compiling and Configuring Duktape for build for more details and examples.

The distributable contains a very simple example program, hello.c, which illustrates this process. Compile the test program with the preconfigured Duktape header and source files e.g. as follows:

$ cd /tmp/duktape-<version>/
$ gcc -std=c99 -o hello -Isrc src/duktape.c examples/hello/hello.c -lm
To customize Duktape configuration use configure.py:

$ cd /tmp/duktape-<version>/
# Here we disable Ecmascript 6 Proxy object support
$ python2 tools/configure.py --output-directory duktape-src -UDUK_USE_ES6_PROXY
$ gcc -std=c99 -o hello -Iduktape-src duktape-src/duktape.c examples/hello/hello.c -lm
The test program creates a Duktape context and uses it to run some Ecmascript code:

$ ./hello
Hello world!
2+3=5
Because Duktape is an embeddable engine, you don't need to change the basic control flow of your program. The basic approach is:

Create a Duktape context e.g. in program initialization (or even on-demand when scripting is needed). Usually you would also load your scripts during initialization, though that can also be done on-demand.
Identify points in your code where you would like to use scripting and insert calls to script functions there.
To make a script function call, first push call arguments to the Duktape context's value stack using the Duktape API. Then, use another Duktape API call to initiate the actual call.
Once script execution is finished, control is returned to your program (the API call returns) and a return value is left on the Duktape context's value stack. C code can then access the return value using the Duktape API.
More broadly, there are several approaches to how you can use Duktape with native code; for example:

Run the main application in C/C++ code but call into Duktape for extending base functionality (e.g. plugins or configuration).
Run the main application in Ecmascript code, and call into simple C/C++ native bindings for I/O, performance intensive operations, etc. The native bindings are often kept stateless so that state logic remains in view of script code.
Run the main application in Ecmascript code, but use more complex, stateful C/C++ native bindings for performance intensive operations. For example, a graphics engine can be implemented as a native object.
See the following Wiki articles for detailed examples:

Getting started: line processing
Getting started: primality testing
Programming model §

Overview §

Programming with Duktape is quite straightforward:

Add Duktape source (duktape.c) and headers (duktape.h and duk_config.h) to your build.
Create a Duktape heap (a garbage collection region) and an initial context (essentially a thread handle) in your program.
Load the necessary Ecmascript script files, and register your Duktape/C functions. Duktape/C functions are C functions you can call from Ecmascript code for better performance, bindings to native libraries, etc.
Use the Duktape API to call Ecmascript functions whenever appropriate. Duktape API is used to pass values to and from functions. Values are converted between their C representation and the Duktape internal (Ecmascript compatible) representation.
Duktape API is also used by Duktape/C functions (called from Ecmascript) to access call arguments and to provide return values.
Let's look at all the steps and their related concepts in more detail.

Heap and context §

A Duktape heap is a single region for garbage collection. A heap is used to allocate storage for strings, Ecmascript objects, and other variable size, garbage collected data. Objects in the heap have an internal heap header which provides the necessary information for reference counting, mark-and-sweep garbage collection, object finalization, etc. Heap objects can reference each other, creating a reachability graph from a garbage collection perspective. For instance, the properties of an Ecmascript object reference both the keys and values of the object's property set. You can have multiple heaps, but objects in different heaps cannot reference each other directly; you need to use serialization to pass values between heaps.

A Duktape context is an Ecmascript "thread of execution" which lives in a certain Duktape heap. A context is represented by a duk_context * in the Duktape API, and is associated with an internal Duktape coroutine (a form of a co-operative thread). Each context is also associated with an environment consisting of global objects; contexts may share the same global environment but can also have different environments. The context handle is given to almost every Duktape API call, and allows the caller to interact with the value stack of the Duktape coroutine: values can be inserted and queries, functions can be called, and so on.

Each coroutine has a call stack which controls execution, keeping track of function calls, native or Ecmascript, within the Ecmascript engine. Each coroutine also has a value stack which stores all the Ecmascript values of the coroutine's active call stack. The value stack always has an active stack frame for the most recent function call (when no function calls have been made, the active stack frame is the value stack as is). The Duktape API calls operate almost exclusively in the currently active stack frame. A coroutine also has an internal catch stack which is used to track error catching sites established using e.g. try-catch-finally blocks. This is not visible to the caller in any way at the moment.

Multiple contexts can share the same Duktape heap. In more concrete terms this means that multiple contexts can share the same garbage collection state, and can exchange object references safely. Contexts in different heaps cannot exchange direct object references; all values must be serialized in one way or another.

Almost every API call provided by the Duktape API takes a context pointer as its first argument: no global variables or states are used, and there are no restrictions on running multiple, independent Duktape heaps and contexts at the same time. There are multi-threading restrictions, however: only one native thread can execute any code within a single heap at any time, see Threading.

To create a Duktape heap and an initial context inside the heap, you can simply use:

duk_context *ctx = duk_create_heap_default();
if (!ctx) { exit(1); }
If you wish to provide your own memory allocation functions and a fatal error handler function (recommended), use:

duk_context *ctx = duk_create_heap(my_alloc,
                                   my_realloc,
                                   my_free,
                                   my_udata,
                                   my_fatal);
if (!ctx) { exit(1); }
To create a new context inside the same heap, with the context sharing the same global objects:

duk_context *new_ctx;

(void) duk_push_thread(ctx);
new_ctx = duk_get_context(ctx, -1 /*index*/);
To create a new context inside the same heap, but with a fresh set of global object:

duk_context *new_ctx;

(void) duk_push_thread_new_globalenv(ctx);
new_ctx = duk_get_context(ctx, -1 /*index*/);
Contexts are automatically garbage collected when they become unreachable. This also means that if your C code holds a duk_context *, the corresponding Duktape coroutine MUST be reachable from a garbage collection point of view.

A heap must be destroyed explicitly when the caller is done with it:

duk_destroy_heap(ctx);
This frees all heap objects allocated, and invalidates any pointers to such objects. In particular, if the calling program holds string pointers to values which resided on the value stack of a context associated with the heap, such pointers are invalidated and must never be dereferenced after the heap destruction call returns.

Call stack and catch stack (of a context) §

The call stack of a context is not directly visible to the caller. It keeps track of the chain of function calls, either C or Ecmascript, currently executing in a context. The main purpose of this book-keeping is to facilitate the passing of arguments and results between function callers and callees, and to keep track of how the value stack is divided between function calls. The call stack also allows Duktape to construct a traceback for errors.

Closely related to the call stack, Duktape also maintains a catch stack for keeping track of current error catching sites established using e.g. try-catch-finally. The catch stack is even less visible to the caller than the call stack.

Because Duktape supports tail calls, the call stack does not always accurately represent the true call chain: tail calls will be "squashed" together in the call stack.

Don't confuse with the C stack.
Value stack (of a context) and value stack index §

The value stack of a context is an array of tagged type values related to the current execution state of a coroutine. The tagged types used are: undefined, null, boolean, number, string, object, buffer, pointer, and lightfunc. For a detailed discussion of the available tagged types, see Types.

The value stack is divided between the currently active function calls (activations) on the coroutine's call stack. At any time, there is an active stack frame which provides an origin for indexing elements on the stack. More concretely, at any time there is a bottom which is referred to with the index zero in the Duktape API. There is also a conceptual top which identifies the stack element right above the highest currently used element. The following diagram illustrates this:

 Value stack
 of 15 entries
 (absolute indices)

.----.
| 15 |
| 14 |
| 13 |
| 12 |      Active stack frame (indices
| 11 |      relative to stack bottom)
| 10 |
|  9 |      .---.
|  8 |      | 5 |   API index 0 is bottom (at value stack index 3).
|  7 |      | 4 |
|  6 |      | 3 |   API index 5 is highest used (at value stack index 8).
|  5 |      | 2 |   
|  4 |      | 1 |   Stack top is 6 (relative to stack bottom).
|  3 | <--- | 0 |
|  2 |      `---'
|  1 |
|  0 |
`----'
There is no direct way to refer to elements in the internal value stack: Duktape API always deals with the currently active stack frame. Stack frames are shown horizontally throughout the documentation for space reasons. For example, the active stack frame in the figure above would be shown as:

0 1 2 3 4 5  
A value stack index is a signed integer index used in the Duktape API to refer to elements in currently active stack frame, relative to the current frame bottom.

Non-negative (>= 0) indices refer to stack entries in the current stack frame, relative to the frame bottom:

0 1 2 3 4 5  
Negative (< 0) indices refer to stack entries relative to the top:

-6 -5 -4 -3 -2 -1  
The special constant DUK_INVALID_INDEX is a negative integer which denotes an invalid stack index. It can be returned from API calls and can also be given to API calls to indicate a "no value".

The value stack top (or just "top") is the non-negative index of an imaginary element just above the highest used index. For instance, above the highest used index is 5, so the stack top is 6. The top indicates the current stack size, and is also the index of the next element pushed to the stack.

0 1 2 3 4 5 6  
API stack operations are always confined to the current stack frame. There is no way to refer to stack entries below the current frame. This is intentional, as it protects functions in the call stack from affecting each other's values.

Don't confuse with the C stack.
Growing the value stack §

At any time, the value stack of a context is allocated for a certain maximum number of entries. An attempt to push values beyond the allocated size will cause an error to be thrown, it will not cause the value stack to be automatically extended. This simplifies the internal implementation and also improves performance by minimizing reallocations when you know, beforehand, that a certain number of entries will be needed during a function.

When a value stack is created or a Duktape/C function is entered, the value stack is always guaranteed to have space for the call arguments and DUK_API_ENTRY_STACK (currently 64) elements. In the typical case this is more than sufficient so that the majority of Duktape/C functions don't need to extend the value stack. Only functions that need more space or perhaps need an input-dependent amount of space need to grow the value stack.

You can extend the stack allocation explicitly with duk_check_stack() or (usually more preferably) duk_require_stack(). Once successfully extended, you are again guaranteed that the specified number of elements can be pushed to the stack. There is no way to shrink the allocation except by returning from a Duktape/C function.

Consider, for instance, the following function which will uppercase an input ASCII string by pushing uppercased characters one-by-one on the stack and then concatenating the result. This example illustrates how the number of value stack entries required may depend on the input (otherwise this is not a very good approach for uppercasing a string):

/* uppercase.c */
#include <stdio.h>
#include <stdlib.h>
#include "duktape.h"

static int dummy_upper_case(duk_context *ctx) {
    size_t sz;
    const char *val = duk_require_lstring(ctx, 0, &sz);
    size_t i;

    /* We're going to need 'sz' additional entries on the stack. */
    duk_require_stack(ctx, sz);

    for (i = 0; i < sz; i++) {
        char ch = val[i];
        if (ch >= 'a' && ch <= 'z') {
            ch = ch - 'a' + 'A';
        }
        duk_push_lstring(ctx, (const char *) &ch, 1);
    }

    duk_concat(ctx, sz);
    return 1;
}

int main(int argc, char *argv[]) {
    duk_context *ctx;

    if (argc < 2) { exit(1); }

    ctx = duk_create_heap_default();
    if (!ctx) { exit(1); }

    duk_push_c_function(ctx, dummy_upper_case, 1);
    duk_push_string(ctx, argv[1]);
    duk_call(ctx, 1);
    printf("%s -> %s\n", argv[1], duk_to_string(ctx, -1));
    duk_pop(ctx);

    duk_destroy_heap(ctx);
    return 0;
}
In addition to user reserved elements, Duktape keeps an automatic internal value stack reserve to ensure all API calls have enough value stack space to work without further allocations. The value stack is also extended and shrunk in somewhat large steps to minimize memory reallocation activity. As a result the internal number of value stack elements available beyond the caller specified extra varies considerably. The caller does not need to take this into account and should never rely on any additional elements being available.

Ecmascript array index §

Ecmascript object and array keys can only be strings. Array indices (e.g. 0, 1, 2) are represented as canonical string representations of the respective numbers. More technically, all canonical string representations of the integers in the range [0, 2**32-1] are valid array indices.

To illustrate the Ecmascript array index handling, consider the following example:

var arr = [ 'foo', 'bar', 'quux' ];

print(arr[1]);     // refers to 'bar'
print(arr["1"]);   // refers to 'bar'

print(arr[1.0]);   // refers to 'bar', canonical encoding is "1"
print(arr["1.0"]); // undefined, not an array index
Some API calls operating on Ecmascript arrays accept numeric array index arguments. This is really just a short hand for denoting a string conversion of that number. For instance, if the API is given the integer 123, this really refers to the property name "123".

Internally, Duktape tries to avoid converting numeric indices to actual strings whenever possible, so it is preferable to use array index API calls when they are relevant. Similarly, when writing Ecmascript code it is preferable to use numeric rather than string indices, as the same fast path applies for Ecmascript code.

Duktape API §

Duktape API is the collection of user callable API calls defined in duktape.h and documented in the API reference.

The Duktape API calls are generally error tolerant and will check all arguments for errors (such as NULL pointers). However, to minimize footprint, the ctx argument is not checked, and the caller MUST NOT call any Duktape API calls with a NULL context.

All Duktape API calls are potentially macros. Calling code must not rely on any Duktape API call being available as a function pointer. The implementation of a certain API call may change between a macro and an actual function even between compatible releases. The Duktape API provides the following guarantees for macros:

Arguments are never evaluated more than once (unless explicitly mentioned). However, an argument may not be evaluated at all if an argument is ignored in the current version.
An API call with a return value can be used as an expression. If the API macro contains multiple statements, it is implemented as a comma expression (e.g. (foo, bar, quux)).
An API call with a void return value may not necessarily work as part of an expression. The API macro may be implemented as a block statement or as a dummy do {...} while (0) loop.
Duktape/C function §

A C function with a Duktape/C API signature can be associated with an Ecmascript function object, and gets called when the Ecmascript function object is called. A Duktape/C API function looks as follows:

duk_ret_t my_func(duk_context *ctx) {
    duk_push_int(ctx, 123);
    return 1;
}
The function gets Ecmascript call argument in the value stack of ctx, with duk_get_top() indicating the number of arguments present on the value stack. The this binding is not automatically pushed to the value stack; use duk_push_this() to access it. When creating an Ecmascript function object associated with a Duktape/C function, one can select the desired number of arguments. Extra arguments are dropped and missing arguments are replaced with undefined. A function can also be registered as a vararg function (by giving DUK_VARARGS as the argument count) in which case call arguments are not modified prior to C function entry.

The function can return one of the following:

Return value 1 indicates that the value on the stack top is to be interpreted as a return value.
Return value 0 indicates that there is no explicit return value on the value stack; an undefined is returned to caller.
A negative return value indicates that an error is to be automatically thrown. Error codes named DUK_RET_xxx map to specific kinds of errors (do not confuse these with DUK_ERR_xxx which are positive values).
A return value higher than 1 is currently undefined, as Ecmascript doesn't support multiple return values in Edition 5.1. (Values higher than 1 may be taken into to support multiple return values in Ecmascript Edition 6.)
A negative error return value is intended to simplify common error handling, and is an alternative to constructing and throwing an error explicitly with Duktape API calls. No error message can be given; a message is automatically constructed by Duktape. For example:

duk_ret_t my_func(duk_context *ctx) {
    if (duk_get_top(ctx) == 0) {
        /* throw TypeError if no arguments given */
        return DUK_RET_TYPE_ERROR;
    }
    /* ... */
}
All Duktape/C functions are considered strict in the Ecmascript sense. Duktape API calls always obey Ecmascript strict mode semantics, even when the API calls are made outside of any Duktape/C function, i.e. with an empty call stack. For instance, attempt to delete a non-configurable property using duk_del_prop() will cause an error to be thrown. This is the case with a strict Ecmascript function too:

function f() {
    'use strict';
    var arr = [1, 2, 3];
    return delete arr.length;  // array 'length' is non-configurable
}

print(f());  // this throws an error because f() is strict
Another consequence of Duktape/C function strictness is that the this binding given to Duktape/C functions is not coerced. This is also the case for strict Ecmascript code:

function strictFunc() { 'use strict'; print(typeof this); }
function nonStrictFunc() { print(typeof this); }

strictFunc.call('foo');     // prints 'string' (uncoerced)
nonStrictFunc.call('foo');  // prints 'object' (coerced)
Duktape/C functions are currently always constructable, i.e. they can always be used in new Foo() expressions. You can check whether a function was called in constructor mode as follows:

static duk_ret_t my_func(duk_context *ctx) {
    if (duk_is_constructor_call(ctx)) {
        printf("called as a constructor\n");
    } else {
        printf("called as a function\n");
    }
}
To save memory, Duktape/C functions don't have a prototype property by default, so the default object instance (given to the constructor as this) inherits from Object.prototype. To use a custom prototype you can define prototype for the Duktape/C function explicitly. Another approach is to ignore the default object instance and construct one manually within the Duktape/C call: as with Ecmascript functions, if a constructor returns an object value, that value replaces the default object instance and becomes the value of the new expression.

The this binding is not automatically pushed to the value stack; use duk_push_this() to access it.
Storing state for a Duktape/C function §

Sometimes it would be nice to provide parameters or additional state to a Duktape/C function out-of-band, i.e. outside explicit call arguments. There are several ways to achieve this.

Properties of function §
First, a Duktape/C function can use its Function object to store state or parameters. A certain Duktape/C function (the actual C function) is always represented by an Ecmascript Function object which is internally associated with the underlying C function. The Function object can be used to store properties related to that particular instance of the function. Note that a certain Duktape/C function can be associated with multiple independent Function objects and thus independent states.

Accessing the Ecmascript Function object related to a Duktape/C function is easy:

duk_push_current_function(ctx);
duk_get_prop_string(ctx, -1, "my_state_variable");
'this' binding §
Another alternative for storing state is to call the Duktape/C function as a method and then use the this binding for storing state. For instance, consider a Duktape/C function called as:

foo.my_c_func()
When called, the Duktape/C function gets foo as its this binding, and one could store state directly in foo. The difference to using the Function object approach is that the same object is shared by all methods, which has both advantages and disadvantages.

Accessing the this binding is easy:

duk_push_this(ctx);
duk_get_prop_string(ctx, -1, "my_state_variable");
Magic value of function §
Duktape/C function objects can store an internal 16-bit signed integer "magic" value (zero by default) with no extra memory cost. The magic value can be used to pass flags and/or small values to a Duktape/C function at minimal cost, so that a single native function can provide slightly varied behavior for multiple function objects:

/* Magic value example: two lowest bits are used for a prefix index, bit 2 (0x04)
 * is used to select newline style for a log write helper.
 */
const char *prefix[4] = { "INFO", "WARN", "ERROR", "FATAL" };
duk_int_t magic = duk_get_current_magic(ctx);

printf("%s: %s", prefix[magic & 0x03], duk_safe_to_string(ctx, 0));
if (magic & 0x04) {
    printf("\r\n");
} else {
    printf("\n");
}
For an API usage example, see the test case test-get-set-magic.c. Duktape uses magic values a lot internally to minimize size of compiled code, see e.g. duk_bi_math.c.

The magic value mechanism is liable to change between major Duktape versions, as the number of available spare bits changes. Use magic values only when it really matters for footprint. Properties stored on the function object is a more stable alternative.
Heap stash §
The heap stash is an object visible only from C code. It is associated with the Duktape heap, and allows Duktape/C code to store "under the hood" state data which is not exposed to Ecmascript code. It is accessed with the duk_push_heap_stash() API call.

Global stash §
The global stash is like the heap stash, but is associated with a global object. It is accessed with the duk_push_global_stash() API call. There can be several environments with different global objects within the same heap.

Thread stash §
The thread stash is like the heap stash, but is associated with a Duktape thread (i.e. a ctx pointer). It is accessible with the duk_push_thread_stash() API call.

Duktape version specific code §

The Duktape version is available through the DUK_VERSION define, with the numeric value (major * 10000) + (minor * 100) + patch. The same value is available to Ecmascript code through Duktape.version. Calling code can use this define for Duktape version specific code.

For C code:

#if (DUK_VERSION >= 10203)
/* Duktape 1.2.3 or later */
#elif (DUK_VERSION >= 800)
/* Duktape 0.8.0 or later */
#else
/* Duktape lower than 0.8.0 */
#endif
For Ecmascript code (also see Duktape built-ins):

if (typeof Duktape !== 'object') {
    print('not Duktape');
} else if (Duktape.version >= 10203) {
    print('Duktape 1.2.3 or higher');
} else if (Duktape.version >= 800) {
    print('Duktape 0.8.0 or higher (but lower than 1.2.3)');
} else {
    print('Duktape lower than 0.8.0');
}
Numeric error codes §

When errors are created or thrown using the Duktape API, the caller must assign a numeric error code to the error. Error codes are positive integers, with a range restricted to 24 bits at the moment: the allowed error number range is thus [1,16777215]. Built-in error codes are defined in duktape.h, e.g. DUK_ERR_TYPE_ERROR.

The remaining high bits are used internally to carry e.g. additional flags. Negative error values are used in the Duktape/C API as a shorthand to automatically throw an error.

Error handling §

Error handling in the Duktape API is similar to how Ecmascript handles errors: errors are thrown either explicitly or implicitly, then caught and handled. However, instead of a try-catch statement application code uses protected Duktape API calls to establish points in C code where errors can be caught and handled. An uncaught error causes the fatal error handler to be called, which is considered an unrecoverable situation and should ordinarily be avoided, see Normal and fatal errors and How to handle fatal errors.

To avoid fatal errors, typical application code should establish an error catch point before making other Duktape API calls. This is done using protected Duktape API calls, for example:

Use protected calls to evaluate code (duk_peval()), compile code (duk_pcompile()), and call (duk_pcall()) functions.
Use a single duk_safe_call() to establish an error catcher so that you can use unsafe primitives freely inside the safe call.
An example of the first technique:

/* Use duk_peval() variant to evaluate a file so that script errors are
 * handled safely.  Both syntax errors and runtime errors are caught.
 */

push_file_as_string(ctx, "myscript.js");
if (duk_peval(ctx) != 0) {
    /* Use duk_safe_to_string() to convert error into string.  This API
     * call is guaranteed not to throw an error during the coercion.
     */
    printf("Script error: %s\n", duk_safe_to_string(ctx, -1));
}
duk_pop(ctx);
An example of the second technique:

/* Use duk_safe_call() to wrap all unsafe code into a separate C function.
 * This approach has the advantage of covering all API calls automatically
 * but is a bit more verbose.
 */

static duk_ret_t unsafe_code(duk_context *ctx, void *udata) {
    /* Here we can use unprotected calls freely. */

    (void) udata;  /* 'udata' may be used to pass e.g. a struct pointer */

    push_file_as_string(ctx, "myscript.js");
    duk_eval(ctx);

    /* ... */

    return 0;  /* success return, no return value */
}

/* elsewhere: */

if (duk_safe_call(ctx, unsafe_code, NULL /*udata*/, 0 /*nargs*/, 1 /*nrets */) != 0) {
    /* The 'nrets' argument should be at least 1 so that an error value
     * is left on the stack if an error occurs.  To avoid further errors,
     * use duk_safe_to_string() for safe error printing.
     */
    printf("Unexpected error: %s\n", duk_safe_to_string(ctx, -1));
}
duk_pop(ctx);
Even within protected calls there are some rare cases, such as internal errors, that will either cause a fatal error or propagate an error outwards from a protected API call. These should only happen in abnormal conditions and are not considered recoverable. To handle also these cases well, a production quality application should always have a fatal error handler with a reasonable strategy for dealing with fatal errors. Such a strategy is necessarily context dependent, but could be something like:

On an embedded device the fatal error handler could write the fatal error information to a flash file and reboot the device. After the reboot, the fatal error could be reported to a diagnostics server so that it can be investigated.
On a UNIX system the fatal error handler could simply exit the process (the default fatal handler uses abort()) and let a wrapper script restart the application.
Note that it may be fine for some applications to make API calls without an error catcher and risk throwing uncaught errors leading to a fatal error. It's not possible to continue execution after a fatal error, so such applications would typically simply exit when a fatal error occurs. Even without an actual recovery strategy, a fatal error handler should be used to e.g. write fatal error information to stderr before process exit.

Normal and fatal errors §

An ordinary error is caused by a throw statement, a duk_throw() API call (or similar), or by an internal, recoverable Duktape error. Ordinary errors can be caught with a try-catch in Ecmascript code or e.g. duk_pcall() (see API calls tagged protected) in C code.

A fatal error is caused by an uncaught error, an explicit call to duk_fatal(), or an unrecoverable error inside Duktape. Each Duktape heap has a heap-wide fatal error handler registered in duk_create_heap(); if no handler is given a built-in default handler is used. There's no safe way to resume execution after a fatal error, so that a fatal error handler must not return or call into the Duktape API. Instead the handler should e.g. write out the message to the console or a log file and then exit the process (or restart an embedded device).

Fatal errors may also happen without any heap context, so that Duktape can't look up a possible heap-specific fatal error handler. Duktape will then always call the built-in default fatal error handler (with the handler userdata argument set to NULL). Fatal errors handled this way are currently limited to assertion failures, so that if you don't enable assertions, no such errors will currently occur. All fatal error handling goes through the heap-associated fatal error handler which is in direct application control.

The built-in default fatal error handler will write a debug log message (but won't write anything to stdout to stderr), and will then call abort(); if that fails, it enters an infinite loop to ensure execution doesn't resume after a fatal error. It is strongly recommended to both (1) provide a custom fatal error handler in heap creation and (2) replace the built-in default fatal error handler using the DUK_USE_FATAL_HANDLER() option in duk_config.h.
See How to handle fatal errors for more detail and examples.

Stack types §

Overview §

Type	Type constant	Type mask constant	Description
(none)	DUK_TYPE_NONE	DUK_TYPE_MASK_NONE	no type (missing value, invalid index, etc)
undefined	DUK_TYPE_UNDEFINED	DUK_TYPE_MASK_UNDEFINED	undefined
null	DUK_TYPE_NULL	DUK_TYPE_MASK_NULL	null
boolean	DUK_TYPE_BOOLEAN	DUK_TYPE_MASK_BOOLEAN	true and false
number	DUK_TYPE_NUMBER	DUK_TYPE_MASK_NUMBER	IEEE double
string	DUK_TYPE_STRING	DUK_TYPE_MASK_STRING	immutable (plain) string or (plain) Symbol
object	DUK_TYPE_OBJECT	DUK_TYPE_MASK_OBJECT	object with properties
buffer	DUK_TYPE_BUFFER	DUK_TYPE_MASK_BUFFER	mutable (plain) byte buffer, fixed/dynamic/external; mimics an Uint8Array
pointer	DUK_TYPE_POINTER	DUK_TYPE_MASK_POINTER	opaque pointer (void *)
lightfunc	DUK_TYPE_LIGHTFUNC	DUK_TYPE_MASK_LIGHTFUNC	plain Duktape/C function pointer (non-object); mimics a Function
Memory allocations §

The following stack types involve additional heap allocations:

String: a single allocation contains a combined heap and string header, followed by the immutable string data.
Object: one allocation is used for a combined heap and object header, and another allocation is used for object properties. The property allocation contains both array entries and normal properties, and if the object is large enough, a hash table to speed up lookups.
Buffer: for fixed buffers a single allocation contains a combined heap and buffer header, followed by the mutable fixed-size buffer. For dynamic buffers the current buffer is allocated separately. For external buffers a single heap object is allocated and points to a user buffer.
Note that while strings and buffers are considered a primitive (pass-by-value) types, they are a heap allocated type from a memory allocation viewpoint.

Pointer stability §

Heap objects allocated by Duktape have stable pointers: the objects are not relocated in memory while they are reachable from a garbage collection point of view. This is the case for the main heap object, but not necessarily for any additional allocations related to the object, such as dynamic property tables or dynamic buffer data area. A heap object is reachable e.g. when it resides on the value stack of a reachable thread or is reachable through the global object. Once a heap object becomes unreachable any pointers held by user C code referring to the object are unsafe and should no longer be dereferenced.

In practice the only heap allocated data directly referenced by user code are strings, fixed buffers, and dynamic buffers. The data area of strings and fixed buffers is stable; it is safe to keep a C pointer referring to the data even after a Duktape/C function returns as long the string or fixed buffer remains reachable from a garbage collection point of view at all times. Note that this is not the case for Duktape/C value stack arguments, for instance, unless specific arrangements are made.

The data area of a dynamic buffer does not have a stable pointer. The buffer itself has a heap header with a stable address but the current buffer is allocated separately and potentially relocated when the buffer is resized. It is thus unsafe to hold a pointer to a dynamic buffer's data area across a buffer resize, and it's probably best not to hold a pointer after a Duktape/C function returns (as there would be no easy way of being sure that the buffer hadn't been resized). The data area of an external buffer also has a potentially changing pointer, but the pointer is only changed by an explicit user API call.

For external buffers the stability of the data pointer is up to the user code which sets and updates the pointer.

Type masks §

Type masks allows calling code to easily check whether a type belongs to a certain type set. For instance, to check that a certain stack value is a number, string, or an object:

if (duk_get_type_mask(ctx, -3) & (DUK_TYPE_MASK_NUMBER |
                                  DUK_TYPE_MASK_STRING |
                                  DUK_TYPE_MASK_OBJECT)) {
    printf("type is number, string, or object\n");
}
There is a specific API call for matching a set of types even more conveniently:

if (duk_check_type_mask(ctx, -3, DUK_TYPE_MASK_NUMBER |
                                 DUK_TYPE_MASK_STRING |
                                 DUK_TYPE_MASK_OBJECT)) {
    printf("type is number, string, or object\n");
}
These are faster and more compact than the alternatives:

// alt 1
if (duk_is_number(ctx, -3) || duk_is_string(ctx, -3) || duk_is_object(ctx, -3)) {
    printf("type is number, string, or object\n");
}

// alt 2
int t = duk_get_type(ctx, -3);
if (t == DUK_TYPE_NUMBER || t == DUK_TYPE_STRING || t == DUK_TYPE_OBJECT) {
    printf("type is number, string, or object\n");
}
None §

The none type is not actually a type but is used in the API to indicate that a value does not exist, a stack index is invalid, etc.

Undefined §

The undefined type maps to Ecmascript undefined, which is distinguished from a null.

Values read from outside the active value stack range read back as undefined.

Null §

The null type maps to Ecmascript null.

Boolean §

The boolean type is represented in the C API as an integer: zero for false, and non-zero for true.

Whenever giving boolean values as arguments in API calls, any non-zero value is accepted as a "true" value. Whenever API calls return boolean values, the value 1 is always used for a "true" value. This allows certain C idioms to be used. For instance, a bitmask can be built directly based on API call return values, as follows:

/* this works and generates nice code */
int bitmask = (duk_get_boolean(ctx, -3) << 2) |
              (duk_get_boolean(ctx, -2) << 1) |
              duk_get_boolean(ctx, -1);

/* more verbose variant not relying on "true" being represented by 1 */
int bitmask = ((duk_get_boolean(ctx, -3) ? 1 : 0) << 2) |
              ((duk_get_boolean(ctx, -2) ? 1 : 0) << 1) |
              (duk_get_boolean(ctx, -1) ? 1 : 0);

/* another verbose variant */
int bitmask = (duk_get_boolean(ctx, -3) ? (1 << 2) : 0) |
              (duk_get_boolean(ctx, -2) ? (1 << 1) : 0) |
              (duk_get_boolean(ctx, -1) ? 1 : 0);
Number §

The number type is an IEEE double, including +/- Infinity and NaN values. Zero sign is also preserved. An IEEE double represents all integers up to 53 bits accurately.

IEEE double allows NaN values to have additional signaling bits. Because these bits are used by Duktape internal tagged type representation (when using 8-byte packed values), NaN values in the Duktape API are normalized. Concretely, if you push a certain NaN value to the value stack, another (normalized) NaN value may come out. Don't rely on NaNs preserving their exact form.

String §

The string stack type is used to represent both plain strings and plain Symbols (introduced in ES2015). A string is an arbitrary byte sequence of a certain length which may contain internal NUL (0x00) values. Strings are always automatically NUL terminated for C coding convenience. The NUL terminator is not counted as part of the string length. For instance, the string "foo" has byte length 3 and is stored in memory as { 'f', 'o', 'o', '\0' }. Because of the guaranteed NUL termination, strings can always be pointed to using a simple const char * as long as internal NULs are not an issue for the application; if they are, the explicit byte length of the string can be queried with the API. Calling code can refer directly to the string data held by Duktape. Such string data pointers are valid (and stable) for as long as a string is reachable in the Duktape heap.

Strings are interned for efficiency: only a single copy of a certain string ever exists at a time. Strings are immutable and must NEVER be changed by calling C code. Doing so will lead to very mysterious issues which are hard to diagnose.

Calling code most often deals with Ecmascript strings, which may contain arbitrary 16-bit codepoints in the range U+0000 to U+FFFF but cannot represent non-BMP codepoints. This is how strings are defined in the Ecmascript standard. In Duktape, Ecmascript strings are encoded with CESU-8 encoding. CESU-8 matches UTF-8 except that it allows codepoints in the surrogate pair range (U+D800 to U+DFFF) to be encoded directly (prohibited in UTF-8). CESU-8, like UTF-8, encodes all 7-bit ASCII characters as-is which is convenient for C code. For example:

U+0041 ("A") encodes to 41.
U+1234 (ETHIOPIC SYLLABLE SEE) encodes to e1 88 b4.
U+D812 (high surrogate) encodes to ed a0 92. This would be invalid UTF-8.
Duktape also uses extended strings internally. Codepoints up to U+10FFFF can be represented with UTF-8, and codepoints above that up to full 32 bits can be represented with extended UTF-8. The extended UTF-8 encoding used by Duktape is described in the table below. The leading byte is shown in binary (with "x" marking data bits) while continuation bytes are marked with "C" (indicating the bit sequence 10xxxxxx):

Codepoint range	Bits	Byte sequence	Notes
U+0000 to U+007F	7	0xxxxxxx	
U+0080 to U+07FF	11	110xxxxx C	
U+0800 to U+FFFF	16	1110xxxx C C	U+D800 to U+DFFF allowed (unlike UTF-8)
U+1 0000 to U+1F FFFF	21	11110xxx C C C	Above U+10FFFF allowed (unlike UTF-8)
U+20 0000 to U+3FF FFFF	26	111110xx C C C C	
U+400 0000 to U+7FFF FFFF	31	1111110x C C C C C	
U+8000 0000 to U+F FFFF FFFF	36	11111110 C C C C C C	Only 32 bits used in practice (up to U+FFFF FFFF)
The downside of the encoding for codepoints above U+7FFFFFFF is that the leading byte will be 0xFE which conflicts with Unicode byte order marker encoding. This is not a practical concern in Duktape's internal use.

Finally, invalid extended UTF-8 byte sequences are used for special purposes such as representing Symbol values. Invalid extened UTF-8/CESU-8 byte sequences never conflict with standard Ecmascript strings (which are CESU-8) and will remain cleanly separated within object property tables. For more information see Symbols and symbols.rst.

Strings with invalid extended UTF-8 sequences can be pushed on the value stack from C code and also passed to Ecmascript functions, with two caveats:

If the invalid byte sequence matches the internal format used to represent Symbols, the value will appear as a Symbol rather than a string for Ecmascript code. For example, typeof val will be symbol.
Behavior of string operations on invalid byte sequences if not well defined and results may vary, and change even in minor Duktape version updates.
Object §

The object type includes Ecmascript objects and arrays, functions, threads (coroutines), and buffer objects. In other words, anything with properties is an object. Properties are key-value pairs with a string key and an arbitrary value (including undefined).

Objects may participate in garbage collection finalization.

Buffer §

The plain buffer type is a raw buffer for user data. It's much more memory efficient than standard buffer object types like Uint8Array or Node.js Buffer. There are three plain buffer sub-types:

Buffer sub-type	Data pointer	Resizable	Memory managed by	Description
Fixed	Stable, non-NULL	No	Duktape	Buffer size is fixed at creation, memory is managed automatically by Duktape. Fixed buffers have an unchanging (stable) non-NULL data pointer.
Dynamic	Unstable, may be NULL	Yes	Duktape	Buffer size can be changed after creation, memory is managed automatically by Duktape. Requires two memory allocations internally to allow resizing. Data pointer may change if buffer is resized. Zero-size buffer may have a NULL data pointer.
External	Unstable, may be NULL	Yes	Duktape and user code	Buffer data is externally allocated. Duktape allocates and manages a heap allocated buffer header structure, while the data area pointer and length are configured by user code explicitly. External buffers are useful to allow a buffer to point to data structures outside Duktape heap, e.g. a frame buffer allocated by a graphics library. Zero-size buffer may have a NULL data pointer.
Unlike strings, buffer data areas are not automatically NUL-terminated and calling code must never access bytes beyond the currently allocated buffer size. The data pointer of a zero-size dynamic or external buffer may be NULL; fixed buffers always have a non-NULL data pointer.

Fixed and dynamic buffers are automatically garbage collected. This also means that C code must not hold onto a buffer data pointer unless the buffer is reachable to Duktape, e.g. resides in an active value stack. The data area of an external buffer is not automatically garbage collected so user code is responsible for managing its life cycle. User code must also make sure that no external buffer value is accessed when the external buffer is no longer (or not yet) valid.

Plain buffers mimic Uint8Array objects to a large extent, so they are a non-standard, memory efficient alternative to working with e.g. ArrayBuffer and typed arrays. The standard buffer objects are implemented on top of plain buffers, so that e.g. an ArrayBuffer will be backed by a plain buffer. See Buffer objects for more discussion.

A few notes:

Because the value written to a buffer index is number coerced, assigning a one-character value does not work as often expected. For instance, buf[123] = 'x' causes zero to be written to the buffer, as ToNumber('x') = 0. For clarity, you should only assign number values, e.g. buf[123] = 0x78.
There is a fast path for reading and writing numeric indices of plain buffer values, e.g. x = buf[123] or buf[123] = x. A similar fast path exists for the different buffer object values.
Buffer virtual properties are not currently implemented in defineProperty(), so you can't write to buffer indices or buffer length with defineProperty() now (attempt to do so results in a TypeError).
Pointer §

The pointer type is a raw, uninterpreted C pointer, essentially a void *. Pointers can be used to point to native objects (memory allocations, handles, etc), but because Duktape doesn't know their use, they are not automatically garbage collected. You can, however, put one or more pointers inside an object and use the object finalizer to free the native resources related to the pointer(s).

Lightfunc §

The lightfunc type is a plain Duktape/C function pointer and a small set of control flags packed into a single tagged value which requires no separate heap allocations. The control flags (16 bits currently) encode: (1) number of stack arguments expected by the Duktape/C function (0 to 14 or varargs), (2) virtual length property value (0 to 15), and (3) a magic value (-128 to 127).

Lightfuncs are a separate tagged type in the Duktape C API, but behave mostly like Function objects for Ecmascript code. They have significant limitations compared to ordinary Function objects, the most important being:

Lightfuncs cannot hold own properties. They have a fixed virtual .name property which appears in tracebacks, and a virtual .length property. Other properties are inherited from Function.prototype.
Lightfuncs can be used as constructor functions, but cannot have a .prototype property. If you need to construct objects which don't inherit from Object.prototype (the default), you need to either (a) construct and return an instance explicitly in the constructor or (b) explicitly override the internal prototype of the default instance in the constructor.
Lightfuncs can be used as accessor properties (getters/setters), but they get converted to actual functions so that their memory advantage is lost. See test-dev-lightfunc-accessor.js.
Lightfuncs cannot have a finalizer as they are a primitive type and don't have a reference count field or otherwise participate in garbage collection. See test-dev-lightfunc-finalizer.js.
Lightfuncs are useful for very low memory environments where the memory impact of ordinary Function objects matters. See Function objects for more discussion.

API C types §

Duktape API uses typedef-wrapped C types such as duk_int_t, duk_idx_t, and duk_ret_t almost exclusively to ensure portability to exotic platforms. On most platforms these map directly to signed or unsigned int; the wrappers make it possible to support platforms where usual type assumptions (like having a 32-bit int) don't hold. See Wiki article API C types for a detailed discussion.

Summary of best practices:

Use Duktape API types such as duk_idx_t and duk_ret_t when declaring variables for maximum portability. Alternatively you may use plain types (like long) but your code will be less portable and you may need to use casts to avoid warnings.
In printf() and duk_push_sprintf() formatting cast Duktape types to a wide integer type and use a standard format specifier. Example: printf("Result: %ld\n", (long) duk_get_int(ctx, -3));.
Use the L (or UL) suffix for constants which are larger than 16 bits to maximize portability. Like the int type, integer constants without a suffix are only guaranteed to be 16 bits wide. With the L suffix constants are guaranteed to be at least 32 bits wide. Example: duk_push_int(ctx, 1234567L);.
Type algorithms §

This section describes how type-related Ecmascript algorithms like comparisons and coercions are extended to Duktape custom types. Duktape specific type algorithms (ToBuffer() and ToPointer()) are also discussed.

Notation §

The following shorthand is used to indicate how values are compared:

Value	Description
t	compares to true
f	compares to false
s	simple compare: boolean-to-boolean, string-to-string (string contents compared)
n	number compare: NaN values compare false, zeroes compare true regardless of sign (e.g. +0 == -0)
N	number compare in SameValue: NaN values compare true, zeroes compare with sign (e.g. SameValue(+0,-0) is false)
p	heap pointer compare
L	lightfunc compare: to be considered equal, Duktape/C function pointers and internal control flags (including the "magic" value) must match
1	string vs. number: coerce string with ToNumber() and retry comparison
2	boolean vs. any: coerce boolean with ToNumber() and retry comparison
3	object vs. string/number: coerce object with ToPrimitive() and retry comparison
Note that Boolean objects, String objects, and Number objects compare as any other objects instead of being automatically unboxed. For example, non-strict equality compares plain string values with a byte-by-byte comparison, but String objects are compared by object reference (like any other objects).

Equality (non-strict) §

Non-strict equality comparison is specified in The Abstract Equality Comparison Algorithm for standard types. Custom type behavior is as follows:

Buffer: plain buffers are compared by their heap pointer (reference), no content comparison is done. This matches Uint8Array behavior.
Pointer: comparison against any other type returns false. Comparison to a pointer returns true if and only if the pointer values are the same. Note in particular that comparing a number to a pointer returns false. This seems a bit unintuitive, but numbers cannot represent 64-pointers accurately, comparing numbers and pointers might be error prone.
Lightfunc: comparison against any other type returns false. Comparison to a lightfunc returns true if and only if both the Duktape/C function pointers and internal control flags (including the "magic" value) match. Note that a lightfunc never compares equal to an ordinary Function object, even when the Function object was created by coercing a lightfunc to an object.
The standard behavior as well as behavior for Duktape custom types is summarized in the table below:

und	nul	boo	num	str	obj	buf	ptr	lfn
und	t	t	f	f	f	f	f	f	f
nul		t	f	f	f	f	f	f	f
boo			s	2	2	2	f	f	f
num				n	1	3	f	f	f
str					s	3	f	f	f
obj						p	f	f	f
buf							p	f	f
ptr								s	f
lfn									L
Strict equality §

Strict equality is much more straightforward and preferable whenever possible for simplicity and performance. It is described in The Strict Equality Comparison Algorithm for standard types. Custom type behavior is as follows:

Buffer: like non-strict equality.
Pointer: like non-strict equality.
Lightfunc: like non-strict equality.
The standard behavior as well as behavior for Duktape custom types is summarized in the table below:

und	nul	boo	num	str	obj	buf	ptr	lfn
und	t	f	f	f	f	f	f	f	f
nul		t	f	f	f	f	f	f	f
boo			s	f	f	f	f	f	f
num				n	f	f	f	f	f
str					s	f	f	f	f
obj						p	f	f	f
buf							p	f	f
ptr								s	f
lfn									L
SameValue §

The SameValue algorithm is not easy to invoke from user code. It is used by e.g. Object.defineProperty() when checking whether a property value is about to change. SameValue is even stricter than a strict equality comparison, and most notably differs in how numbers are compared. It is specified in The SameValue algorithm for standard types. Custom type behavior is as follows:

Buffer: like non-strict (and strict) equality.
Pointer: like non-strict (and strict) equality.
Lightfunc: like non-strict (and strict equality).
The standard behavior as well as behavior for Duktape custom types is summarized in the table below:

und	nul	boo	num	str	obj	buf	ptr	lfn
und	t	f	f	f	f	f	f	f	f
nul		t	f	f	f	f	f	f	f
boo			s	f	f	f	f	f	f
num				N	f	f	f	f	f
str					s	f	f	f	f
obj						p	f	f	f
buf							p	f	f
ptr								s	f
lfn									L
Type conversion and testing §

The custom types behave as follows for Ecmascript coercions described Type Conversion and Testing (except SameValue which was already covered above):

buffer	pointer	lightfunc
DefaultValue	Usually "[object Uint8Array]"; like Uint8Array	TypeError	"light_<PTR>_<FLAGS>" (toString/valueOf)
ToPrimitive	Usually "[object Uint8Array]"; like Uint8Array	identity	"light_<PTR>_<FLAGS>" (toString/valueOf)
ToBoolean	true	false for NULL pointer, true otherwise	true
ToNumber	ToNumber(String(buffer)), usually ToNumber("[object Uint8Array]") = NaN	0 for NULL pointer, 1 otherwise	NaN
ToInteger	same as ToNumber; usually 0	same as ToNumber	0
ToInt32	same as ToNumber; usually 0	same as ToNumber	0
ToUint32	same as ToNumber; usually 0	same as ToNumber	0
ToUint16	same as ToNumber; usually 0	same as ToNumber	0
ToString	Usually [object Uint8Array]; like Uint8Array	sprintf() with %p format (platform specific)	"light_<PTR>_<FLAGS>"
ToObject	Uint8Array object (backs to argument plain buffer)	Pointer object	Function object
CheckObjectCoercible	allow (no error)	allow (no error)	allow (no error)
IsCallable	false	false	true
SameValue	(covered above)	(covered above)	(covered above)
When a buffer is string coerced it behaves like an Uint8Array, with the result usually being "[object Uint8Array]". This behavior was changed in Duktape 2.0. To create a string from buffer contents you can use e.g. the Node.js Buffer binding or the Encoding API.

When a buffer is object coerced a new Uint8Array object is created, with a new ArrayBuffer backing to the plain buffer (no copy is made).

When a lightfunc is coerced with ToPrimitive() it behaves like an ordinary function: it gets coerced with Function.prototype.toString() with the result (normally) being the same as ToString() coercion.

When a lightfunc is object coerced, a new Function object is created and the virtual properties (name and length and the internal "magic" value are copied over to the Function object.

Custom coercions (ToBuffer, ToPointer) §

ToBuffer() coercion is used when a value is forced into a buffer type e.g. with the duk_to_buffer() API call. The coercion is as follows:

A buffer coerces to itself (identity). The same buffer value is returned.
Any other type (including pointer and lightfunc) is first string coerced with ToString, and the resulting string is then copied, byte-by-byte, into a fixed-size buffer.
ToPointer() coercion is used e.g. by the duk_to_pointer() call. The coercion is as follows:

A pointer coerces to itself.
Heap-allocated types (string, object, buffer) coerce to a pointer value pointing to their internal heap header. This pointer has only a diagnostic value. Note, in particular, that the pointer returned for a buffer or a string does not point to the buffer/string data area. (This coercion is liable to change even in minor versions, avoid relying on it.)
Any other types (including number) coerce to a NULL pointer.
Lightfunc coerces to a NULL pointer. This is the case because C function pointers cannot be coerced to a void * in a portable manner.
The following table summarizes how different types are handled:

ToBuffer	ToPointer
undefined	buffer with "undefined"	NULL
null	buffer with "null"	NULL
boolean	buffer with "true" or "false"	NULL
number	buffer with string coerced number	NULL
string	buffer with copy of string data	ptr to heap hdr
object	buffer with ToString(value)	ptr to heap hdr
buffer	identity	ptr to heap hdr
pointer	sprintf() with %p format (platform specific)	identity
lightfunc	buffer with ToString(value)	NULL
There is currently no ToLightFunc() coercion. Lightfuncs can only be created using the Duktape C API.
Addition §

The Ecmascript addition operator is specified in The Addition operator (+). Addition behaves specially if either argument is a string: the other argument is coerced to a string and the strings are then concatenated. This behavior is extended to custom types as follows:

As for standard types, object values are first coerced with ToPrimitive(); plain buffers and lightfuncs are normally coerced with ToString(). For plain buffers the result is usually "[object Uint8Array]" and for lightfuncs "[object Function]".
Pointer values fall into the default number addition case. They are coerced with ToNumber() and then added as numbers. NULL pointers coerce to 0, non-NULL pointers to 1, so addition results may not be very intuitive.
Addition is not generally useful for custom types. For example, if two plain buffers are added, the result is usually "[object Uint8Array][object Uint8Array]", which matches how standard addition behaves for two Uint8Array instances.

Property access §

If a plain buffer or pointer is used as a property access base value, properties are looked up from the (initial) built-in prototype object (Uint8Array.prototype or Duktape.Pointer.prototype). This mimics the behavior of standard types.

For example:

duk> buf = Duktape.dec('hex', '414243');  // plain buffer
= ABC
duk> buf.subarray();
= function subarray() {"native"}
duk> typeof buf.toString();
= string
Lightfuncs have a few non-configurable and non-writable virtual properties (name and length) and inherit their remaining properties from Function.prototype, which allows ordinary inherited Function methods to be called:

var bound = myLightFunc.bind('dummy', 123);
Duktape built-ins §

This section summarizes Duktape-specific and non-Ecmascript built-in objects, methods, and values.

Additional global object properties §

Property	Description
global	References the global object itself. See proposal-global.
Duktape	The Duktape built-in object. Contains miscellaneous implementation specific stuff.
TextEncoder	TextEncoder() from WHATWG Encoding API. Converts a string into a buffer using UTF-8 encoding.
TextDecoder	TextDecoder() from WHATWG Encoding API. Converts a buffer into a string using UTF-8 encoding.
The Duktape object §

Property	Description
version	Duktape version number: (major * 10000) + (minor * 100) + patch.
env	Cryptic, version dependent summary of most important effective options like endianness and architecture.
fin	Set or get finalizer of an object.
enc	Encode a value (hex, base-64, JX, JC): Duktape.enc('hex', 'foo').
dec	Decode a value (hex, base-64, JX, JC): Duktape.dec('base64', 'Zm9v').
info	Get internal information (such as heap address and alloc size) of a value in a version specific format. The C API equivalent is duk_inspect_value().
act	Get information about call stack entry.
gc	Trigger mark-and-sweep garbage collection.
compact	Compact the memory allocated for a value (object).
errCreate	Callback to modify/replace a created error.
errThrow	Callback to modify/replace an error about to be thrown.
Pointer	Pointer constructor (function).
Thread	Thread constructor (function).
version §
The version property allows version-based feature detection and behavior. Version numbers can be compared directly: a logically higher version will also be numerically higher. For example:

if (typeof Duktape !== 'object') {
    print('not Duktape');
} else if (Duktape.version >= 10203) {
    print('Duktape 1.2.3 or higher');
} else if (Duktape.version >= 800) {
    print('Duktape 0.8.0 or higher (but lower than 1.2.3)');
} else {
    print('Duktape lower than 0.8.0');
}
The value of version for pre-releases is one less than the actual release, e.g. 1199 for a 0.12.0 pre-release and 10299 for a 1.3.0 pre-release. See Versioning.

Remember to check for existence of Duktape when doing feature detection. Your code should typically work on as many engines as possible. Avoid the common pitfall of using a direct identifier reference in the check:

// Bad idea: ReferenceError if missing
if (!Duktape) {
    print('not Duktape');
}

// Better: check through 'this' (bound to global)
if (!this.Duktape) {
    print('not Duktape');
}

// Better: use typeof to check also type explicitly
if (typeof Duktape !== 'object') {
    print('not Duktape');
}
env §
env summarizes the most important effective compile options in a version specific, quite cryptic manner. The format is version specific and is not intended to be parsed programmatically. This is mostly useful for developers (see duk_hthread_builtins.c for the code which sets the value).

Example from Duktape 1.1.0:

ll u n p2 a4 x64 linux gcc     // l|b|m integer endianness, l|b|m IEEE double endianness
                               // p|u packed/unpacked tval
                               // n|various, memory optimization options (n = none)
                               // p1|p2|p3 prop memory layout
                               // a1|a4|a8: align target
                               // x64|x86|arm|etc: architecture
                               // linux|windows|etc: operating system
                               // gcc|clang|msvc|etc: compiler
fin() §
When called with a single argument, gets the current finalizer of an object:

var currFin = Duktape.fin(o);
When called with two arguments, sets the finalizer of an object (returns undefined):

Duktape.fin(o, function(x) { print('finalizer called'); });
Duktape.fin(o, undefined);  // disable
enc() §
enc() encodes its argument value into chosen format. The first argument is a format (currently supported are "hex", "base64", "jx" and "jc"), second argument is the value to encode, and any further arguments are format specific.

For "hex" and "base64", buffer values are encoded as is, other values are string coerced and the internal byte representation (extended UTF-8) is then encoded. The result is a string. For example, to encode a string into base64:

var result = Duktape.enc('base64', 'foo');
print(result);  // prints 'Zm9v'
For "jx" and "jc" the argument list following the format name is the same as for JSON.stringify(): value, replacer (optional), space (optional). For example:

var result = Duktape.enc('jx', { foo: 123 }, null, 4);
print(result);  // prints JX encoded {foo:123} with 4-space indent
dec() §
dec() provides the reverse function of enc().

For "hex" and "base64" the input value is first string coerced (it only really makes sense to decode strings). The result is always a plain buffer. For example:

var result = Duktape.dec('base64', 'Zm9v');
print(typeof result, result);  // prints 'object foo'
If you prefer a full Uint8Array over a plain buffer, you can coerce the result as follows:

var result = Object(Duktape.dec('base64', 'Zm9v'));
print(typeof result, result);  // prints 'object foo'
If you wish to get back a string value, you can coerce the plain buffer to a string e.g. as follows:

// Use TextDecoder which decodes the input as UTF-8.  You can also use
// the Node.js Buffer binding to achieve a similar result.

var result = new TextDecoder().decode(Duktape.dec('base64', 'Zm9v'));
print(typeof result, result);  // prints 'string foo'
For "jx" and "jc" the argument list following the format name is the same as for JSON.parse(): text, reviver (optional). For example:

var result = Duktape.dec('jx', "{foo:123}");
print(result.foo);  // prints 123
info() §
Duktape.info() returns an object exposing internal information related to its argument value. See duk_inspect_value() for description of current fields.

The properties of the result object are not under versioning guarantees and may change in an incompatible fashion even in minor versions (but not patch versions).
act() §
Get information about a call stack entry. Takes a single number argument indicating depth in the call stack: -1 is the top (innermost) entry, -2 is the one below that etc. Returns an object describing the call stack entry, or undefined if the entry doesn't exist. See duk_inspect_callstack_entry() for description of current fields.

The properties of the result object are not under versioning guarantees and may change in an incompatible fashion even in minor versions (but not patch versions).
Example:

function dump() {
    var i, t;
    for (i = -1; ; i--) {
        t = Duktape.act(i);
        if (!t) { break; }
        print(i, t.lineNumber, t.function.name, Duktape.enc('jx', t));
    }
}

dump();
The example, when executed with the command line tool, currently prints something like:

-1 0 act {lineNumber:0,pc:0,function:{_func:true}}
-2 4 dump {lineNumber:4,pc:16,function:{_func:true}}
-3 10 global {lineNumber:10,pc:5,function:{_func:true}}
The interesting entries are lineNumber and function which provides e.g. the function name.

You can also implement a helper to get the current line number using Duktape.act():

function getCurrentLine() {
    'use duk notail';

    /* Tail calls are prevented to ensure calling activation exists.
     * Call stack indices: -1 = Duktape.act, -2 = getCurrentLine, -3 = caller
     */

    var a = Duktape.act(-3) || {};
    return a.lineNumber;
}
print('running on line:', getCurrentLine());
gc() §
Trigger a forced mark-and-sweep collection. The call takes an optional integer flags field, see duktape.h for constants.

compact() §
Minimize the memory allocated for a target object. Same as the C API call duk_compact() but accessible from Ecmascript code. If called with a non-object argument, this call is a no-op. The argument value is returned by the function, which allows code such as:

var obj = {
    foo: Duktape.compact({ bar:123 })
}
This call is useful when you know that an object is unlikely to gain new properties, but you don't want to seal or freeze the object in case it does.

errCreate() and errThrow() §
These can be set by user code to process/replace errors when they are created (errCreate) or thrown (errThrow). Both values are initially non-existent.

See Error handlers (errCreate and errThrow) for details.

Duktape.Pointer (constructor) §

Property	Description
prototype	Prototype for Pointer objects.
The Pointer constructor is a function which can be called both as an ordinary function and as a constructor:

When called as a function, coerces the first argument to a pointer using the custom ToPointer coercion. The return value is a plain pointer (not a Pointer object).
When called as a constructor, coerces the first argument to a pointer using the custom ToPointer coercion. Returns a Pointer object whose internal value is the pointer resulting from the coercion. The internal prototype of the newly created Pointer will be the Duktape.Pointer.prototype object.
Duktape.Pointer.prototype §

Property	Description
toString	Convert Pointer to a printable string.
valueOf	Return the primitive pointer value held by Pointer.
toString() and valueOf accept both plain pointers and Pointer objects as their this binding. This allows code such as:

var plain_ptr = Duktape.Pointer({ test: 'object' });
print(plain_ptr.toString());
Duktape.Thread (constructor) §

Property	Description
prototype	Prototype for Thread objects.
resume	Resume target thread with a value or an error. Arguments: target thread, value, flag indicating whether value is to be thrown (optional, default false).
yield	Yield a value or an error from current thread. Arguments: value, flag indicating whether value is to be thrown (optional, default false).
current	Get currently running Thread object.
The Thread constructor is a function which can be called both as an ordinary function and as a constructor. The behavior is the same in both cases:

The first argument is checked to be a function (if not, a TypeError is thrown). The function must be an Ecmascript function (bound or non-bound). The return value is a new thread whose initial function is recorded to be the argument function (this function will start executing when the new thread is first resumed). The internal prototype of the newly created Thread will be the Duktape.Thread.prototype object.
Duktape.Thread.prototype §

Property	Description
No properties at the moment.
TextEncoder §

TextEncoder() is part of the WHATWG Encoding API and provides a clean way of encoding a string into a buffer (Uint8Array) using UTF-8 encoding. Surrogate pairs are combined during the process. For example:

var str = '\u{1f4a9}';                   // non-BMP codepoint
print(str.length);                       // length is 2, represented as a surrogate pair
var u8 = new TextEncoder().encode(str);
print(u8.length);                        // length is 4, a single UTF-8 codepoint
print(Duktape.enc('jx', u8));            // |f09f92a9|, UTF-8 bytes F0 9F 92 A9
TextDecoder §

TextDecoder() is part of the WHATWG Encoding API and provides a clean way of decoding a buffer into a string using UTF-8 encoding. Non-BMP codepoints are represented as surrogate pairs in the resulting string. For example:

var u8 = new Uint8Array([ 0xf0, 0x9f, 0x92, 0xa9 ]);  // a single non-BMP codepoint
var str = new TextDecoder().decode(u8);
print(str.length);                       // length is 2, represented as a surrogate pair
print(str.charCodeAt(0));                // 55357, high surrogate
print(str.charCodeAt(1));                // 56489, low surrogate
Post-ES5 features §

Duktape implements features from ES2015 (ES6), ES2016 (ES7), and later specification drafts. See Wiki article Post-ES5 features for current status.

Duktape status is also updated for new releases in kangax/compat-table.

Custom behavior §

This section summarizes Duktape behavior which deviates from the E5.1 or other relevant specifications.

Duktape built-in and custom types §

The Duktape built-in is (of course) non-standard and provides access to Duktape specific features. Also the buffer, pointer, and lightfunc types are custom.

Hidden Symbols §

Objects may have properties with hidden Symbol keys. These are similar to ES2015 Symbols but won't be enumerated or returned from even Object.getOwnPropertySymbols(). Ordinary Ecmascript code cannot refer to such properties because the keys intentionally use an invalid (extended) UTF-8 representation.

"use duk notail" directive §

The "use duk notail" directive is non-standard. It prevents a function from being tail called.

"const" treated mostly like "var" §

The const keyword is supported with minimal non-standard semantics (officially defined in Ecmascript 6). See Const variables for more detail.

Additional Error and Function object properties §

See Error objects and Function objects.

Non-strict function instances don't have a caller property in the E5/E5.1 specification. Some real world code expects to have this property, so it can be enabled with the config option DUK_USE_NONSTD_FUNC_CALLER_PROPERTY.

Function statements §

E5.1 does not allow a function declaration to appear outside program or function top level:

function test() {
    // point A
    try {
        throw new Error('test');
    } catch (e) {
        // This is a SyntaxError in E5.1
        function func() {
            print(typeof e);
        }
        // point B
    }
    // point C
}
These declarations are also referred to as "function statements", and appear quite often in real world code (including the test262 test suite), so they are allowed by Duktape. Unfortunately there are several semantics used by different Javascript engines (ES2015 unfortunately doesn't specify semantics for function statements either). Duktape follows the V8 behavior for function statements:

Strict function: a SyntaxError is thrown (standard behavior).
Non-strict function: treat a function statement like an ordinary function declaration, conceptually "hoisting" it to the top of the function.
As an illustration, the above example would behave as the following:

function test() {
    function func() {
        print(typeof e);
    }
 
    try {
        throw new Error('test');
    } catch (e) {
    }
}
func() in the above example would already be declared and callable in point A, and would not have access to the e binding in any of the points A, B, or C.

RegExp leniency §

Most Ecmascript engines support more syntax than guaranteed by the Ecmascript E5.1 specification (Section 15.10.1 Patterns). As a result there's quite a lot of code that won't work with strict Ecmascript E5.1 regexp syntax. Much of the additional syntax expected of web browser engines is documented in ES2015 Annex B.1.4 Regular Expression Patterns. However, note that features in Annex B Additional ECMAScript Features for Web Browsers are not recommended for new code: "These features are not considered part of the core ECMAScript language. Programmers should not use or assume the existence of these features and behaviours when writing new ECMAScript code. ECMAScript implementations are discouraged from implementing these features unless the implementation is part of a web browser or is required to run the same legacy ECMAScript code that web browsers encounter."

Duktape also allows some ES2015 Annex B syntax to better support existing code. You can turn this non-standard behavior off using config options if you prefer. Some examples of additional syntax supported:

  /{(\d+)}/    // unescaped left curly, digits, unescaped right curly; ES2015 Annex B
  /\{(\d+)\}/  // same, ES5 compliant

  /]/          // unescaped right bracket; ES2015 Annex B
  /\]/         // same, ES5 compliant

  /\$/         // literal dollar using escape; ES2015 Annex B
  /\u0024/     // same, ES5 compliant
Array.prototype.splice() when deleteCount not given §

When deleteCount (the 2nd argument) is not given to Array.prototype.splice(), the standard behavior is to work as if the 2nd argument was undefined (or 0, which has the same behavior after coercions). A more real world compatible behavior is to treat the missing argument like positive infinity, i.e. to extend the splice operation to the end of the array.

Because the non-standard real world behavior is expected by much existing code, Duktape uses this behavior by default. The strict standards compliant behavior can be enabled by disabling the config option DUK_USE_NONSTD_ARRAY_SPLICE_DELCOUNT.

Array.prototype.concat() trailing non-existent elements §

When the result of an array concat() would have trailing non-existent elements, the standard behavior is to ignore them so that they are not reflected in the result length. Real world behavior is to include them in the result value length. See test-bi-array-proto-concat-nonstd-trailing.js.

The real world behavior seems consistent in other engines (V8, Rhino, Spidermonkey at least), so Duktape uses the real world behavior by default. The strict standards compliant behavior can be enabled by disabling the config option DUK_USE_NONSTD_ARRAY_CONCAT_TRAILER.

Array.prototype.map() trailing non-existent elements §

Similar issue as with Array.prototype.concat(), see test-bi-array-proto-map-nonstd-trailing.js. The strict standards compliant behavior can be enabled by disabling the config option DUK_USE_NONSTD_ARRAY_MAP_TRAILER.

Setter/getter key argument §

Ecmascript standard behavior is that setters and getters are not given the name of the property being accessed. This prevents reusing a single setter or a getter for multiple properties; separate functions are needed for each property which is sometimes inconvenient and wastes memory.

Duktape provides the property key name as a non-standard additional argument to setter and getter functions. See test-dev-nonstd-setget-key-argument.js and Property virtualization for more discussion. The strict standards compliant behavior can be enabled by disabling the config options DUK_USE_NONSTD_GETTER_KEY_ARGUMENT and DUK_USE_NONSTD_SETTER_KEY_ARGUMENT.

Object.setPrototypeOf and Object.prototype.__proto__ (ES2015) §

See Object.setPrototypeOf and Object.prototype.__proto__.

Proxy object (ES2015) §

See Proxy object (subset).

JSON.stringify() escapes U+2028 and U+2029 §

JSON.stringify() standard behavior is to output U+2028 and U+2029 without escaping. This leads to counterintuitive behavior when the output is used in a web page or parsed with eval(): the U+2028 and U+2029 characters are considered line terminators which leads to a syntax error (unterminated string). Duktape escapes U+2028 and U+2029 by default to avoid this issue; you can turn on the compliant behavior by disabling the config option DUK_USE_NONSTD_JSON_ESC_U2028_U2029.

String.fromCharCode() accepts 32-bit codepoints §

String.fromCharCode() standard behavior is to use ToUInt16() coercion for codepoint values. Duktape uses ToUint32() by default to better support non-BMP strings. You can force the compliant behavior by disabling the config optipn DUK_USE_NONSTD_STRING_FROMCHARCODE_32BIT.

Array instance numeric index writes §

By default Duktape provides a fast path for writing to Array instances. The fast path is active when numeric indices are used (e.g. arr[7] = 'foo') and a few internal conditions are met. When the fast path is taken, Duktape doesn't check Array.prototype for conflicting properties (these are very rare in practical code), which makes common array writes faster. The behavior is non-compliant, but there's no outward difference unless Array.prototype has properties with numeric keys. You can turn on the compliant behavior by disaling the config options DUK_USE_NONSTD_ARRAY_WRITE and DUK_USE_ARRAY_PROP_FASTPATH. See the following for more details on the fast path behavior: test-misc-array-fast-write.js.

TypedArray binding §

Duktape provides the ES2015 TypedArray binding, with some details yet to be fixed, e.g. small differences in argument coercion e.g. for offset and length values.

The plain buffer custom type behaves mostly like an Uint8Array object for Ecmascript code, but has a separate type in the Duktape C API.

Node.js Buffer binding §

Duktape provides a Node.js-like Buffer binding. There are some differences between the Node.js behavior and Duktape behavior. These differences include:

Interoperability with other buffer types: ArrayBuffer, DataView, or a typed array (Uint8Array etc) is usually allowed wherever a Node.js Buffer would be allowed.
Buffer data is always zeroed on allocation and when concatenation totalLength exceeds combined size of input buffers.
Read/write offset and length arguments are always validated to ensure memory safe behavior, even when noAssert is true. Failed reads return NaN and failed writes return 0.
Partial read/writes are never done: if a read/write is partially outside the valid buffer, it is rejected.
Small differences in argument coercion for e.g. offset and length values.
Write calls coerce their value argument like TypedArray, e.g. when writing 0x100 using writeUInt8() silently coerces to 0x00 rather than throwing a TypeError.
Duktape only supports the "utf8" encoding (and accepts no spelling variants). Most API calls ignore an encoding argument, and use UTF-8 implicitly for string-to-buffer coercion.
UTF-8 decoding replacement character approach follows Unicode Technical Committee Recommended Practice for Replacement Characters which matches WHATWG Encoding API specification but differs from Node.js (at least up to version v6.9.1).
Shebang comment support §

duk_compile() flag DUK_COMPILE_SHEBANG allows shebang comment parsing: #! on the first column of the first line causes the line to be treated as a comment. For example:

#!/usr/bin/duk
print('Hello world!');
The feature can be disabled by undefining DUK_USE_SHEBANG_COMMENTS.

Custom JSON formats §

Ecmascript JSON shortcomings §

The standard JSON format has a number of shortcomings when used with Ecmascript:

undefined and function values are not supported
NaN and infinity values are not supported
Duktape custom types are, of course, not supported
Codepoints above BMP cannot be represented except as surrogate pairs
Codepoints above U+10FFFF cannot be represented even as surrogate pairs
The output is not printable ASCII which is often inconvenient
These limitations are part of the Ecmascript specification which explicitly prohibits more lenient behavior. Duktape provides two more programmer friendly custom JSON format variants: JX and JC, described below.

Custom JX format §

JX encodes all values in a very readable manner and parses back almost all values in a faithful manner (function values being the most important exception). Output is pure printable ASCII, codepoints above U+FFFF are encoded with a custom escape format, and quotes around object keys are omitted in most cases. JX is not JSON compatible but a very readable format, most suitable for debugging, logging, etc.

JX is used as follows:

var obj = { foo: 0/0, bar: [ 1, undefined, 3 ] };
print(Duktape.enc('jx', obj));
// prints out: {foo:NaN,bar:[1,undefined,3]}

var dec = Duktape.dec('jx', '{ foo: 123, bar: undefined, quux: NaN }');
print(dec.foo, dec.bar, dec.quux);
// prints out: 123 undefined NaN
Custom JC format §

JC encodes all values into standard JSON. Values not supported by standard JSON are encoded as objects with a marker key beginning with an underscore (e.g. {"_ptr":"0xdeadbeef"}). Such values parse back as ordinary objects. However, you can revive them manually more or less reliably. Output is pure printable ASCII; codepoints above U+FFFF are encoded as plain string data with the format "U+nnnnnnnn" (e.g. U+0010fedc).

JC is used as follows:

var obj = { foo: 0/0, bar: [ 1, undefined, 3 ] };
print(Duktape.enc('jc', obj));
// prints out: {"foo":{"_nan":true},"bar":[1,{"_undef":true},3]}

var dec = Duktape.dec('jc', '{ "foo": 123, "bar": {"_undef":true}, "quux": {"_nan":true} }');
print(dec.foo, dec.bar, dec.quux);
// prints out: 123 [object Object] [object Object]
The JC decoder is essentially the same as the standard JSON decoder at the moment: all JC outputs are valid JSON and no custom syntax is needed. As shown in the example, custom values (like {"_undef":true}) are not revived automatically. They parse back as ordinary objects instead.

Codepoints above U+FFFF and invalid UTF-8 data §

All standard Ecmascript strings are valid CESU-8 data internally, so behavior for codepoints above U+FFFF never poses compliance issues. However, Duktape strings may contain extended UTF-8 codepoints and may even contain invalid UTF-8 data.

The Duktape JSON implementation, including the standard Ecmascript JSON API, use replacement characters to deal with invalid UTF-8 data. The resulting string may look a bit odd, but this behavior is preferable to throwing an error.

JSON format examples §

The table below summarizes how different values encode in each encoding:

Value	Standard JSON	JX	JC	Notes
undefined	n/a	undefined	{"_undef":true}	Standard JSON: encoded as null inside arrays, otherwise omitted
null	null	null	null	standard JSON
true	true	true	true	standard JSON
false	false	false	false	standard JSON
123.4	123.4	123.4	123.4	standard JSON
+0	0	0	0	standard JSON
-0	0	-0	-0	Standard JSON allows -0 but serializes negative zero as 0 (losing the sign unnecessarily)
NaN	null	NaN	{"_nan":true}	Standard JSON: always encoded as null	
Infinity	null	Infinity	{"_inf":true}	Standard JSON: always encoded as null
-Infinity	null	-Infinity	{"_ninf":true}	Standard JSON: always encoded as null
"köhä"	"köhä"	"k\xf6h\xe4"	"k\u00f6h\u00e4"	
U+00FC	"\u00fc"	"\xfc"	"\u00fc"	
U+ABCD	"\uabcd"	"\uabcd"	"\uabcd"	
U+1234ABCD	"U+1234abcd"	"\U1234abcd"	"U+1234abcd"	Non-BMP characters are not standard Ecmascript, JX format borrowed from Python
object	{"my_key":123}	{my_key:123}	{"my_key":123}	ASCII keys matching identifer requirements encoded without quotes in JX
array	["foo","bar"]	["foo","bar"]	["foo","bar"]	
buffer	n/a	|deadbeef|	{"_buf":"deadbeef"}	
pointer	n/a	(0xdeadbeef)
(DEADBEEF)	{"_ptr":"0xdeadbeef"}
{"_ptr":"DEADBEEF"}	Representation inside parentheses or quotes is platform specific
NULL pointer	n/a	(null)	{"_ptr":"null"}	
function	n/a	{_func:true}	{"_func":true}	Standard JSON: encoded as null inside arrays, otherwise omitted
lightfunc	n/a	{_func:true}	{"_func":true}	Formats like ordinary functions
Limitations §

Some limitations include:

Only enumerable own properties are serialized in any of the formats.
Array properties (other than the entries) are not serialized. This would be useful in e.g. logging, e.g. as [1,2,3,"type":"point"].
There is no automatic revival of special values when parsing JC data.
There is no canonical encoding. This would be easy to arrange with a simple option to sort object keys during encoding.
(See internal documentation for more future work issues.)

Custom directives §

Ecmascript E5/E5.1 employs a directive prologue to allow version or implementation specific features be activated. The standard only provides one such directive, "use strict", while asm.js uses "use asm". Duktape custom directives are discussed in this section.

use duk notail §

The use duk notail directive indicates that the function should never be tail called. Tail calls affect the call stack so they are visible in stack traces (usually harmless) and affect functions which inspect the call stack using e.g. Duktape.act(). This directive may be useful in special cases to ensure call stack has a known shape. Example:

function noTailCall() {
    'use duk notail';

    // ...
}
Native functions are never tailcalled, so a corresponding declaration is not necessary for them.

Buffer objects §

Overview of buffer types §

Duktape provides the following buffer and buffer-related types:

Type	Standard	Duktape version	Description
Plain buffer	No
Duktape specific	1.0	Plain, primitive buffer value (not an object), similar to how a plain string relates to a String object. Behaves like an Uint8Array instance where possible, object coerces to an actual Uint8Array.
ArrayBuffer object	Yes
ES2015	1.3	Standard object type for representing a byte array. References an underlying plain buffer.
DataView, typed array objects	Yes
ES2015	1.3	View objects to access an underlying ArrayBuffer. References an underlying plain buffer.
Node.js Buffer object	No
Node.js-like	1.3	Object with Node.js Buffer API, inherits from Uint8Array.prototype. References an underlying plain buffer.
See buffers.rst for a detailed discussion, including a detailed table of buffer types and their properties.

Plain buffers §

Plain buffers are a non-standard memory efficient way of representing buffer data. Plain buffers mimic Uint8Array objects so that they inherit from Uint8Array.prototype, are accepted as typed array constructor arguments, and so on. While plain buffers don't have a property table and can't hold properties of their own, they have the following virtual or inherited properties (example values are for a 24-byte buffer):

Property name	Example value	Description
[index]	0-255	Index properties in the range [0, length-1]. Reads and writes behave like for Uint8Array.
length	24	Length of buffer in bytes. Length is not writable, so you can't resize a buffer by assigning its length.
byteOffset	0	Always 0, present to match typed arrays.
byteLength	24	Same as .length.
BYTES_PER_ELEMENT	1	Always 1, present to match typed arrays.
buffer	 	Getter property which returns a new ArrayBuffer instance backing to the plain buffer without making a copy. Because plain buffers don't have a property table, a new ArrayBuffer is created on every property read. Absent if buffer object support is disabled in Duktape configuration.
Buffer objects like ArrayBuffer and Node.js Buffer are implemented on top of plain buffer values and provide additional functionality like view/slice support, typed accessors, and methods to manipulate data in different endianness. However, they have more overhead than plain buffers.

For more details, see:

How to work with buffers
Type algorithms
buffers.rst
Working with buffers §

Buffer values work in both C and Ecmascript code:

For Ecmascript code most of the behavior is defined in the relevant API standards, with exceptions for Duktape-specific features like mixing different buffer types.
For C code there are API calls to work with plain buffers and buffer objects.
See How to work with buffers for examples.

In special cases the plain buffer backing a buffer object may not be large enough to cover the apparent size of the buffer object; the buffer object is then "uncovered" or "unbacked". Duktape guarantees memory safe behavior for such buffers, but other than that behavior varies between calls. For example, a call may ignore the situation silently returning undefined, NaN, or zero, or it may throw a TypeError. The behavior for unbacked buffers is not part of versioning guarantees and may change between minor versions.
Current limitations §

See TypedArray binding custom behavior.
See Node.js Buffer binding custom behavior.
Error objects §

Property summary §

Ecmascript Error objects have very few standard properties, so many Ecmascript implementations have added quite a few custom properties. Duktape uses standard Error properties but also borrows the most useful properties used by other implementations. The number of "own" properties of error objects is minimized to keep error objects as small as possible.

Error objects have the following properties (mostly inherited):

Property name	Compatibility	Description
name	standard	Name of error, e.g. TypeError, inherited
message	standard	Optional message of error, own property, empty message inherited if absent
fileName	Rhino	Filename related to error source, inherited accessor
lineNumber	Rhino	Linenumber related to error source, inherited accessor
stack	V8	Traceback as a multi-line human redable string, inherited accessor
Assigning the most useful fileName and lineNumber is somewhat complicated. The related issues and current behavior are described in: error-objects.rst.
If Duktape is compiled with traceback support:

stack, fileName, and lineNumber are accessor properties inherited from Error.prototype. You can override the properties by simply assigning over them: an inherited setter will capture the write but will create an own property as if a normal assignment was done. This behavior was changed in Duktape 1.4.0 to better match other engines.
The raw traceback data needed by the accessor properties is stored in an internal property (\xFFTracedata) which is not normally accessible from Ecmascript code.
If Duktape is compiled without traceback support:

The stack accessor will be equivalent to Error.prototype.toString(), so that printing the stacktrace always produces a useful, human readable result.
fileName and lineNumber will be own properties of the Error object. You can override the properties using assignment.
When error objects are created using the Duktape API from C code and the caller does not give a format string for a message, the message property is set to a numeric error code given in the API call. The type of message will be number in this case; normally error messages are strings. In minimized Duktape builds all errors generated internally by Duktape use numeric error codes only.

An object is considered an "error object" if its internal prototype chain contains the (original) Error.prototype object. Only objects matching this criteria get augmented with e.g. traceback data.

Traceback §

The stack property is an accessor (setter/getter) property which provides a printable traceback related to an error. The traceback reflects the call stack when the error object was created (not thrown). Traceback data is automatically collected and added to an object:

when an Error instance is constructed;
when an error is thrown from C code using the Duktape API;
when an error is thrown from inside Duktape.
The data used to create the traceback is stored in an internal property (\xFFTracedata), in an internal and version-dependent format described error-objects.rst. You shouldn't access the traceback data directly.

The printable traceback format is intended to be human readable only. You shouldn't rely on an exact traceback format as it may change between versions (for example, tracebacks were improved for the 1.5.0 release). As an example of the current traceback format, the program:

// shortened from tests/ecmascript/test-dev-traceback-example.js
try {
    decodeURIComponent('%e1%a9%01');  // invalid utf-8
} catch (e) {
    print(e.stack);
}
would print something like:

URIError: invalid input
    at [anon] (duk_bi_global.c:343) internal
    at decodeURIComponent () native strict preventsyield
    at global (test.js:3) preventsyield
In builds where tracebacks are disabled, the stack accessor will return the same value as calling toString() on the error would. This means you can always print e.stack and get a useful output.

The most portable traceback printing approach is something like:

try {
    decodeURIComponent('%e1%a9%01');  // invalid utf-8
} catch (e) {
    // Print stacktrace on at least Duktape and V8, or a standard error
    // string otherwise.
    print(e.stack || e);
}
Attempt to write to stack is silently ignored. You can still override the accessor by defining an own property of the same name explicitly with Object.defineProperty(). This behavior differs from V8 where stack is an own property of the Error instance, and if you assign a value to stack, the value reads back as assigned.

Error handlers (errCreate and errThrow) §

If Duktape.errCreate has been set, it is called right after Duktape has added traceback information to an object, and can process the error further or even replace the error value entirely. The error handler only gets called with Error instances, and its return value is used as the final error value. If the error handler throws an error, that error replaces the original error. The error handler is usually called only once per error. However, in corner cases related to constructors, the error handler can be called multiple times for a single error value.

An error handler should avoid overwriting any properties already present in an object, as that would be quite confusing for other code. In general, an error handler should always avoid throwing an error, as that error replaces the original error and would also be confusing. As a specific example, an error handler must not try to add a new property to a non-extensible object, as that would cause a TypeError.

Below is an example error handler for adding a creation timestamp to errors at their creation:

Duktape.errCreate = function (e) {
    if (!(e instanceof Error)) {
        // this check is not really needed because errCreate only gets
        // called with Error instances
        return e;
    }
    if ('created' in e) {
        // already augmented or conflicting property present
        return e;
    }
    if (!Object.isExtensible(e)) {
        // object not extensible, don't try to add a new property
        return e;
    }
    e.created = new Date();
    return e;
}
To remove the handler, delete the property (setting it to e.g. null does not work and causes a TypeError when Duktape attempts to call the null value):

// Remove error handler for error creation
delete Duktape.errCreate;
Similarly, if Duktape.errThrow has been set, it is called right before an error is thrown, and can process or replace the error value. Because Ecmascript allows any value type to be thrown, the error handler may get called with arbitrary input values (not just Error instances). It may also be called more than once for the same value because an error can be re-thrown multiple times.

For example, to add a throw timestamp (recording the first time the object has been thrown) to errors:

Duktape.errThrow = function (e) {
    if (!(e instanceof Error)) {
        // refuse to touch anything but Error instances
        return e;
    }
    if ('thrown' in e) {
        // already augmented or conflicting property present
        return e;
    }
    if (!Object.isExtensible(e)) {
        // object not extensible, don't try to add a new property
        return e;
    }
    e.thrown = new Date();
    return e;
}
Again, to remove the handler, delete the property:

// Remove error handler for error throwing
delete Duktape.errThrow;
Current limitations §

There is no cause chain support. Cause chains would be useful but there are no cause chains in Ecmascript, nor does there seem to be a de facto standard for them.
These is currently no way to access traceback elements programmatically.
If an error is created with a non-constructor function call to a custom error class (MyError('msg') instead of new MyError('msg')) it won't get augmented with custom fields such as traceback data. When called as a constructor custom errors inheriting from Error get augmented normally. Built-in standard errors (like TypeError) always get augmented, even when created with a non-constructor function call (the tracebacks look slightly different depending on how the error is created, though).
Function objects §

Ecmascript functions §

Duktape Function objects add a few properties to standard Ecmascript properties. The table below summarizes properties assigned to newly created function instances (properties can of course be added or removed afterwards):

Property name	Compatibility	Description
length	standard	Function (nominal) argument count (if relevant). Present for all Function objects, including bound functions.
prototype	standard	Prototype used for new objects when called as a constructor. Present for most constructable Function objects, not copied to bound functions.
caller	standard	Accessor which throws an error. Present for strict functions and bound functions. Not copied to bound functions. (If DUK_USE_NONSTD_FUNC_CALLER_PROPERTY is given, non-strict functions will get a non-standard caller property.)
arguments	standard	Accessor which throws an error. Present for strict functions and bound functions. Not copied to bound functions.
name	Duktape	Function name, see below. Bound function name is based on this property, with a "bound " prefix (standard ES2015 behavior).
fileName	Duktape	Filename or context where function was declared (same name as in error tracebacks). Copied to bound function from target function.
callee	n/a	Never assigned by default (listed here to clarify relationship to "caller" property).
The name property is assigned to all functions and is also the name used in tracebacks. It is assigned as follows:

function funcDecl() {
    /* Function declaration: 'name' is declaration name, here 'funcDecl'. */
}

var foo = function namedFunc() {
    /* Named function expression: 'name' is the name used in expression,
     * here 'namedFunc' (not 'foo').
     */
}

var bar = function () {
    /* Anonymous function expression: 'name' is the empty string. */
}
Several Ecmascript built-in functions have properties different from user created Functions.
Duktape/C functions §

User-created Duktape/C functions (duk_push_c_function()) have a different set of properties to reduce Function object memory footprint:

Property name	Compatibility	Description
length	standard	Function argument count, matches argument to duk_push_c_function(), 0 for varargs. Non-writable and non-configurable.
Note in particular that the standard prototype, caller, and arguments properties are missing by default. This is not strictly compliant but is important to reduce function footprint. User code can of course assign these properties but is not required to do so.

There's also no (non-standard) name property. Setting it manually is useful because it affects how a function appears in tracebacks.

Lightweight Duktape/C functions §

Lightweight Duktape/C functions (lightfuncs) are a very memory efficient way of representing a native function in the Ecmascript environment. Lightfuncs don't have a property table so they can't hold properties. However, they inherit from Function.prototype and have the following virtual properties (which are non-configurable and non-writable):

Property name	Compatibility	Description
length	standard	Function (nominal) argument count.
name	Duktape	Function name: "light_<PTR>_<FLAGS>".
The name property is an automatically generated virtual function name. <PTR> is a platform dependent dump of the Duktape/C function pointer, and <FLAGS> is a raw hex dump of the 16-bit internal control fields (the format is Duktape internal). You shouldn't rely on a specific format. For example:

duk> print(myLightFunc.name);
light_0805b94c_0511
As for ordinary functions, a lightfunc coerces to an implementation dependent string. You shouldn't rely on a specific format. For example:

duk> print(myLightFunc);
function light_0805b94c_0511() {"light"}
For more details, see:

How to work with lightfuncs
Type algorithms
lightweight-functions.rst
Date and time §

Date and time handling is often a portability issue when adapting Duktape to more exotic environments. The duk_config.h configuration header selects platform specific providers needed to implement the Date built-in. Duktape has built-in providers for mainstream platforms (Linux, Windows, OS X) which should usually work without changes. You can also write an external Date provider for more exotic environments. An external Date provider can also be used e.g. when a time offset needs to be applied to the platform time, or when using time virtualization.

Ecmascript code interacts with date/time through the standard Date built-in which is, by specification, limited to millisecond resolution. There are currently no Duktape specific Ecmascript date/time APIs. (A custom API may be added later to deal with sub-millisecond resolution.)

C code can of course use platform date/time APIs directly, but the Duktape C API also provides date/time API calls. These calls see the same time values as Ecmascript code which may matter when e.g. time virtualization is used. Using these calls makes your code platform neutral and thus more portable. The Duktape C API allows sub-millisecond resolution for time values. See How to work with time values for more details.

Random numbers §

Random numbers are used internally for Math.random(). They are also used currently for random pivot selection in the Array.prototype.sort() implementation.

The default internal random number generator is xoroshiro128+ with SplitMix64 seed mixing. Shamir's three-op PRNG is used on low memory targets and when the compiler doesn't have 64-bit types. The generators are not suitable for serious statistics algorithms due to e.g. limited quality of the seed material, and are not at all suitable for cryptography.

You can replace the internal random number generator using the DUK_USE_GET_RANDOM_DOUBLE config option.

Debugger §

Duktape has built-in debugger support as an option you can enable during compilation. Debugger support adds about 15-20kB of code footprint (depending on what debugger features are enabled) and has very minimal memory footprint. Debugger features include:

Execution status information such running/paused at file/line, callstack, local variables for different callstack levels
Execution control including pause/resume, step over/into/out, breakpoints targeted at file/line, debugger statement
Generic Eval at any callstack level, get/put variable at any callstack level
A mechanism for application-defined requests (AppRequest) and notifications (AppNotify)
Heap object in-depth inspection, Duktape heap walking, and getting a full heap dump
The debugger is based on the following main concepts:

Duktape provides a built-in debug protocol which is the same for all applications. The application doesn't need to parse or understand the debug protocol. The debug protocol is a compact binary protocol so that it works well on low memory targets with low speed connectivity. There is a JSON mapping for the debug protocol and a JSON debug proxy to make it easier to integrate a debug client.
The debug protocol runs over a reliable, stream-based debug transport. To maximize portability, the concrete transport is provided by application code as a set of callbacks implementing a stream interface. A streamed transport allows unbuffered streaming of debug messages, which keeps memory usage very low.
A debug client terminates the transport connection and uses the Duktape debug protocol to interact with Duktape internals: pause/resume, stepping, breakpoints, eval, etc. You can also use the JSON debug proxy for easier integration.
A very narrow debug API is used by the application code to attach and detach a debugger, and to provide the callbacks needed to implement the debug transport. All other debug activity happens through the debug protocol which is implemented by Duktape directly with no application involvement.
The most appropriate debug transport varies a lot between debug targets; it can be Wi-Fi, Bluetooth, a serial line, a stream embedded into a custom management protocol, etc. Although there is no "standard" transport, a TCP connection is a useful default. The Duktape distributable includes all the pieces you need to get started with debugging using a TCP transport:

An example implementation of the callbacks needed for a TCP transport: duk_trans_socket_unix.c (there's also a Windows example)
Debugger support for the Duktape command line tool (duk) using the TCP transport: --debugger option
A debugger web UI based on Node.js, Express, and socket.io: duk_debug.js
The Node.js based debugger web UI (duk_debug.js) can connect to the Duktape command line, but can also talk directly with any other target implementing a TCP transport. You can also customize it to use a different transport or use a proxy which converts between TCP and your custom transport. It's also possible to write your own debug client from scratch and e.g. integrate it to a custom IDE. You can integrate directly with a debug target using the binary debug protocol, or use the JSON proxy provided by duk_debug.js (Node.js) or duk_debug_proxy.js (DukLuv).

Debug targets and debug clients are intended to be mixed and matched: apart from the transport (which is usually either TCP or easy to adapt) the debug protocol is the same. Core functionality will be the same regardless of the debug client or the debug target, but some optional features may be missing. Debug clients and debug targets may also implement application specific commands (AppRequest) and notifications (AppNotify) for richer integration which can be used when both the client and the target support them (they're easy and safe to ignore if not supported). Custom commands and notifications allow e.g. downloading of source files directly from the target, deep inspection of the state of a custom memory allocator, rebooting the target on command, etc.

For more details on the implementation and how to get started, see:

debugger/README.rst
debugger.rst
duk_trans_dvalue.c: example debug transport with local debug protocol decoding/encoding
duk_debug_proxy.js
Modules §

CommonJS module loading frameworks §

There's no built-in module loading framework because it's difficult for a single framework to match a wide variety of different module loading use cases. The Duktape distributable includes several optional module loader frameworks, for example:

module-duktape	A Duktape 1.x compatible loader based on CommonJS modules version 1.1.1, with additional support for module.exports and a few Duktape specific module object properties. The internals are documented in modules.rst, see How to use modules for examples. This loader was a built-in in Duktape 1.x but was moved into an optional extra in Duktape 2.x.
module-node	A Node.js modules compatible loader. See How to use Node.js-like modules for examples.
You can also implement your own module loader from scratch: the above loaders are implemented using the Duktape public API with no special access to internals.

The module loaders provide a require() function which allows modules to be loaded as follows:

var mod = require('foo/bar');
mod.hello();
The loaders abstract actual module resolution/loading to user-provided hook(s) to allow the loader to be embedded in wide variety of environments. For example:

Module loading can be used in environments with no file system by loading modules from a set of virtual built-in files.
On-the-fly transpiling can be done during loading, for example using Babel.
ES2015 modules §

There's currently no support for ES2015 import/export and ES2015 modules.

C module convention §

There's a recommended (but not mandatory) C module convention which allows C modules to be loaded and initialized from DLLs: c-module-convention.rst.

Logging §

The Duktape distributable includes a logging framework with a small footprint, reasonable performance, and redirectable output. The framework was a built-in in Duktape 1.x but was moved into an optional extra in Duktape 2.x.

Basic usage example:

var val1 = 'foo';
var val2 = 123;
var val3 = new Date(123456789e3);

var logger = new Duktape.Logger();  // or new Duktape.Logger('logger name')
logger.info('three values:', val1, val2, val3);
The example would print something like the following to stderr:

2014-10-17T19:26:42.141Z INF test.js: three values: foo 123 1973-11-29 23:33:09.000+02:00
See the Wiki article How to use logging and logging.rst for more details.

Finalization §

Overview §

Duktape supports object finalization as a custom feature. A finalizer is called when an object is about to be freed, so that application code can e.g. free native resources associated with the object. The finalizer can be either an Ecmascript function or a Duktape/C function. However, Ecmascript finalizers may interact badly with script timeouts, see below.

See How to use finalization for examples.

Getting and setting the current finalizer §

An object which has an internal _Finalizer property in its prototype chain (or in the object itself) is subject to finalization before being freed. The internal property should not be accessed directly, but can be read/written using the following:

Duktape.fin(obj) (Ecmascript) or duk_get_finalizer() (C) gets the current finalizer.
Duktape.fin(obj, fn) (Ecmascript) or duk_set_finalizer() (C) sets the current finalizer.
Finalizer function arguments and return value §

The finalizer function is called with two arguments:

The object being finalized.
A boolean flag indicating if the object is being forcibly freed as part of heap destruction. This argument was added in Duktape 1.4.0:
If false (normal case), the finalizer may rescue the object by creating a live reference to the object before returning and the finalizer is guaranteed to be called again later (heap destruction at the latest).
If true (forced finalization in heap destruction), the object cannot be rescued and will be forcibly freed after the finalizer finishes. Native resources should be freed without expecting any further calls into the finalizer.
The return value of a finalizer is ignored. Any errors thrown by the finalizer are also silently ignored.

Finalizer execution guarantees §

The main finalizer guarantees are:

Finalizers are executed for unreachable objects detected by reference counting or mark-and-sweep. The finalizer may not execute immediately, however, not even when reference counting detects that the object became unreachable.
Finalizers are also executed for all remaining objects, regardless of their reachability status, when a Duktape heap is destroyed.
A finalizer is called exactly once, at the latest when the heap is destroyed, unless the object is rescued by making it reachable again. An object may be rescued by its own finalizer, or by another object's finalizer when mark-and-sweep finalizes a set of objects. For example, if X.ref = Y, and both X and Y become unreachable, it's possible for Y's finalizer to run, and later on X's finalizer to rescue both X and Y.
An object may be rescued an arbitrary number of times; the finalizer is called exactly once for each "rescue cycle". Even with this guarantee in place, it's best practice for a finalizer to be re-entrant and carefully avoid e.g. freeing a native resource multiple times if re-entered.
A finalizer is not executed for a Proxy object, but is executed for the plain target object. This ensures that a finalizer isn't executed multiple times when Proxy objects are created.
Together these guarantee that a finalizer gets executed at some point before a heap is destroyed, which allows native resources (such as sockets and files) to be freed reliably. There are a few exceptions to this guarantee, see below for more discussion:

Heap destruction finalizer sanity limit may cause a finalizer not to be executed.
When a script timeout is being propagated out of the current callstack, Ecmascript finalizers will immediately rethrow the script timeout error. Duktape/C finalizers will execute normally.
If Duktape runs out of memory (despite emergency GC) when trying to call a finalizer, the call error is silently ignored and the finalizer will be skipped.
When an object is finalized by mark-and-sweep but becomes unreachable before the next mark-and-sweep round has a change to detect the rescue, the object's finalizer will not be executed.
When the Duktape heap is being destroyed there are a few limitations for finalizer behavior:

Finalizers are executed for all finalizable objects in the heap, including reachable objects.
Finalizers cannot rescue objects; the semantics for a "rescue" would be ambiguous. The finalizer's second argument is true when called during heap destruction to indicate rescue is not possible.
A finalizer can create new finalizable objects and these objects will also be finalized. For example, a finalizer may post a HTTP notification of an object destruction which may use native network resources with their own finalizers. However, there's a sanity limit to this process to ensure runaway finalizers cannot prevent a heap from being destroyed.
The finalizer sanity algorithm is version specific, see How to use finalization. The algorithm allows the number of finalizable objects to grow initially, but it must decrease in a reasonable time or the finalization process is aborted, which may cause some native resource leaks.
Other current limitations §

When script execution timeout (DUK_USE_EXEC_TIMEOUT_CHECK) is used and a timeout occurs, it's possible for an Ecmascript finalizer to start running but immediately fail due to a script timeout. If this is a concrete concern, use a Duktape/C native finalizer instead which will run normally even when propagating a timeout.
The context (Duktape thread) executing the finalizer can currently be any coroutine in the heap. This must be taken into account in sandboxing.
Finalizers cannot currently yield.
Coroutines §

Duktape has a support for simple coroutines. Execution is strictly nesting: coroutine A resumes or initiates coroutine B, coroutine B runs until it yields or finishes (either successfully or through an uncaught error), after which coroutine A continues execution with the yield result.

Coroutines are created with new Duktape.Thread(), which gets as its sole argument the initial function where the new coroutine begins execution on its first resume. The resume argument becomes the initial function's first (and only) argument value.

A coroutine is resumed using Duktape.Thread.resume() which takes the following arguments: the coroutine to resume, the resume value, and (optionally) a flag indicating whether the resume value is an ordinary value or an error to be injected into the target coroutine. Injecting an error means that the resume value will be "thrown" at the site of the target coroutine's last yield operation. In other words, instead of returning with an ordinary value, the yield will seemingly throw an error.

A coroutine yields its current execution using Duktape.Thread.yield() which takes as its arguments: the value to yield, and (optionally) a flag indicating whether the yield value is an ordinary value or an error to be thrown in the context of the resuming coroutine. In other words, an error value causes the resume operation to seemingly throw an error instead of returning an ordinary value.

If a coroutine exists successfully, i.e. the initial function finishes by returning a value, it is handled similarly to a yield with the return value. If a coroutine exists because of an uncaught error, it is handled similarly to a yield with the error: the resume operation will rethrow that error in the resuming coroutine's context. In either case the coroutine which has finished can no longer be resumed; attempt to do so will cause a TypeError.

There are currently strict limitations on when a yield is possible. In short, a coroutine can only yield if its entire active call stack consists of plain Ecmascript-to-Ecmascript calls. The following prevent a yield if they are present anywhere in the yielding coroutine's call stack:

a Duktape/C function call
a constructor call
a getter/setter call
a proxy trap call
an eval() call
Function.prototype.call() or Function.prototype.apply()
a finalizer call
See How to use coroutines for examples.

Virtual properties §

Duktape provides two mechanisms for interacting with property accesses programmatically:

Accessor properties (setters and getters)
The Proxy object
See How to use virtual properties for examples.

Symbols §

Duktape supports ES2015 Symbols and also provides a Duktape specific hidden Symbol variant similar to internal strings in Duktape 1.x. Hidden Symbols differ from ES2015 Symbols in that they're hidden from ordinary Ecmascript code: they can't be created from Ecmascript code, won't be enumerated or JSON-serialized, and won't be returned even from Object.getOwnPropertyNames(). Properties with hidden Symbol keys can only be accessed by a direct property read/write when holding a reference to a hidden Symbol.

Duktape uses hidden Symbols for various implementation specific purposes, such as storing an object's finalizer reference. User code can also use hidden Symbols for its own purposes, e.g. to store hidden state in objects. User code should never try to access Duktape's hidden Symbol keyed properties: the set of such properties can change arbitrarily between versions.

Symbols of all kinds are represented internally using byte sequences which are invalid UTF-8; see symbols.rst for the current formats in use. When C code pushes a string using e.g. duk_push_string() and the byte sequence matches an internal Symbol format, the string value is automatically interpreted as a Symbol.

Note that the internal UTF-8 byte sequences cannot be created from Ecmascript code as a valid Ecmascript string. For example, a hidden Symbol might be represented using \xFFxyz, i.e. the byte sequence ff 78 79 7a, while the Ecmascript string "\u00ffxyz" would be represented as the CESU-8 bytes c3 bf 78 79 7a in memory.
Creating a Symbol is straightforward from C code:

/* Create a hidden Symbol which can then be used to read/write properties.
 * The Symbol can be passed on to Ecmascript code like any other string or
 * Symbol.  Terminating a string literal after a hex escape is safest to
 * avoid some ambiguous cases like "\xffab".
 */
duk_push_string(ctx, "\xff" "mySymbol");
For more discussion on C string hex escaping, see c_hex_esc.c.

Hidden Symbols cannot be created from Ecmascript code using the default built-ins alone. Standard ES2015 Symbols can be created using the Symbol built-in, e.g. as Symbol.for('foo'). When sandboxing, ensure that application C bindings don't accidentally provide a mechanism to create hidden Symbols by e.g. converting an input buffer as-is to a string without applying an encoding.

There's currently no special access control for properties with hidden Symbol keys: if user code has access to the Symbol, it can read/write the property value. This will most likely change in future major versions so that Ecmascript code cannot access a property with a hidden Symbol key, even when holding a reference to the hidden Symbol value.

Bytecode dump/load §

The API calls duk_dump_function() and duk_load_function() allow calling C code to (1) serialize an Ecmascript function into a portable bytecode and then (2) load the bytecode to reconstitute the function.

The bytecode format is Duktape version specific and it's unsafe to load bytecode from a different Duktape minor version (patch version doesn't affect bytecode compatibility). Duktape configuration options may also affect bytecode compatibility, so dump/load source and target must be compiled with the same options. The bytecode format is platform neutral so that you can compile bytecode on one platform and then load it on another, which is useful for cross-platform builds. Duktape does not validate loaded bytecode so calling code must ensure bytecode is intact and valid for the running Duktape version.

Calling code must ensure that bytecode being loaded is intact and valid for the running Duktape version (i.e. it has been compiled with the same version of Duktape and hasn't been modified since). Loading invalid bytecode may lead to memory unsafe behavior; loading maliciously crafted bytecode may even lead to exploitable vulnerabilities.
See bytecode.rst for more details.

Threading §

Duktape supports a limited form of multithreading:

A particular Duktape heap created with duk_create_heap() is single threaded: only one native thread can execute code in the heap at a time. The native thread can change over time, as long as two native threads are not active at the same time in the same Duktape heap.
It is possible to suspend Duktape execution in a Duktape/C call using duk_suspend() and later resume it with duk_resume(). Between these calls another thread may call into the same Duktape heap. Application code must manage any locking necessary to ensure only one native thread calls into Duktape at a time.
Duktape heaps are completely isolated from each other. Multiple native threads can execute code at the same time, as long as there is only one active native thread per Duktape heap.
The duk_suspend() and duk_resume() API calls (introduced in Duktape 1.6.0) allow a native thread to suspend while another native thread executes calls into a Duktape heap.
For some background, a Duktape heap is a single memory management region regardless of how many Duktape threads exist in the heap (don't confuse native threads and Duktape threads). Because the Duktape threads in a heap can share object references, multithreading support would need synchronization for garbage collection and all object handling. Synchronization would be a major portability issue, so a practical approach is to limit a Duktape heap to be single threaded. Duktape heaps don't share anything so there are no threading limitations between them as a general rule. However, when some platform features are not available (such as variadic preprocessor macros or re-entrant system calls) there are some limitations.

See threading.rst and How to use multiple native threads for a detailed discussion of threading limitations and best practices.

Sandboxing §

Sandboxed environments allow execution of untrusted code with two broad goals in mind:

Security: prevent untrusted code from doing unsafe operations such as accessing memory directly, causing segfaults, etc.
Availability: prevent untrusted code from hogging resources, e.g. consuming all available memory or entering an infinite loop.
Duktape provides mechanisms to achieve these goals for untrusted Ecmascript code. All C code is expected to be trusted. See sandboxing.rst for a detailed discussion of how to implement sandboxing.

Sandboxing support in Duktape 2.0 is still a work in progress.
Performance §

Duktape is an interpreted engine with currently no JIT support. It uses reference counting which makes memory usage tight at the cost of some execution performance. Overall Duktape performance should be similar to other interpreted languages.

See How to optimize performance for discussion of Duktape performance characteristics and hints to optimize code for performance.

Profile guided optimization (PGO) is strongly recommended if performance is a major concern. For example, GCC -O2 with PGO can be around 20% faster than GCC -O2 without PGO.

Memory usage §

Duktape allocates memory on demand and doesn't require a pre-allocated heap. When you create a heap on a 32-bit system, Duktape needs about 70kB for the built-in Ecmascript objects. With specific low memory options initial memory usage is about 27kB. This can be further reduced to about 3kB when moving built-in objects and strings to ROM (read-only data section). It's also possible to move custom native bindings fully into ROM.

After heap creation additional memory is then allocated as needed for executing application scripts. Reference counting ensures there is very little unused allocated memory, the only exception being objects which participate in reference loops; these are collected eventually by mark-and-sweep.

The memory allocations needed by Duktape fall into two basic categories. First, there are a lot of small allocations between roughly 16 to 128 bytes which are needed for strings, buffers, objects, object property tables, etc. Second, there are much fewer larger allocations needed for e.g. Ecmascript function bytecode, large strings and buffers, value stacks, the global string table, and the Duktape heap object.

For most systems memory usage or the memory allocation pattern is not an issue. On low memory environments, e.g. less than 1MB of system RAM, you may want to use a custom allocator to optimize memory usage. A pool-based allocator deals well with the small allocation churn without fragmentation issues. The downside is that you need to tune the memory pool sizes to match the concrete allocation patterns. You may want to use a pool allocator or a hybrid allocated if the platform allocation primitives perform poorly with a lot of small allocations.

See low memory options and low-memory.rst for more discussion on what low memory features exists and how to tune the memory pools for low memory systems.

With default options Duktape uses a 32-bit refcount field which may technically wrap on 64-bit systems with very large memory sizes. In practice this is unlikely to happen and requires the Duktape heap to be larger than 64GB. Disable DUK_USE_REFCOUNT32 to use size_t for refcount fields.

Compiling §

Overview §

There are two basic steps to compiling Duktape:

Configuration. Use tools/configure.py to prepare Duktape source and header files for compilation. The requested Duktape configuration is described using configure.py command line options, see Configuring Duktape for build. For example:
$ python2 tools/configure.py --output-directory /tmp/output -DDUK_USE_FASTINT -UDUK_USE_ES6_PROXY
$ ls /tmp/output/
duk_config.h  duk_source_meta.json  duktape.c  duktape.h
Compilation. Ensure that the generated header files are in the compiler include search path, and include the generate source file(s) in your application build; there's no official Duktape build script or Makefile. For example:
$ gcc -O2 -Wall -otest -I/tmp/output /tmp/output/duktape.c my_application.c -lm
While Duktape is usually compiled together with your application, you can also build it into a static or shared library. Duktape can also be installed as a system-wide library, see system-install.rst.

The DUK_OPT_xxx feature options are no longer supported in Duktape 2.x. All configuration information is embedded in duk_config.h and/or autogenerated sources and headers.
Configuring §

Preconfigured sources and default configuration §
The Duktape distributable contains preconfigured sources and headers with a few variants:

src/: a single source file version which consists of duk_config.h, duktape.h, and duktape.c.
src-noline/: same as src/ but with no #line directives in the combined source file; this matters in some environments, see https://github.com/svaarala/duktape/pull/363.
src-separate/: a separate source files version which consists of duk_config.h, duktape.h, and a set of separate source files. The single source file version is preferred, but separate files work better with some toolchains.
These preconfigured sources provide automatic platform, compiler, and architecture detection and use the Duktape default configuration:

Full Ecmascript E5/E5.1 compliance (including the optional Annex B features), except for intentional real world compatibility deviations (see Custom behavior)
ES2015 typed array and Node.js Buffer support
Some features from Ecmascript 2015 (E6) and Ecmascript 2016 (E7)
Packed value representation (8 bytes per value) when available, unpacked value representation (usually 16 bytes per value) when not
Reference counting and mark-and-sweep garbage collection
Full error messages and tracebacks
No debug printing, no asserts, etc
All portable fast paths enabled
The preconfigured sources cannot be used to build Duktape into a Windows DLL. Run configure.py with the --dll option to do that.
Running configure.py to customize Duktape configuration §
The configure.py utility prepares Duktape source and header files for a specific configuration described using command line options. For example, to prepare Duktape sources for a DLL build with fastint support enabled and Ecmascript 6 Proxy object support disabled:

# Default output format is single source file (--separate-sources for separate
# sources) and no #line directives (--line-directives to enable them).

$ python2 tools/configure.py \
      --output-directory /tmp/output \
      --dll \
      -DDUK_USE_FASTINT \
      -UDUK_USE_ES6_PROXY

# The output directory /tmp/output contains the header and source files to
# be included in your build.

$ ls /tmp/output
duk_config.h  duk_source_meta.json  duktape.c  duktape.h
Configuration options given to configure.py affect several different aspects of the prepared header and source files, for example:

Optional features in Duktape source code are enabled/disabled using config options (DUK_USE_xxx) which are provided by the duk_config.h configuration header. The configuration header also handles platform, architecture, and compiler detection, and all other aspects of platform portability.
Built-in objects and strings are read in from metadata files and bit-packed built-in initialization data is generated. It's also possible to supply custom metadata files to add custom built-in bindings or modify standard built-ins such as Math and JSON.
When using "ROM built-ins", built-in objects and strings are placed into the read-only code section to reduce RAM footprint. The necessary source code for built-in object/string initializers is autogenerated by configure.py. Also custom built-ins can be placed in the read-only code section.
Unicode data files (UnicodeData.txt and SpecialCasing.txt) are converted into bit-packed run-time Unicode tables. The Unicode data files can also be overridden e.g. to reduce Unicode table footprint.
The configure.py utility requires Python 2.x support. If your build environment doesn't support Python 2.x, you can run configure.py on a different platform and compile the resulting files in your build environment.

Even if the default options are OK, it's recommended that you run configure.py as part of your build instead of using the preconfigured sources. Custom options may be necessary on e.g. low memory platforms. See Configuring Duktape for build for more practical details.

Commonly needed configuration options §
Some commonly needed configuration options are:

DUK_USE_FATAL_HANDLER, strongly recommended. The built-in default fatal error handler will write a debug log message (but won't write anything to stdout to stderr), and will then call abort(). If that fails, it enters an infinite loop to ensure execution doesn't resume after a fatal error. This is usually not the best behavior for production applications which may already have better fatal error recovery mechanisms. To replace the default fatal handler, see How to handle fatal errors.
Long control transfer: setjmp/longjmp and C++ exceptions. By default Duktape uses setjmp() and longjmp() (or their variants) for internal long control transfers. If you're compiling with a C++ compiler you may want to use DUK_USE_CPP_EXCEPTIONS which causes Duktape to use C++ exceptions for long control transfers and allows scope-based resource management (automatic destructors, etc; sometimes referred to as RAII) in Duktape/C functions to work as expected.
Windows DLL build. Windows DLL build requires some declspec() declarations for DLL library symbols. Use the --dll option when running configure.py to enable them for Duktape symbols.
Forced byte order or alignment. If you're using Duktape on a platform where Duktape's automatic feature detection doesn't (yet) work, you may need to force a specific byte order or alignment requirements during the configuration step.
Date built-in on exotic platforms. When porting to new or exotic platforms the Duktape built-in Date support may not work on the platform. In such a case you can implement an external "Date provider" which allows you to provide the necessary date/time primitives without Duktape changes. See datetime.rst.
Memory management alternatives §
There are two supported memory management alternatives:

Reference counting and mark-and-sweep (default): heap objects are freed immediately when they become unreachable except for objects participating in unreachable reference cycles. Such objects are freed by a periodic voluntary, stop the world mark-and-sweep collection. Mark-and-sweep is also used as the emergency garbage collector if memory allocation fails.
Mark-and-sweep only: reduces code footprint and memory footprint (heap headers don't need to store a reference count), but there is more memory usage variance than in the default case. The frequency of voluntary, stop the world mark-and-sweep collections is also higher than in the default case where reference counting is expected to handle almost all memory management. Voluntary (non-emergency) mark-and-sweep can be disabled via config options.
Reference counting relies on mark-and-sweep to handle reference cycles. For example, every Ecmascript function instance is required to be in a reference loop with an automatic prototype object created for the function. You can break this loop manually if you wish. For internal technical reasons, named function expressions are also in a reference loop; this loop cannot be broken from user code and only mark-and-sweep can collect such functions.

Compiling §

General guidelines §
Duktape doesn't have an official Makefile or a build script: given the number of different portability targets, maintaining an official build script would be difficult. Instead, you should add Duktape to your existing build process in whatever way is most natural.

Duktape is compiled with a C or C++ compiler (C99 is recommended) and then linked to your program in some way; the exact details vary between platforms and toolchains. For example, you can:

Compile Duktape together with your program without an explicit linking step.
Compile Duktape as a static library, and link the static library with your program.
Compile Duktape as a dynamic library, and link the dynamic library with your program.
Compile Duktape as a dynamic library, install it system-wide, and use it from several applications; see system-install.rst.
All Duktape API functions are potentially macros, and the implementation of a certain API primitive may change between a macro and an actual function even between compatible releases. Some Duktape configuration options also affect binary compatibility. To ensure binary compatibility:

Include duktape.h in application code. This is good practice in general, but without the header your compiler will incorrectly assume that all Duktape API functions are actual functions which will cause linking to fail.
Use the same prepared Duktape sources and headers when compiling Duktape and your application. This ensures Duktape and your application are compiled with the exactly same Duktape version and configuration. This is especially important when Duktape is compiled as a library in a separate compilation step.
Use the same compiler when compiling Duktape and your application. Using a different compiler may affect e.g. type detection in the Duktape duk_config.h header or function calling convention, and thus compromise binary compatibility. In practice compilers can be mixed to some extent, e.g. GCC and Clang are generally compatible.
Recommended compiler options §
Recommended compiler options for GCC/clang, use similar options for your compiler:

-std=c99: recommended to ensure C99 semantics which improve C type detection and allows Duktape to use variadic macros.
-Wall: recommended to catch potential issues early.
-Os: optimize for smallest footprint, which is usually desired when embedding Duktape. -O2 is a good compromise for performance optimized builds.
-fomit-frame-pointer: omit frame pointer, further reduces footprint but may interfere with debugging (leave out from debug builds).
-fstrict-aliasing: use strict aliasing rules, Duktape is compatible with these and they improve the resulting C code.
Configure.py with --dll is needed when Duktape is built as a DLL, at least when compiling for Windows.
Compilation warnings §
Duktape usually compiles without warnings when using a mainstream compiler (e.g. GCC, Clang, MSVC, or MinGW) in C99 mode with warnings enabled (e.g. -Wall in gcc/clang), and using default Duktape configuration options. There may be some warnings when using a non-mainstream compiler, very strict warning levels (like -Wextra in gcc/clang or /W4 in MSVC), or non-default Duktape configuration options. Eliminating compilation warnings for all compilers and all configuration option combinations is very difficult and is thus explicitly not a project goal. You're still encouraged to report warnings so that they can be fixed if possible.

Using a C++ compiler §
Duktape works with both C and C++ compilers and applications. You can compile Duktape and the application with a C or a C++ compiler in any combination. Even so, it is recommended to compile both Duktape and the application with the same compiler (i.e. both with a C compiler or both with a C++ compiler) and with the same compiler options.

The duktape.h header contains the necessary glue to make all of these combinations work. Specifically, all symbols needed by Duktape public API are inside a extern "C" { ... } wrapper when compiled with a C++ compiler. This ensures that such symbols are defined and used without C++ name mangling. Specifically:

When compiling Duktape itself with a C++ compiler, symbols needed by Duktape public API are not mangled. Other Duktape internal symbols will be mangled, but are not externally visible and should thus cause no problems even if the application is compiled with a C compiler.
When compiling an application with a C++ compiler, the wrapper ensures that Duktape public API symbols used by the application are looked up without mangling.
If you mix C and C++ compilation, you should do the final linking with the C++ toolchain. At least when mixing gcc/g++ you may encounter something like:

$ g++ -c -o duktape.o -Isrc/ src/duktape.c
$ gcc -c -o duk_cmdline.o -Isrc/ examples/cmdline/duk_cmdline.c
$ gcc -o duk duktape.o duk_cmdline.o -lm
duktape.o:(.eh_frame+0x1ab): undefined reference to `__gxx_personality_v0'
collect2: error: ld returned 1 exit status
One fix is to use g++ for linking:

$ g++ -c -o duktape.o -Isrc/ src/duktape.c
$ gcc -c -o duk_cmdline.o -Isrc/ examples/cmdline/duk_cmdline.c
$ g++ -o duk duktape.o duk_cmdline.o -lm
Because duk_config.h selects C/C++ data types needed by Duktape and also does other feature detection, mixing C and C++ compilers could theoretically cause the C and C++ compilers to end up with different active features or data types. If that were to happen, Duktape and the application would be binary incompatible which would lead to very difficult to diagnose issues. This is usually not an issue, but to avoid the potential, compile Duktape and the application with the same compiler.

By default scope-based resource management (sometimes referred to as RAII) won't work in Duktape/C functions because Duktape uses longjmp() for internal long control transfers, bypassing C++ stack unwind mechanisms. You can use DUK_USE_CPP_EXCEPTIONS to cause Duktape to use C++ exceptions for internal long control transfers, which allows scope-based resource management to work in Duktape/C functions.

Portability §

Duktape is widely portable to platforms with at least a C89 compiler (with a few exceptions like requiring snprintf() and vsnprintf()). Because Duktape has very limited dependency on platform functions, it's possible to Duktape to very exotic platforms. One major platform dependency is the Date built-in which may need a custom provider for an exotic platform.

See Portability for platform and compiler specific portability issues, porting recommendations, and troubleshooting tips.

Compatibility §

This section discussed Duktape compatibility with Ecmascript dialects, extensions, frameworks, and test suites.

Ecmascript E5 / E5.1 §

The main compatibility goal of Duktape is to be Ecmascript E5/E5.1 compatible. However, ES5 feature semantics are updated to ES2015 (or later) where incompatible changes have been made in newer specification versions. Current level of compatibility should be quite high.

Ecmascript 2015 (E6) §

Duktape implements some features from Ecmascript 2015 (E6), but generally there is no compatibility with E6 yet.

Ecmascript 2016 (E7) §

Duktape implements some features from Ecmascript 2016 (E7), but generally there is no compatibility with E7 yet.

Ecmascript E3 §

There is no effort to maintain Ecmascript E3 compatibility, other than required by the E5/E5.1 specification.

CoffeeScript §

CoffeeScript compiles to JavaScript which should be compatible with Duktape. There are no known compatibility issues.

Some CoffeeScript examples are included in the distributable. Simply run make in examples/coffee/. For instance, hello.coffee:

print 'Hello world!'
print 'version: ' + Duktape.version
compiles to:

(function() {

  print('Hello world!');

  print('version: ' + Duktape.version);

}).call(this);
Coco §

Like CoffeeScript, Coco compiles to Javascript. There are no known issues.

LiveScript §

Like CoffeeScript, LiveScript compiles to Javascript. There are no known issues.

TypeScript §

TypeScript compiles to Javascript. There are no known issues with compiling TypeScript using the Microsoft TypeScript compiler (in the ES5/CommonJS mode) and running the resulting Javascript using Duktape. It's also possible to run the TypeScript compiler with Duktape.

Underscore.js §

Underscore.js provides a lot of useful utilities to plain Ecmascript. Duktape passes almost all of Underscore's test cases, see underscore-status.rst for current compatibility status.

Test262 §

test262 is a test suite for testing E5.1 compatibility, although it includes also tests outside of standard E5.1. Duktape passes almost all of test262 cases, see test262-status.rst for current compatibility status.

Asm.js §

asm.js is a "strict subset of JavaScript that can be used as a low-level, efficient target language for compilers". As a subset of JavaScript, functions using asm.js type annotations should be fully compatible with Duktape. However, Duktape has no specific support for asm.js and won't optimize asm.js code. In fact, asm.js code will generate unnecessary bytecode and execute slower than normal Ecmascript code. The "use asm" directive specified by asm.js is ignored by Duktape. Also, because there is not typed array support yet, no "heap object" can be provided.

Emscripten §

Emscripten compiles C/C++ into Javascript. Duktape is currently (as of Duktape 1.5.0) Emscripten compatible and supports ES2015 TypedArray which allows Emscripten fastcomp to be used.

Large programs may fail due to Duktape compiler running out of virtual registers, and performance is somewhat limited as Duktape is an interpreted engine. See emscripten-status.rst for current compatibility status.

Because Duktape itself compiles with Emscripten, it is possible to run Duktape inside a web page for instance, see Dukweb REPL.

Lua.js §

lua.js translates Lua code to Javascript. There are no known issues in running the generated Javascript, except that Duktape doesn't provide console.log which lua.js expects. This is easy to remedy, e.g. by prepending the following:

console = { log: function() { print(Array.prototype.join.call(arguments, ' ')); } };
JS-Interpreter §

JS-Interpreter interprets Javascript in Javascript. JS-Interpreter works with Duktape, except that Duktape doesn't provide window which JS-Interpreter expects. This can be fixed by prepending:

window = {};
Versioning §

Semantic versioning §

Duktape follows Semantic Versioning:

Major version changes when API incompatible changes are made.
Minor version changes when backwards-compatible functional changes are made.
Patch version changes when backwards-compatible bug fixes are made.
The "public API" to which these rules apply include:

The Duktape API calls documented on duktape.org; except those tagged experimental.
The global environment visible to Ecmascript code, including the Duktape object and other Ecmascript extensions, as documented on duktape.org; except changes needed to align with latest Ecmascript specifications.
The following are not part of the "public API" versioning guarantees:

Duktape API calls tagged experimental.
Internal calls made by API macros. While API calls implemented as macros are part of the public API, any internal calls the macros make are not, even if their symbol visibility is public.
Changing an API call from a function call to a macro (or vice versa). These are considered compatible changes (but are not done in patch releases).
Aligning with latest Ecmascript specifications. Duktape tracks the latest Ecmascript specification (currently ES2016). Backwards incompatible changes required to align with the latest specifications may be done in minor versions too (but not in patch versions unless necessary to fix a bug). Typically such changes are relatively minor, for example argument coercion or property inheritance changes.
Specific behavior which is explicitly noted to potentially change even in minor versions, for example:
Behavior of buffer objects when their backing buffer is smaller than the apparent size of the buffer object. Memory safe behavior is guaranteed, but otherwise behavior may vary between versions.
Duktape config options. Incompatible config option changes are not made in patch releases, but can be made in minor releases. The goal is to cause a compile error (if possible) when a no-longer-supported feature option is used so that any incorrect assumptions can be fixed.
Extras distributed with Duktape (extras/ directory).
When a patch version is released, the following things are guaranteed:

API binary compatibility is maintained: constant values don't change, function typing doesn't change, API call function/macro status doesn't change.
Bytecode dump/load format doesn't change so that you can load bytecode dumped from an older version which only differs in patch version.
Ecmascript semantics fixes are not included unless necessary to fix a bug.
Config options won't change in an incompatible manner.
Experimental features §

Some new features and API calls are marked experimental which means that they may change in an incompatible way even in a minor release.

Features may be marked experimental e.g. because they are useful but incomplete, or because the best design is not obvious and it's useful to gather some feedback before committing to the design. Typically a feature is experimental for one minor release and then, after the necessary changes, becomes a fully supported feature.

Version naming §

Releases use the form (major).(minor).(patch), e.g. 1.0.3.

Development pre-releases use the form (major).(minor).(patch)-alpha.(number), e.g. 1.3.0-alpha.2. The form 0.(minor).0 was used for development pre-releases before the 1.0 release.

DUK_VERSION and Duktape.version §

DUK_VERSION and Duktape.version provide version identification using a single number computed as: (major * 10000 + minor * 100 + patch), then subtracting one for development pre-releases.

Note the limitations for development pre-releases:

Development pre-releases for the same release are not distinguished from one another: for example, both 1.3.0-alpha.1 and 1.3.0-alpha.2 are identified as 10299.
Development pre-releases for patch releases are not distinguished from the previous patch release: for example, 1.3.3-alpha.6 and 1.3.2 are both identified as 10302.
Development pre-releases shouldn't be used in production, but the current DUK_VERSION and Duktape.version number provides a useful approximation for version comparison: an alpha release will compare smaller than the actual release, but higher (or equal) than a previous release.

Examples §

The table below provides some examples, in ascending version order:

Version	Pre-release?	DUK_VERSION &
Duktape.version	Notes
0.12.0	yes	1200	Pre-release before 1.0 release.
1.0.0	no	10000	
1.3.0-alpha.1	yes	10299	Identified like 1.2.99, first 1.3.0 development pre-release.
1.3.0-alpha.2	yes	10299	Identified like 1.2.99, no difference to 1.3.0-alpha.1.
1.3.0	no	10300	
1.3.2	no	10302	
1.3.3-alpha.6	yes	10302	Identified like 1.3.2, no difference to 1.3.2 release.
1.3.3	no	10303	
2.0.0-alpha.3	yes	19999	Identified like 1.99.99.
2.0.0	no	20000	
Maintenance of stable versions §

There's no long term maintenance policy yet: stable versions will get bug fixes (patch releases) at least until the next stable version has been released, and there has been some time to migrate to it.

Incompatible changes §

The general goal for incompatible changes is that an application relying on old, unsupported features will fail to build. It is preferable to have the build fail rather than to be silently broken. This means for example that:

When API call semantics are changed, the old API call is removed (causing a build failure if used) and a new one is added.
When support for an old feature option is removed, an attempt to use it will cause a build failure.
This is not a hard rule, but the default guideline.

Limitations §

The following is a list of known limitations of the current implementation. Limitations include shortcomings from a semantics perspective, performance limitations, and implementation limits (which are inevitable).

Trivial bugs are not listed unless they are "long term bugs".

No re-entrancy §

A single Duktape heap, i.e. contexts sharing the same garbage collector, is not re-entrant. Only one C/C++ thread can call Duktape APIs at a time for a particular Duktape heap (although the calling thread can change over time). See Threading.

String and buffer limits §

The internal representation allows a maximum length of 2**31-1 (0x7fffffff) bytes (not characters) for strings. 16-bit codepoints encode into 3 bytes of UTF-8 in the worst case, so the maximum string length which is guaranteed to work is about 0.7G characters.

Buffer values are also limited to 2**31-1 (0x7fffffff) bytes.

Property limits §

An object can have at most DUK_HOBJECT_MAX_PROPERTIES (an internal define). Currently this limit is 0x7ffffffff.

Array limits §

When array item indices go over the 2**31-1 limit (0x7fffffff), Duktape has some known bugs with array semantics.

Regexp quantifier over empty match §

The regexp engine gets stuck when a quantifier is used over an empty match but eventually bails out with an internal recursion (or execution step) limit. For instance, the following should produce a "no match" result but hits an internal recursion limit instead:

$ duk
duk> t = /(x*)*/.exec('y');
RangeError: regexp executor recursion limit
    at [anon] (duk_regexp_executor.c:145) internal
    at exec () native strict preventsyield
    at global (input:1) preventsyield
Duktape does not fully support locales §

Although Duktape supports the concept of a local time, it doesn't support other locale related features such as: locale specific Date formatting, locale specific string comparison, locale/language specific Unicode rules (such as case conversion rules for Turkish, Azeri, and Lithuanian).

Unicode case conversion is not locale or context sensitive §

E5 Sections 15.5.4.16 to 15.5.4.19 require context and locale processing of Unicode SpecialCasing.txt. However, Duktape doesn't currently have a notion of "current locale".

Array performance when using non-default property attributes §

All array elements are expected to be writable, enumerable, and configurable (default property attributes for new properties). If this assumption is violated, even temporarily, the entire "array part" of an object is abandoned permanently and array entries are moved to the "entry part". This involves interning all used array indices as explicit string keys (e.g. "0", "1", etc). This is not a compliance concern, but degrades performance.

Array performance when writing elements using Object.defineProperty() §

When number indexed array elements are written with Object.defineProperty() the current implementation abandons the internal "array part" which makes later array access much slower. Write array elements with direct assignments such as a[123] = 321 to avoid this.

Global/eval code is slower than function code §

Bytecode generated for global and eval code cannot assign variables statically to registers, and will use explicit name-based variable read/write accesses. Bytecode generated for function code doesn't have this limitation; most variables are assigned statically to registers and direct register references are used used to access them.

This is a minor issue unless you spend a lot of time running top-level global/eval code. The workaround is simple: put code in a function which you call from the top level; for instance:

function main() {
    // ...
}
main();
There is also a common idiom of using an anonymous function for this purpose:

(function () {
    // ...
})();
Function temporaries may be live for garbage collection longer than expected §

Ecmascript functions are compiled into bytecode with a fixed set of registers. Some registers are reserved for arguments and variable bindings while others are used as temporaries. All registers are considered live from a garbage collection perspective, even temporary registers containing old values which the function actually cannot reference any more. Such temporaries are considered reachable until they are overwritten by the evaluation of another expression or until the function exits. Function exit is the only easily predicted condition to ensure garbage collection.

If you have a function which remains running for a very long time, it should contain the bare minimum of variables and temporaries that could remain live. For instance, you can structure code like:

function runOnce() {
    // run one iteration, lots of temporaries
}

function foreverLoop() {
    for (;;) {
        runOnce();
    }
}
This is typically not an issue if there are no long-running functions.

Function instances are garbage collected only by mark-and-sweep §

Every Ecmascript function instance is, by default, in a reference loop with an automatic prototype object created for the function. The function instance's prototype property points to the prototype object, and the prototype's constructor property points back to the function instance. Only mark-and-sweep is able to collect these reference loops at the moment. If you build with reference counting only (not recommended), function instances may appear to leak memory; the memory will be released when the relevant heap is destroyed.

You can break the reference loops manually (although this is a bit cumbersome):

var f = function() { };
var g = function() { };
var h = function() { };
Duktape.fin(f, function() { print('finalizer for f'); });
Duktape.fin(g, function() { print('finalizer for g'); });
Duktape.fin(h, function() { print('finalizer for h'); });

// not collected until heap destruction in a reference counting only build
f = null;            // not collected immediately

// break cycle by deleting 'prototype' reference (alternative 1)
g.prototype = null;
g = null;            // collected immediately, finalizer runs

// break cycle by deleting 'constructor' reference (alternative 2)
h.prototype.constructor = null;
h = null;            // collected immediately, finalizer runs

// mark-and-sweep triggers finalizer for 'f'
Duktape.gc();
For internal technical reasons, named function expressions are also in a reference loop with an internal environment record object. This loop cannot be broken from user code and only mark-and-sweep can collect such functions. Ordinary function declarations and anonymous functions don't have this limitation. Example:

var fn = function myfunc() {
    // myfunc is in reference loop with an internal environment record,
    // and can only be collected with mark-and-sweep.
}
Since Duktape 2.x mark-and-sweep is always enabled so that objects participating in reference loops are eventually freed. You can disable periodic "voluntary" (non-emergency) mark-and-sweep via config options to reduce collection pauses in time sensitive environments.

Non-standard function 'caller' property limitations §

When DUK_USE_NONSTD_FUNC_CALLER_PROPERTY is given, Duktape updates the caller property of non-strict function instances similarly to e.g. V8 and Spidermonkey. There are a few limitations, though:

When a (non-strict) function is called from eval code, Duktape sets caller to null if the eval code is non-strict, and eval (reference to the eval built-in function) if the eval code is strict. This deviates from e.g. V8 behavior.
Coroutines and caller don't mix well: caller may be left in a non-null state even after coroutine call stacks have been fully unwound. Also, if a coroutine is garbage collected before its call stack is unwound, the caller property of functions in its call stack will not get updated now.
See the internal test-bi-function-nonstd-caller-prop.js test case for further details.

Garbage collection during debugger paused state §

When debugger support is activated, a debugger session is active, and Duktape is paused, there are a few current limitations:

Because garbage collection is disabled during the paused state, calls to Duktape.gc() and duk_gc() will be silently ignored.
Comparison to Lua §

Duktape borrows a lot from Lua conceptually. Below are a few notes on what's different in Duktape compared to Lua. This may be useful if you're already familiar with Lua.

Array and stack indices are zero-based §

All array and stack indices are zero-based, not one-based as in Lua. So, bottom of stack is 0, second element from bottom is 1, and top element is -1. Because 0 is no longer available to denote an invalid/non-existent element, the constant DUK_INVALID_INDEX is used instead in Duktape.

String indices are also zero-based, and slices are indicated with an inclusive start index and an exclusive end index (i.e. [start,end[). In Lua, slices are indicated with inclusive indices (i.e. [start,end]).

Object type represents functions and threads §

In Lua functions and threads are a separate type from objects. In Duktape the object type is used for plain objects, Ecmascript and native functions, and threads (coroutines). As a result, all of these have a mutable and extensible set of properties.

Lua userdata and lightuserdata §

The concept closest to Lua userdata is the Duktape buffer type, with the following differences:

Duktape buffers can be resizable, Lua userdata values cannot. If a Duktape buffer is resizable, its data pointer is no longer guaranteed to be stable.
Duktape buffers are raw byte arrays without any properties, Lua userdata objects can store an environment reference.
Lua lightuserdata and Duktape pointer are essentially the same.

If you need to associate properties with a Duktape buffer, you can use a buffer object instead (or create your own object and store the plain buffer as its property). You can then add a finalizer to the object to free any resources related to the buffer. This works reasonably well as long as nothing else holds a reference to the buffer. If this were the case, the buffer could get used after the object had already been finalized. To safeguard against this, the native C structure should have a flag indicating whether the data structure is open or closed. This is good practice anyway for robust native code.

Garbage collection §

Duktape has a combined reference counting and non-incremental mark-and-sweep garbage collector (mark-and-sweep is needed only for reference cycles). Collection pauses can be avoided by disabling voluntary mark-and-sweep passes (disable DUK_USE_VOLUNTARY_GC). Lua has an incremental collector with no pauses, but has no reference counting.

Duktape has an emergency garbage collector. Lua 5.2 has an emergency garbage collector while Lua 5.1 does not (there is an emergency GC patch though).

duk_safe_call() vs. lua_cpcall() §

duk_safe_call() is a protected C function call which operates in the existing value stack frame. The function call is not visible on the call stack all.

lua_cpcall() creates a new stack frame.

Bytecode dump/load §

Starting from Duktape 1.3 Duktape has a bytecode dump/load mechanism similar to Lua lua_dump(). See Bytecode dump/load.

Metatables §

There is no equivalent of Lua metatables in Ecmascript E5/E5.1, but Ecmascript ES2015 Proxy objects provide similar functionality. To allow property virtualization better than available in E5/E5.1, Duktape implements an ES2015 Proxy subset.

lua_next() vs. duk_next() §

lua_next() replaces the previous key and value with a new pair, while duk_next() does not; the caller needs to explicitly pop the key and/or value.

Raw accessors §

There is no equivalent to Lua raw table access functions like lua_rawget. One can use the following Ecmascript built-ins for a similar effect (though not with respect to performance): Object.getOwnPropertyDescriptor ( O, P ), Object.defineProperty ( O, P, Attributes ).

Coroutines §

There are no primitives for coroutine control in the Duktape API (Lua API has e.g. lua_resume). Coroutines can only be controlled using the functions exposed by the Duktape built-in. Further, Duktape has quite many coroutine yield restrictions now; for instance, coroutines cannot yield from inside constructor calls or getter/setter calls.

Multiple return values §

Lua supports multiple return values, Duktape (or Ecmascript) currently doesn't. This may change with Ecmascript ES2015, which has a syntax for multiple value returns. The Duktape/C API reserves return values above 1 so that they may be later used for multiple return values.

Weak references §

Lua supports weak references. Duktape currently doesn't.

Unicode §

Lua has no built-in Unicode support (strings are byte strings), while Duktape has support for 16-bit Unicode as part of Ecmascript compliance.

Streaming compilation §

Lua has a streaming compilation API which is good when code is read from the disk or perhaps decompressed on-the-fly. Duktape currently does not support streaming compilation because it needs multiple passes over the source code.


