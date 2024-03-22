# Module 6
## Reflection I
+ After Importing `use std::net::{TcpListener, TcpStream};`, we will bound it to a specified address in this case `127.0.0.1:7878` and unwrap it as listener. 
+ It then accept connections and process them serially with the for loop. 
+ We made a `handle_connection(mut stream: TcpStream)` function to process each connection.
+ the function works by buffering the stream.Then mapping it into a vector into appropiate format using the series of `.lines()`,`.map()`,`take_while`,`.collect()`.
> A BufReader<R> performs large, infrequent reads on the underlying Read and maintains an in-memory buffer of the results BufReader<R> can improve the speed of programs that make small and repeated read calls to the same file or network socket.
+ We then print out the result of the request.

## Reflection II

## Reflection III

## Reflection IV

## Reflection V