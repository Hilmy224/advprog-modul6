# Module 6
## Commit I Reflection Notes
+ After Importing `use std::net::{TcpListener, TcpStream};`, we will bound it to a specified address in this case `127.0.0.1:7878` and unwrap it as listener. 
+ It then accept connections and process them serially with the for loop. 
+ We made a `handle_connection(mut stream: TcpStream)` function to process each connection.
+ the function works by buffering the stream.Then mapping it into a vector into appropiate format using the series of `.lines()`,`.map()`,`take_while`,`.collect()`.
> A BufReader<R> performs large, infrequent reads on the underlying Read and maintains an in-memory buffer of the results BufReader<R> can improve the speed of programs that make small and repeated read calls to the same file or network socket.
+ We then print out the result of the request.

## Commit II Reflection Notes
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

## Commit III Reflection Notes
![image2](/assets/images/commit3failed.png)
+ How we differentiate between response is first modify our `buf_reader` to be reading the first line of the HTTP request instead of mapping the whole thing into a vector and saving it inside `request_line`.
```
let request_line = buf_reader.lines().next().unwrap().unwrap();
```
+ We the check the content of `request_line` wether if it equals the request line of a GET request to the / path. If it does, the if block returns the contents of our HTML file and If the request_line does not equal the GET request to the / path, it means we’ve received some other request.
```
if request_line == "GET / HTTP/1.1"
```
+ Thats when those other request will be handled in the else block. The response we will give out a status line with status code 404 and the reason phrase NOT FOUND. The content of the response will be the HTML in the file 404.html.
```
else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
```
+ After reviewing the code we can tell the code is a bit repetitive with both reading files and writing the contents of the files to the stream. We can make the code more readible and concise by modifying the if blocks to instead just check for the request line and assign (status_line, filename) appropiately without reading and writing the content files again:
```
//Refactored if else blocks
let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
```
## Commit IV Reflection Notes
+ The first change we did is transitioning from using if statements to `match` expressions now that we have three distinct cases to handle.
+ The first case handles the if block and the third case handles the else block of the previous code. On the otherhand, the second case it handles requests to /sleep, causing the server to pause for 10 seconds before rendering the HTML page.
```
...
"GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(10)); 
            ("HTTP/1.1 200 OK", "hello.html") 
        }
...
```
+ After running the program we can observe that the delay between requesting something in another browser window after `http://127.0.0.1:7878/sleep` will result on a 10 second delay as it waits for the /sleep request to complete its 10-second sleep.
> thread::sleep = Puts the current thread to sleep for at least the specified amount of time. (Dont forget to Import thread)
> time::Duration = A Duration type to represent a span of time, typically used for system timeouts.
## Commit V Reflection Notes
+ To implement an efficient system using multithreading, it's ideal to construct a ThreadPool that enables simultaneous handling of various requests. 
+ The next step involves creating Workers, tasked with receiving and executing specific jobs or tasks sent to each request. Smooth communication between the ThreadPool and each Worker necessitates setting up a messaging mechanism, where the ThreadPool can send signals via a sender to cloned receivers assigned to each Worker. 
+ This ensures that when the ThreadPool receives a request to execute a task, a signal is dispatched via the sender to the relevant Worker's receiver, which then processes the job.
+ Within each Worker, there's a single thread consistently awaiting incoming data. Upon receiving the data, the Worker locks the receiver to process it. 
+ Upon completion, the lock on the receiver is released, allowing other Workers to receive information and execute subsequent tasks.
