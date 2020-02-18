Server Sockets in JRuby
=======================

JRuby uses classes from the Java platform for sockets, so our API differs slightly when it comes to the lowest-level "Socket" class.

Creating a Server Socket
------------------------

In JRuby, the Ruby ```Socket``` class can only be used for client-side connections. As a result, only ```connect``` works, not ```accept```. We provide a separate class called ```ServerSocket``` with mostly the same API as ```Socket```, but it handles ```accept``` and not ```connect```.

Here's a simple example of a server:

```
require "socket"

socket = ServerSocket.new(Socket::AF_INET, Socket::SOCK_STREAM, 0)
sockaddr = ServerSocket.pack_sockaddr_in(12345, "127.0.0.1")
socket.bind(sockaddr)
socket.accept
```

The API works basically like ```Socket```, but with ```ServerSocket``` in its place. You can also include the following code to make server sockets that work on JRuby and other impls with both server and client support in ```Socket```:

```
if RUBY_ENGINE != 'jruby'
  ServerSocket = Socket
end
```

Bind and Listen and Backlogs
----------------------------

The normal sequence for listening for connections on a Ruby Socket is to call ```new``` for a new Socket, ```bind``` to bind to an address and port, ```listen``` to express intent to listen for connections with the specified connection backlog, and finally ```accept``` to accept incoming connections.

On JRuby's ServerSocket, ```bind``` and ```listen``` are combined into ```bind```, which accepts an optional backlog argument.

Here's an example:

```
socket = ServerSocket.new(Socket::AF_INET, Socket::SOCK_STREAM, 0)
sockaddr = ServerSocket.pack_sockaddr_in(12345, "127.0.0.1")
socket.bind(sockaddr, 5)
# no call to listen...
# socket.listen(5)
socket.accept
```

For non-JRuby impls, the following simple monkey-patch makes their ```bind``` work like ours, for API-compatibility:

```
if RUBY_ENGINE != 'jruby'
  ServerSocket = Socket # as shown above
  class Socket
    alias :_old_bind, :bind
    def bind(addr, backlog)
      _old_bind(addr)
      listen(backlog)
    end
  end
end
```

Details of Sockets and ServerSockets
------------------------------------

Because JRuby is built atop the Java Development Kit and uses its classes to implement the Ruby IO subsystem, we must choose at construction time whether a socket will be used for client or server operations. Specifically, we must construct a [```Socket```](http://docs.oracle.com/javase/6/docs/api/java/net/Socket.html) or a [```ServerSocket```](http://docs.oracle.com/javase/6/docs/api/java/net/ServerSocket.html) to back up the Ruby object.

This means that it's impossible for us to construct a Ruby ```Socket``` object and support both ```connect``` and ```accept``` on that same object; we have to choose which of those operations will work right away.

Because connecting to remote servers is more common, ```Socket``` in JRuby is client-only, and therefore will only support the ```connect``` operation (not ```accept```). We define an additional class, ```ServerSocket``` that has the same basic operations as ```Socket``` but supports ```accept``` instead of ```connect```.

A simple example is at the top of this page, along with code you can include to make it work on non-JRuby versions of Ruby.

Details of Bind and Listen
--------------------------

The JDK ServerSocket class provides no ```listen``` method, instead allowing users to specify backlog at construction time (while simultaneously binding a port) or at bind time (combining the POSIX ```bind``` and ```listen``` operations). Because of this, we have no way to separate the two phases of "binding" and "listening" in the API we expose to Ruby.

For this reason, ```ServerSocket#bind``` in JRuby does both ```bind``` and ```listen``` at the same time, and accepts an optional backlog argument.

See above for an example, along with code you can use to make the API work in non-JRuby versions of Ruby.