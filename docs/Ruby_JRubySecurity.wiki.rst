[[Home|&raquo; JRuby Project Wiki Home Page]] &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [[Internals|&raquo; Design: Internals]] 
<h1>Ruby/JRuby Security</h1>
A new implementation to replace Safe/Taint, since Safe and Taint do not work and provide a false sense of 
security.  There are two options: Using a Sandbox type environment or a permission-based protocol.  The Sandbox has been around since Java 1.0 and was found to be good only at a macro level.  For finer grained control, the <tt>Permission</tt> class and <tt>AccessControl</tt> class were introduced in Java 1.2.  

Having a <tt>Permission</tt> based security implementation would be ideal.



'''Requirements'''
* It must be something that can be implemented in MRI/KRI, possibly using the same API/mechanisms used right now.
* It must not introduce overhead back into the system like taint/safe does now. 
* Security should be based on addition of rights instead of the removal of them.
* Fine-grained control should be allowed for distributed programming.  For example, DRb


'''Possible Solutions'''
* Keep the current Implementation of Safe and Taint.
* A sandbox based on the same principles as the Java Sandbox.<br/>Here is a sandbox base for MRI: [http://code.whytheluckystiff.net/sandbox].
* Use a <tt>Permission</tt> based architecture such as Java 1.2 and later.