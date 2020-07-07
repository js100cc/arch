# Unix Standard Streams

Unix provide many abstract layer for IPC, an important one is standard streams.

## Redirection

By default, stdin is keyboard, stdout is monitor. With redirection we can
change this.

```shell
# normal
cat < ~/.vimrc
ls > /tmp/ls-output

# append
ls / >> /tmp/ls-output

## trick
>  filename		# create a new file
>> filename		# same as 'touch filename' in bash

## stderr
ajflasfa &> /tmp/output
# or
ajflasfa > /tmp/output 2>&1
```

## Pipeline

A process's stdout as another's stdin is called pipeline.

```shell
# hello.js
console.log("hello world!");
console.log("hello js!");

# pipeline
node hello.js | grep world | tr a-z A-Z		# HELLO WORLD!

# pipeline + redirection
node hello.js | grep world | tee /tmp/hello-world | tr a-z A-Z
# stdout: HELLO WORLD!
# /tmp/world: hello world
```

## Programming

For programming, K&R maybe is the best book for learning standard streams.

From The C Programming Language, K&R, 2nd:

>> The model of input and output supported by the standard library is very
>> simple. Text input or output, regardless of where it originates or where
>> it goes to, is dealt with as streams of characters. A text stream is a
>> sequence of characters divided into lines; each line consists of zero or
>> more characters followed by a newline character.

`getchar` is an API from `stdio` library. We can use it to implement many
prototype programs to do similar things like some classic Unix programs do.

You can see that in K&R, for simplicity, I use Javascript(Node.js) here:

```js
// cat.js
process.stdin.pipe(process.stdout);
```

A line is an unit handled by stdio，that's why we need manually type
<keyboard>Enter<keyboard> to get feedback, no matter we use `cat` or cat.js
here or implement it by `getchar/putchar` in C.

## Reference

1. [Wikipedia.org - Standard streams](https://en.wikipedia.org/wiki/Standard_streams)
2. The C programming language, K&R, 2nd
3. [Node.js](https://nodejs.org/)
