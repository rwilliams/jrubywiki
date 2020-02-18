JRuby is available at our [http://www.jruby.org/download website], but it is also available in many distributions.  This page documents all the brave souls who help package up JRuby and distribute it via various packaging technologies. Currently, the best way of installing JRuby for POSIX systems is: [https://github.com/rbenv/rbenv/wiki rbenv] or [http://rvm.io/ RVM]. Linux packages are hopelessly out of date, with the exception of Archlinux and Alpine Linux.

{|- border="2"
!OS
!Package
!Maintainer
!Current versions (1.7.27, 9.1.12.0)
!Notes
|-
|POSIX
|[https://github.com/rbenv/rbenv/wiki rbenv]
| Mislav Marohnić
| 1.7.24, 9.0.5.0
| see also [https://github.com/rbenv/ruby-build ruby-build] (available Ruby interpreters: ruby-build --definitions)
|-
|POSIX
|[http://rvm.io/ RVM]
| Wayne E. Seguin & Michal Papis
| 1.7.11
| (available Ruby interpreters: rvm list known)
|-
|Mac OS-X
|[http://brew.sh/ Homebrew]
| [https://github.com/Homebrew?tab=members maintainers]
| 9.2.7.0
| [https://formulae.brew.sh/formula/jruby formula]
|-
|Mac OS-X
|[http://www.macports.org/ MacPorts]
| ameingast?
| 1.7.4
| [http://trac.macports.org/browser/trunk/dports/lang/jruby/Portfile portfile]
|-
|Mac OS-X
|Fink
| None
|
|
|-
|Redhat/Fedora
|RPM
| Mohammed Morsi
| 1.5.3 [1.5.6 requested]
|[https://bugzilla.redhat.com/show_bug.cgi?id=561484 issue]
|-
|Debian
|APT
| Sebastien Delafond, Torsten Werner ???
|
|
|-
|Ubuntu
|APT
| ubuntu-motu : lists.ubuntu.com
| 1.5.1
| on Natty
|-
|Gentoo
|Portage (dev-java/jruby)
| 
| 1.6.8 
| [http://packages.gentoo.org/package/dev-java/jruby gentoo portage]
|-
|Alpine Linux
|APK (community/jruby)
| Jakub Jirutka
| 9.1.12.0
| [https://pkgs.alpinelinux.org/package/edge/community/x86_64/jruby package]
|-
|Archlinux
|pacman (community/jruby)
| [https://www.archlinux.org/packages/?maintainer=heftig Jan Steffens]
| 9.2.7.0
| [https://www.archlinux.org/packages/community/any/jruby/ package]
|-
|Windows
|install4j
| jruby-core
| 1.7.24, 9.0.5.0
| [http://jruby.org/download download installer]
|-
| Windows, Linux, Mac OS X, Virtual Appliance and Amazon Cloud Images
| BitNami
| BitNami
| 9.2.7.0 (frequently updated)
| [http://bitnami.org/stack/jrubystack download]
|}