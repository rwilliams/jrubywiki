Redhat Linux
------------

<table>
	<tr>
		<th>JRuby Version</th>
		<th>jruby-openssl</th>
		<th>net-ssh</th>
	</tr>
	<tr>
		<td>1.1.6</td>
		<td>0.3</td>
		<td>2.0.4</td>
	</tr>
	<tr>
		<td>1.2.0</td>
		<td>0.3</td>
		<td>2.0.4</td>
	</tr>
</table>

Installing old gems
-------------------
Specify the version to install, e.g.:
```bash
 jruby -S gem install jruby-openssl -v 0.3
```

Using old gems in your script
-----------------------------
```ruby
 require 'rubygems'   # not required in a rails controller
 gem 'jruby-openssl', '0.3'
 gem 'net-ssh', '2.0.4'
 require 'net/ssh'
```


