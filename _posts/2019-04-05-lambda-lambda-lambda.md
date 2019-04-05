---
layout: post
title: Lambda Lambda Lambda
comments: true
tags: [cpp]
---

<!-- Explain what lambdas are -->
Technically, I am talking about unnamed functions. A concept which is ubiquitous in functional programming languages like Haskell or Lisp. There are many possible benefits toward this style of programming, but the reason I started loving lambdas is that it makes the code much more clear and concise.

So what does it look like in c++, and how does it work?

# The Simplest Lambda: \[](){}();
<!-- Lambdas look strange, historically use confusing operator -->
First, lets look at a normal function in c: 
```c
static int three = 100;
int my_special_function(int one, int two) {
    return one + two + three;
}
```
Note the different parts of the syntax. You've told the compiler:
- Return type: `int`
- Function name: `my_special_function`
- Parameter names and their types: `int one`, `int two`
- Function definition: ` return one + two + three;`
  
Lets attempt to write this in the syntax of a lambda in c++:
```c++
static int three = 100;
auto my_special_function = [](int one, int two){
    return one + two + three;
};
```
Both of these versions are invoked in the same way, `my_special_function(200, 300);` would return `600`.

So the () is for parameters, the {} is for the function definition, what is the [] for? Lets change three to not be static.
```c++
int three = 100;
auto my_special_function = [](int one, int two){
    return one + two + three;
};
```
This doesn't compile with error: `error C3493: 'three' cannot be implicitly captured because no default capture mode has been specified`. This happens because lambdas won't implicitly capture local variables, the compiler doesn't know if it should use a copy of that variable or try to mutate the original. All this means that in order to use outside variables in a lambda, you have to pass it by copy or reference in the capture list.

```c++
int three = 100;
auto my_special_function = [&three](int one, int two){
    return one + two + three;
};
```

A function which returns nothing, takes no parameters, and captures no variables looks like this: `[](){}`

# Why Use This When My Old Stuff Works Too
Lambdas are better than named functions in cases where I use higher order functions. The logic is much more local to where it is being called, and there is less boilerplate code. Therefore it is easier to read and understand, and more closely matches my intent. Lets look at two common examples:

## Algorithm predicates
Lets say I want to check if a string is a number. `"abc"` isn't a number, but `"123"` is; even though they are both strings. A naive solution to this problem might be to loop through the characters, checking if they are digits.

```c++
bool is_number(const std::string &s) {
    if(s.empty()) return false;
    for(char c : s) {
        if(!std::isdigit(c)){
            return false;
            continue;
        }
    }
    return true;
}
```

The for loop here can be replaced with a standard algorithm, I'll use find_if:

```c++
bool digit_checker(char c) {
    return !std::isdigit(c);
}

bool is_number(const std::string &s) {
    return !s.empty() && std::find_if(s.begin(), s.end(), digit_checker) == s.end();
}
```

If the algorithm checks the whole string and gets to the end without finding a character that isn't a digit, then the string is a number. But since this is the only place that we are going to use digit_checker, the code becomes more concise if we use a lambda:

```c++
bool is_number(const std::string &s) {
  return !s.empty() && std::find_if(s.begin(), s.end(), [](char c) { return !std::isdigit(c); }) == s.end();
}
```

Standard algorithms often take these kind of predicate functions, it makes the algorithms much more flexible as higher order functions. Passing lambdas to these algorithms is way easier and simple than writing named functions every time you want to invoke an algorithm.

## Asynchronous callbacks
An extremely common method of asynchronous programming is the usage of callbacks for sequencing operations. Very often, I need to *do_something* after I *do_something_else*. Most of these asynchronous functions operate as higher order functions that accept a completion handler as one of its parameters, like everything in the asio<sup>[1](#a1)</sup> library.

```c++
asio::read(stream, buffer, [](std::error_code error, std::size_t bytes) {
    if(!error)
        std::cout << (int) bytes_read << " bytes were read into the buffer\n";
});
```

The lambda that we gave to the read function is called after the read completes.


# Behind The Curtain
<!-- Lambdas under the hood are objects that overload the function operator () -->
The compiler is doing things behind the scenes when it sees a lambda, and we can approximate that with our own code and get equivalent results.

Consider this lambda:
```c++
int three = 100;
auto lambda = [&three](int one, int two){ return one + two + three; };
std::cout << lambda(1, 2); // Output: 103
```

Equivalent code with a struct:
```c++
struct __lambda {
    __lambda(int &val) {
        three = val;
    }
    int &three;

    int operator()(int one, int two) {
        return one + two + three;
    }
}
int three = 100;
auto lambda = __lambda(three);
std::cout << lambda(1, 2); // Output 103
```

The compiler output for these two different approaches will actually be nearly identical, because that is what the compiler is doing behinds the scenes for the lambda. This property that the lambda is an object is what allows it to be passed around to functions. Check out Jason Turner's video<sup>[2](#a2)</sup> about lambdas for more info.



<a name="a1">1</a>: [https://think-async.com/Asio/](https://think-async.com/Asio/) [↩](#asynchronous-callbacks)

<a name="a2">2</a>: [https://youtu.be/br4tez2G9eM](https://youtu.be/br4tez2G9eM) [↩](#behind-the-curtain)