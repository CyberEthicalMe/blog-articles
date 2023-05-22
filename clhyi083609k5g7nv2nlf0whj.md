---
title: "I've asked ChatGPT to write a honeypot"
seoDescription: "Is it possible to create a simple working honeypot using prompt engineering? How does it perform under port scanner?"
datePublished: Mon May 22 2023 07:00:39 GMT+0000 (Coordinated Universal Time)
cuid: clhyi083609k5g7nv2nlf0whj
slug: ive-asked-chatgpt-to-write-a-honeypot
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1684704568043/7e3e8f43-661b-467a-825c-7c3f1837d90b.png
tags: cybersecurity-1, honeypot, chatgpt

---

%%[support-cta] 

# Introduction

> Honeypots are decoy systems that mimic real targets and lure attackers into revealing their techniques and intentions. In this article, we will explore how to create honeypots using various tools and methods, and how to analyze the data collected from them. We will also discuss the benefits and challenges of using honeypots for cybersecurity purposes.
> 
> *{Bing AI, Professional, Paragraph, Short}*

At this point I would like to state, I'm no prompt engineering expert. So the methods I'm using are simply conclusions from previous experience, or articles I'm reading.

# Idea

Yeah, why not? I've seen people [creating games](https://www.youtube.com/watch?v=IyKKhxYJ4U4) when they don't have any experience with programming or making a [Minecraft hack](https://www.youtube.com/watch?v=ukKfAV4Ap6o). I thought, let's see by myself where I can get.

So the plan was to write a honeypot in Python, that will simply respond to connections on port 80 in a way it can "fool" a malicious actor. I'm using free online version of ChatGPTv3 for that - becasue I think it is the most accessible service.

# Initial Prompt

<details data-node-type="hn-details-summary"><summary>AI Prompt</summary><div data-type="detailsContent">Assume a role of cybersecurity expert that is advising your colleague who writes from a scratch a simple honeypot software in Python. The basic version responds only to HTTP requests on port 80. He asks you for example code of such software.</div></details>

I'm assuming there could be some ethical problems for AI to discuss some of the topics like port scanning techniques and such - so I've started with some older version of bypass to mitigate negative responses. "Assume a role of expert who is advising somebody else". In the first response advised me: *it is important to thoroughly test and secure any honeypot software before deploying it in a real-world environment*.

The first version of honeypot looks like this (original, non-altered version - comments are also from AI):

```python
import socket

def honeypot():
    host = ''
    port = 80

    # create socket object
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # bind socket to a public host and a well-known port
    s.bind((host, port))

    # listen for incoming connections
    s.listen(5)

    print('Listening on port', port)

    while True:
        # establish connection with client
        conn, addr = s.accept()

        # log the connection
        print('Received connection from', addr)

        # send a fake HTTP response
        response = 'HTTP/1.1 200 OK\r\nContent-Length: 20\r\n\r\n<Honeypot response>'
        conn.sendall(response.encode())

        # close the connection
        conn.close()

if __name__ == '__main__':
    honeypot()
```

To be honest, it does its work. I had a question about what `s.listen(5)` means, which was answered in details.

# Improvements

%%[join-cta] 

After that, I introduced some suggestions and more questions, for example, I wanted that:

* Content-Length to be calculated dynamically,
    
* string interpolation was used instead of `format()` ,
    
* get more information about the client from incoming connection,
    
* log the information in the file in a way I can later use the data to analyze it.
    

Before I concluded my work I asked ChatGPT if there are any security vulnerabilities that I should address before I start using it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684678620454/3a3855c0-ef21-4b36-b650-7be1d1099165.png align="center")

If you think about it - some of these hints are not making sense in honeypot - like filtering incoming connections or implementing authentication mechanisms.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684678700705/32ed3d0b-47e4-44ba-ae13-ebae39af5943.png align="center")

So well, it's got a good point that honeypots could be used as a proxy or could act as an entrypoint for the internal network.

# Implementation

Finally I've ended with following code

```python
import socket
import datetime

HOST = ''  # Empty string means bind to all available interfaces
PORT = 80  # Port to listen on

# Create a socket object
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Bind the socket to a public host and port
s.bind((HOST, PORT))

# Set the socket to listening mode with a maximum backlog of connections
s.listen(5)

print(f'Honeypot is listening on {HOST}:{PORT}')

# Open the log file for writing
with open('honeypot.log', 'a') as f:

    while True:
        # Wait for a connection
        conn, addr = s.accept()

        # Log the connection
        current_time = datetime.datetime.now()
        print(f'Received connection from {addr[0]}:{addr[1]} at {current_time}')
        f.write(f'{current_time}, connection from {addr[0]}:{addr[1]}\n')

        # Receive the data in small chunks and retransmit it
        data = b''
        try:
            while True:
                chunk = conn.recv(1024)
                if not chunk:
                    break
                data += chunk

                # Check if the data size is too large
                if len(data) > 1024*1024:
                    print(f'Data from {addr[0]}:{addr[1]} is too large, closing connection.')
                    f.write(f'{current_time}, data from {addr[0]}:{addr[1]} is too large, closing connection\n')
                    conn.close()
                    break

            # Send the response with a custom banner
            response = b'HTTP/1.1 200 OK\r\n'
            response += b'Server: Apache/2.2.17 (Unix) mod_ssl/2.2.17 OpenSSL/0.9.8g\r\n'
            response += b'Content-Type: text/html\r\n'
            response += b'Content-Length: 20\r\n'
            response += b'\r\n'
            response += b'This is not the web page you are looking for.'

            conn.sendall(response)

        except (socket.error, ConnectionResetError):
            print(f'Error receiving data from {addr[0]}:{addr[1]}, closing connection.')
            f.write(f'{current_time}, error receiving data from {addr[0]}:{addr[1]}, closing connection\n')
        
        except socket.timeout:
            print(f'Timeout error occurred when sending response to {addr[0]}:{addr[1]}, closing connection.')
            f.write(f'{current_time}, timeout error when sending response to {addr[0]}:{addr[1]}, closing connection\n')
        
        finally:
            # Log the request
            f.write(f'{current_time}, request from {addr[0]}:{addr[1]}, data: {data.decode("utf-8", "ignore")}\n')
            # Close the connection
            conn.close()
```

# Tests

First, I've tried a simple `curl`. It worked.. partially. For some reason, `curl` hanged. But I've ignored that issue and launched the `nmap -sV -sC -p80 localhost -vvv` scan. Interestingly, it was completed in a little below 4 minutes. So that simple program can slow down attackers to 4 minutes on a single port 80! Well, that's quite amazing. Interestingly, because we are logging each request in the file - I can see what `nmap` is doing behind the scenes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684680405449/65f4858a-204d-4509-9d7d-661883ae97df.png align="center")

The results of the scan are inconclusive.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684703023638/4da41569-bcf1-476c-81f7-5a3b20649ee1.png align="center")

# Conclusion

Did it work? Yeah, I must say it works pretty well for the untargeted scans. It's far too little tests done, but I wanted mainly to check how it will handle such a specific task - and how would such honeypot perform under `nmap` scan.

There is still a lot of work to do, to make it more robust, add some more services, improve logging.

## Resources

%%[follow-cta] 

* [Using AI To Build A Game From Scratch (NO Experience) | Matt Wolfe](https://www.youtube.com/watch?v=IyKKhxYJ4U4)
    
* [Can AI Create a Minecraft Hack? | LiveOverflow](https://www.youtube.com/watch?v=ukKfAV4Ap6o)
    
* [Extensive list of honeypots software](https://github.com/paralax/awesome-honeypots)