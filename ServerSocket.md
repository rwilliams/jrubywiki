Server Sockets in JRuby
=======================

If you are here because of an error message in JRuby, here's a short explanation. A longer version comes later on this page.

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

The long description of why JRuby works like this follows below.

Details of Sockets and ServerSockets
------------------------------------

Because JRuby is built atop the Java Development Kit and uses its classes to implement the Ruby IO subsystem, we must choose at construction time whether a socket will be used for client or server operations. Specifically, we must construct a [```Socket```](http://docs.oracle.com/javase/6/docs/api/java/net/Socket.html) or a [```ServerSocket```](http://docs.oracle.com/javase/6/docs/api/java/net/ServerSocket.html) to back up the Ruby object.

This means that it's impossible for us to construct a Ruby ```Socket``` object and later either #connect or #accept on that object; we have to choose which of those operations will work right away.

Because connecting to remote servers is more common, Ruby ```Socket``` in JRuby is client-only, and therefore will only support the ```connect``` operation (not ```accept```). We define an additional class, ```ServerSocket``` that has the same basic operations as ```Socket``` but supports ```accept``` instead of ```connect```. By using ```ServerSocket``` when you want a server ```Socket```.

A simple example is at the top of this page, along with code you can include to make it work on non-JRuby versions of Ruby.

