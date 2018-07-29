---
layout: post
comments: true
title:  "Jetty and loopback"
---

One day i have got the simple task from my QA department to compel my Jetty server to run on loopback for further support improvements.
Fortunatly it turned out quite simply as a result, but in the process of searching some info about it i'v spent a lot of time. 
So, I decidet to put this info to help you and make something like reminder for myself.

Let's see my main server run class:

```java
public class Server extends org.eclipse.jetty.server.Server {
    private static final Logger LOG = LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());
 
    public static void main(String[] args) {
        new JCommander(Parameters.getInstance(), args);
 
        final Server server = new Server();
 
        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    server.stop();
                } catch (Exception ex) {
                    LOG.error("Shutting down server error.", ex);
                }
            }
        }));
 
        server.initConnector(server);
        server.setupAndStart();
    }
	
	private void initConnector(Server server) {
        ServerConnector connector = new ServerConnector(server);
        connector.setHost(Parameters.getInstance().getHost());
        connector.setPort(Parameters.getInstance().getPort());
        server.addConnector(connector);
    }
	
	private void setupAndStart() {
        //start Jetty
    }
 
	//server initialisation code...
 
}
```


The key part of this code is [ServerConnector.](http://www.eclipse.org/jetty/javadoc/current/org/eclipse/jetty/server/ServerConnector.html)

So, as you can see:

Create new object of ServerConnector
```java
ServerConnector connector = new ServerConnector(server);
```

Specify the necessary host and port through our Parameters.
```java
connector.setHost(Parameters.getInstance().getHost());
connector.setPort(Parameters.getInstance().getPort());
```

Set it to our Jetty server
```java
server.addConnector(connector);
```

Well, that's all you need for your Jetty loopback.


{% if page.comments %}
<div id="disqus_thread"></div>
<script>
(function() { 
var d = document, s = d.createElement('script');
s.src = 'https://https-kussart-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
{% endif %}