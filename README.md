# stdin & stdout, the right way

How do you read a file from stdin? If you thought,

```js
var contents = fs.readFileSync("/dev/stdin", "utf8");
```

you’d be wrong, because this only reads up to what Node thinks is the size of the file. So, if you redirect a file to your program (`cat file | program`), you’ll only read the first 65,536 bytes of your file.

OK, so how do you write a file to stdout? If you thought,

```js
fs.writeFileSync("/dev/stdout", contents, "utf8");
```

you’d also be wrong, because this tries to close stdout, and you get this error:

```
Error: UNKNOWN, unknown error
    at Object.fs.writeSync (fs.js:528:18)
    at Object.fs.writeFileSync (fs.js:975:21)
```

Shucks. So what should you do?

Well, you could use a different pattern for reading from stdin:

```js
var chunks = [];

process.stdin
    .on("data", function(chunk) { chunks.push(chunk); })
    .on("end", function() { console.log(chunks.join("").length); })
    .setEncoding("utf8");
```

But that’s a pain, since now your code has two different code paths for reading inputs, depending on whether you’re reading a real file or stdin.

And the code gets even more complex if you want to read that file synchronously.

## rw

The **rw** module fixes these problems. It provides an interface just like readFile, readFileSync, writeFile and writeFileSync, but these methods work the way you expect on stdin and stdout.

Like this:

```js
var contents = rw.readSync("/dev/stdin", "utf8");
```

And this:

```js
rw.writeSync("/dev/stdin", contents, "utf8");
```

Also, **rw** automatically squashes EPIPE errors, so you can pipe the output of your program to `head` and you won’t get a spurious stack trace.