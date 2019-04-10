---
layout: post
title: Setting up a REPL
comments: true
tags: [cpp, database]
---

One of the very common criticisms of c++ is that it is very complicated. I saw someone on a programming stream yesterday claim that they prefer raw c because it feels more *intuitive*. To me, this just seems out of touch with reality. Lets look at the simplest REPL, from part 1 of the cstack database from scratch series. I think by the end anyone would agree with me that the c++ version of this code is better in every way to its c counterpart.

# Read Evaluate Print Loop

What we are creating here, REPL for short, is a very common introductory tool for someone just getting used to working with a SQL database. It allows a beginner user to interact with the database line by line, discovering how SQL works through tinkering with the language. Many of these REPL shells are also used in scripting languages, and its the default mode that python starts in if you pass it no arguments.

In this part, we are just looking at the simplest REPL ever, which just prompts the user with some context in the terminal, and allows lines of input to be evaluated. The only command we are evaluating for now is just the SQL .exit metacommand. Any other commands passed to our program are unrecognized. Here is the c version:

```c
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct InputBuffer_t {
  char* buffer;
  size_t buffer_length;
  ssize_t input_length;
};
typedef struct InputBuffer_t InputBuffer;

InputBuffer* new_input_buffer() {
  InputBuffer* input_buffer = malloc(sizeof(InputBuffer));
  input_buffer->buffer = NULL;
  input_buffer->buffer_length = 0;
  input_buffer->input_length = 0;

  return input_buffer;
}

void print_prompt() { printf("db > "); }

void read_input(InputBuffer* input_buffer) {
  ssize_t bytes_read =
      getline(&(input_buffer->buffer), &(input_buffer->buffer_length), stdin);

  if (bytes_read <= 0) {
    printf("Error reading input\n");
    exit(EXIT_FAILURE);
  }

  // Ignore trailing newline
  input_buffer->input_length = bytes_read - 1;
  input_buffer->buffer[bytes_read - 1] = 0;
}

void close_input_buffer(InputBuffer* input_buffer) {
    free(input_buffer->buffer);
    free(input_buffer);
}

int main(int argc, char* argv[]) {
  InputBuffer* input_buffer = new_input_buffer();
  while (true) {
    print_prompt();
    read_input(input_buffer);

    if (strcmp(input_buffer->buffer, ".exit") == 0) {
      close_input_buffer(input_buffer);
      exit(EXIT_SUCCESS);
    } else {
      printf("Unrecognized command '%s'.\n", input_buffer->buffer);
    }
  }
}
```

Here's a whole mess of things that a modern Rust or C++ developer cringe at.
- Using malloc to allocate memory for an input buffer
- Using a strcmp free function to deep compare equality of raw buffers.
- Manually freeing the memory allocated for the input buffer after we're done.

This isn't fast, and it isn't intuitive. Its also 56 lines of code with a large amount of boilerplate code for handling memory allocation. 

Lets break this down into a much simpler and more intuitive c++ alternative step by step.

```c
//Replace this input buffer pointer with a std::string
InputBuffer* input_buffer = new_input_buffer();
```
```c
// Our single line prompt function doesn't need to call an external free function
// Get rid of the free function 
void print_prompt() { printf("db > "); }
```
```c
/// strcmp is unnecessary if we're using a built in RAII string type
if (strcmp(input_buffer->buffer, ".exit") == 0)
```
```c
// malloc and free are code smells. Like new and delete, they should not exist in modern c++ code.
void close_input_buffer(InputBuffer* input_buffer) {
    free(input_buffer->buffer);
    free(input_buffer);
}
```

Here is our nearly identical c++ version with those simplifications:
```c++
#include <iostream>
#include <string>

int main() {
  std::string input;
  while (true) {
    std::cout << "db > ";
    std::getline(std::cin, input);

    if(input == ".exit") {
      return EXIT_SUCCESS;
    } else {
      std::cout << "Unrecognized command: " << input << '\n';
    }
  }
}
```

Compare the two different versions' disassembly on a website like https://godbolt.org/ and we can see that the c++ version very similar and potentially faster than the raw c version.

|                    | c                                | c++                                             |
|--------------------|----------------------------------|-------------------------------------------------|
| Lines of code      |                               56 | 15                                              |
| Disassembly length | 99                               | 99                                              |
| Potential bugs     | Infinite chances of memory leaks | Impossible to leak with RAII types on the stack |

This very simple REPL for part 1 perfectly illustrates how insane people sound to me when they claim that c is simpler or more intuitive than c++. In this simple case, c++ is approximately as easy to write and understand as a high level scripting language like python, but compiles down to a smaller binary than c.

This post is part of a series which I am going to go through step by step simplifying and converting the cstack sqlite database into a more modern c++ version. So stay tuned for a better REPL parser implementation, database structure and persistence, and a look at btrees.