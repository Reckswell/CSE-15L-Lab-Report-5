# Lab Report 5: Debugging

## Part 1: Debugging Scenario

## Example Student Debugging Process

**What environment are you using (computer, operating system, web browser, terminal/editor, and so on)?**

I am using a Windows Desktop, Google Chrome as the web browser, Visual Studio Code as the editor, and bash/windows terminal for the terminals.

**Detail the symptom you're seeing. Be specific; include both what you're seeing and what you expected to see instead. Screenshots are great, copy-pasted terminal output is also great. Avoid saying “it doesn't work”.**

In accordance to my code, the website should iterate every time you put /iterate into the address bar, and should return a message in order of 1st, 2nd, and 3rd (I named it this way to mimic that of a terminal). If the path has no arguments, or anything besides /iterate, it should display a message which shows the user the total number of visits to the website, as well as suggesting the user types /iterate to get a different message. In the testServer.sh file, it refers to iterations as 0th, 1st, and 2nd, in reference to their results when using the mod indicator `%` (eg. `repeats % 3 == 0` in the if-statement).

However, whenever running my code, it seems to index the number of site visitors incorrectly, and as a result displays the output message for /iterate incorrectly. It should be in order of 1st, 2nd, and 3rd, but ends up being in a mismatched order. However, the total number of visits to the website is correct, as I visited the website 6 times in total. I'm not sure what is causing this, as the 1st visit to the website should correspond with the 1st message, 2nd to 2nd, etc.

**Detail the failure-inducing input and context. That might mean any or all of the command you're running, a test case, command-line arguments, working directory, even the last few commands you ran. Do your best to provide as much context as you can.**

I start the server using Javac and Java to compile and run `TestServer`. In Git Bash, I use curl http://localhost:4000 to visit the website normally, then I run my Bash Script to both check for the total website visits as well as copy the results of curling http://localhost:4000/iterate (which shows different messages based on the total number of times the website has been visited).

![1 terminal](https://github.com/Reckswell/CSE-15L-Lab-Report-5/assets/73510375/d76b4d1c-88f7-4ea7-9ee7-a90622ddd509)
![1 code](https://github.com/Reckswell/CSE-15L-Lab-Report-5/assets/73510375/6e720c60-26aa-4258-bf97-d6d676811b6c)

## TA's Response:
Is your use of the mod indicator for your if-else statements in `TestServer.java` corresponding with what you want to do? If every 3rd visit to the website should return the third message, then the corresponding `repeats % 3` should be a different value to match that. Try changing your mod indicators, and then restarting the server to see if it's fixed.

Also, be wary of having your function return null as a placeholder for java. Instead, change one of your else-if statements to just an else statement.

## Student's Response:
Thank you, it works now.

![image](https://github.com/Reckswell/CSE-15L-Lab-Report-5/assets/73510375/f4ca5fc5-39a7-4fdf-b883-186aa412c827)
![image](https://github.com/Reckswell/CSE-15L-Lab-Report-5/assets/73510375/ef52129a-fc15-4767-a145-8609c65542d0)

## Setup Information
File/Directory Structure:
* | +--- /Lab Report 5
  - | +--- testServer.sh
  - | +--- Server.java
  - | +--- TestServer.java

##Contents of each file prior to fixing:
### `TestServer.java`:
```
import java.io.IOException;
import java.net.URI;

class Handler implements URLHandler{
    //Counts the number of times the website has been visited.
    int repeats = 0;
    public String handleRequest(URI url) {
        System.out.println("Path: " + url.getPath());
        if (url.getPath().contains("iterate")) {
            repeats++;
            if(repeats % 3 == 0) {
                return "You are in the 1st state of the website. Maybe later, something will be different. Visits: " + repeats;
            }
            else if (repeats % 3 == 1) {
                return "You are in the 2nd state of the website. Who knows what lies ahead? Visits: " + repeats;
            }
            else if (repeats % 3 == 2) {
                return "You are in the 3rd state of the webiste. Just maybe, you were being tricked this whole time. Visits: " + repeats;
            }
            return null;
        }
        else {
            repeats++;
            return "Website works. Maybe if you put /iterate in the search bar, something will change \n \nTotal visits: " + repeats;
        }
    }
}

public class TestServer {
    public static void main(String[] args) throws IOException {
        if(args.length == 0){
            System.out.println("Missing port number! Try any number between 1024 to 49151");
            return;
        }

        int port = Integer.parseInt(args[0]);

        Server.start(port, new Handler());
    }
}
```
### `testServer.sh`
```
#!/bin/bash

SERVER_URL="http://localhost:4000/iterate"

status=$(curl -s $SERVER_URL)
iterations=${status#*: }
echo $status

if [[ "$status" == *"You are in the 1st state of the website. Maybe later, something will be different."* ]]; then
    echo "Should be 0th iteration"
    echo "Total iterations" $iterations
elif [[ "$status" == *"You are in the 2nd state of the website. Who knows what lies ahead?"* ]]; then
    echo "Should be 1st iteration"
    echo "Total iterations" $iterations
elif [[ "$status" == *"You are in the 3rd state of the webiste. Just maybe, you were being tricked this whole time."* ]]; then
    echo "Should be 2nd iteration"
    echo "Total iterations" $iterations
fi   
```
### `Server.java`
```
// A simple web server using Java's built-in HttpServer

// Examples from https://dzone.com/articles/simple-http-server-in-java were useful references

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.net.URI;

import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

interface URLHandler {
    String handleRequest(URI url);
}

class ServerHttpHandler implements HttpHandler {
    URLHandler handler;
    ServerHttpHandler(URLHandler handler) {
      this.handler = handler;
    }
    public void handle(final HttpExchange exchange) throws IOException {
        // form return body after being handled by program
        try {
            String ret = handler.handleRequest(exchange.getRequestURI());
            // form the return string and write it on the browser
            exchange.sendResponseHeaders(200, ret.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(ret.getBytes());
            os.close();
        } catch(Exception e) {
            String response = e.toString();
            exchange.sendResponseHeaders(500, response.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(response.getBytes());
            os.close();
        }
    }
}

public class Server {
    public static void start(int port, URLHandler handler) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(port), 0);

        //create request entrypoint
        server.createContext("/", new ServerHttpHandler(handler));

        //start the server
        server.start();
        System.out.println("Server Started! Visit http://localhost:" + port + " to visit.");
    }
}
```

### Command Lines that trigger Bugs:
```
$bash testServer.sh <- Out of order, should be the 1st message but is instead the 2nd
$curl http://localhost:4000/
$bash testServer.sh <- Out of order, should be the 3rd message but is instead the 1st
$curl http://localhost:4000/
$bash testServer.sh <- Out of order, should be the 2nd message but is instead the 3rd
$curl http://localhost:4000/
```

# Part 2: Reflection
I personally like learning about how the autograders are made, specifically the concept of how bash files are able to read contents of other files or grab contents from a webpage. I also like the concept of how you can use bash files to automate file modification (eg. mass filtering a large amount of files and then putting them all into a single document), as I think it could have alot of real-world applications.
