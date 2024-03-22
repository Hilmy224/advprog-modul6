# Module 6
## Reflection I
+ After Importing `use std::net::{TcpListener, TcpStream};`, we will bound it to a specified address in this case `127.0.0.1:7878` and unwrap it as listener. 
+ It then accept connections and process them serially with the for loop. 
+ We made a `handle_connection(mut stream: TcpStream)` function to process each connection.
+ the function works by buffering the stream.Then mapping it into a vector into appropiate format using the series of `.lines()`,`.map()`,`take_while`,`.collect()`.
> A BufReader<R> performs large, infrequent reads on the underlying Read and maintains an in-memory buffer of the results BufReader<R> can improve the speed of programs that make small and repeated read calls to the same file or network socket.
+ We then print out the result of the request.

## Reflection II
![image](/assets/images/commit2.png)
+ So we added it so we can display a simple html page that can be rendered by the browser
+ we do this first by making a a html file that we want to display, in this case `hello.html`
+ we then import `fs` so we can read and convert `hello.html` into `contents` as part of `response` that will be sent through the TCP stream.
```
let contents = fs::read_to_string("hello.html").unwrap(); 
```
+ Next we will make the othe companents components for the response that is  `status_line` and `Content_Length`(for content length we can just use `.length()` the content)
```
let status_line = "HTTP/1.1 200 OK"; 
```
+ After that we put together the response like so: 
```
let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
```
+ Then we convert the response string into a byte slice, and `write_all` writes the entire byte slice to the TCP stream. This sends the response back to the client
```
stream.write_all(response.as_bytes()).unwrap();
```

## Reflection III

## Reflection IV

## Reflection V