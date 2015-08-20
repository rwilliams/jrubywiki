For those people interested in helping to triage JRuby issues, then this page will help you get rolling...

# Initial Triage

First thing to consider is the quality of the issue (see JRubyBugReportingStyleGuide).  If someone is not providing enough info or their reproduction is too large then you can follow up as a comment and ask them for more info.  PLEASE NOTE:  Every issue report we receive we should treat as a gift.  Please treat the reporter in as pleasant of a way as possible.  We want them to keep reporting issues and potentially get more involved in our project.

Another important aspect of triaging an issue is which compatibility level is the issue for?  Is someone reporting a Ruby 2.2 issue against JRuby 1.7.22 in Ruby 1.9.3 mode?  Are they reporting against 2.2 and JRuby 9k, but the compatibility issue has existed since Ruby 1.8.7?  Narrow down which Ruby release the bug is valid for and then mark the labels 'JRuby 1.7' and/or 'JRuby 9000'.  In a comment specify which versions you tracked the problem back to (e.g. "I can confirm this behavior changed in 1.8.7").

Highly recommended is to use RVM to get each significant release (1.8.7, 1.9.3, 2.0, 2.2.x) and then set up aliases like:

```sh
alias mri22='rvm 2.2.2 do ruby'
alias mri20='rvm 2.0.0 do ruby'
alias mri19='rvm 1.9 do ruby'
alias mri18='rvm 1.8.7 do ruby'
```

Then you can simply run scripts:

```sh
mri19 -e '1/ 1.0'
```

# Completing An Issue

Once an issue has been fixed or you determine that the issue is not valid you should mark a milestone in addition to closing the issue.  

For issues which are duplicate or invalid you can mark as 'Invalid or Duplicate'.  If an issue was deemed not fixable then mark it as 'Wont Fix'.  If the issue is not tied to any particular release then mark it as 'Non-Release'.

For issues which have been corrected then there are two rules.  First if the issue only affects a single support branch (e.g. only fixes a Ruby 1.8.7 problem on JRuby 1.7.x) then mark it against the current JRuby 1.7 point release (e.g. 1.7.22).  If it applies to both release branches then also mark it against the older support branch (e.g. 1.7.22 vs 9.0.1.0).  We only do this because github does not support multiple milestone selection at this time.  Once we do then we should mark both point releases.