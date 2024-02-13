# Proxy buffer directive in Nginx

Performance optimization done by developers who have spent countless hours optimizing every little thing goes down the drain as soon as a client machine has a slow internet connection. The data transfer gets bottlenecked when sending a significant size of response to a client with low bandwidth. As server waits for the acknowledgement of the each packet it sends to the client, it significantly slows down the response time and impacts the server's ability to handle the other requests, potentially leading to performance degradation.

Thanks to the proxy buffer feature of nginx, we could maintain consistent performance on the server. nginx's proxy_buffer saves the entire response from the server in its buffer and frees up the server to take on other requests. Meanwhile, clients will get the data from nginx at their own pace without degrading the server performance. It may not improve the response time, limited by the client's connection but surely would improve the performance of the backend.

PS: you can set the proxy_buffer_size dynamically based on the response headers.