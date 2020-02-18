RDoc has generally been used as the primary measure of general-purpose performance in JRuby, since it involves extremely heavy interpreted code and since it performs rather poorly under JRuby currently.

==Profiling output from gem install rake==

This output is from running a gem install rake (local) with RDoc/RI generation and profiling enabled. It can be used to help work on improving JRuby performance.

  %   cumulative   self              self     total
 time   seconds   seconds    calls  ms/call  ms/call  name
 12.42    56.41     56.41    37464     1.51     2.37  RubyToken.Token
  6.65    86.62     30.21    60000     0.50     5.78  IRB::SLex::Node#match_io
  4.68   107.90     21.28   233778     0.09     0.09  RubyLex::BufferedReader#getc
  4.42   128.00     20.10    80126     0.25     3.34  RDoc::RubyParser#get_tk
  4.00   146.16     18.16   207562     0.09     0.18  RubyLex#getc
  3.91   163.94     17.78     1201    14.81    36.71  SM::AttributeManager#convert_specials
  3.04   177.76     13.83   132797     0.10     0.10  Range#each
  2.97   191.27     13.51     8538     1.58     6.11  RubyLex#identify_identifier
  2.91   204.51     13.24   626767     0.02     0.02  Kernel.==
  2.90   217.67     13.15    33784     0.39     7.02  RubyLex#token
  2.79   230.32     12.66   130162     0.10     0.20  SM::AttrSpan#set_attrs
  2.53   241.84     11.51      580    19.85  1639.52  RDoc::RubyParser#parse_statements
  2.23   251.97     10.13     1201     8.44    12.47  SM::AttributeManager#split_into_flow
  1.76   259.95      7.99    24006     0.33     9.94  RDoc::RubyParser#skip_tkspace
  1.65   267.45      7.50     1166     6.43    29.16  TemplatePage#substitute_into
  1.42   273.91      6.45     1856     3.48    13.22  RubyLex#identify_comment
  1.40   280.26      6.35     5678     1.12     1.68  #<Class:JavaUtilities>#matching_method
  1.23   285.84      5.59    36736     0.15     0.37  IRB::Notifier::LeveledNotifier#notify?
  1.17   291.14      5.30   239335     0.02     0.02  Kernel.hash
  1.06   295.98      4.84    54864     0.09     0.14  RubyLex#ungetc
  1.02   300.61      4.63   202748     0.02     0.02  Kernel.is_a?
  1.00   305.14      4.53    46342     0.10     0.12  RDoc::RubyParser#unget_tk

Here is the same output under Ruby 1.8.5 on OS X:

  %   cumulative   self              self     total
 time   seconds   seconds    calls  ms/call  ms/call  name
 10.65   115.46    115.46   233778     0.49     0.65  RubyLex::BufferedReader#getc
  4.83   167.82     52.36   175207     0.30    24.57  Array#each
  4.15   212.84     45.02     1201    37.49    86.34  SM::AttributeManager#split_into_flow
  3.65   252.47     39.63    80126     0.49     6.67  RDoc::RubyParser#get_tk
  3.54   290.81     38.34    13271     2.89    13.03  Hash#each
  3.27   326.32     35.51    60000     0.59    11.40  IRB::SLex::Node#match_io
  2.98   358.68     32.36   926916     0.03     0.03  String#==
  2.91   390.29     31.61    40440     0.78     2.41  Hash#each_value
  2.13   413.34     23.05     8538     2.70    12.23  RubyLex#identify_identifier
  1.82   433.05     19.71     1856    10.62    44.87  RubyLex#identify_comment
  1.73   451.82     18.77      580    32.36  3264.64  RDoc::RubyParser#parse_statements
  1.69   470.10     18.28   207562     0.09     0.74  RubyLex#getc
  1.67   488.18     18.08    37464     0.48     1.73  RubyToken.Token
  1.62   505.80     17.62   551810     0.03     0.03  Module#===
  1.55   522.64     16.84    54864     0.31     0.44  RubyLex::BufferedReader#ungetc
  1.53   539.19     16.55    33938     0.49     0.82  Array#&
  1.52   555.69     16.50   479047     0.03     0.03  Fixnum#==
  1.38   570.70     15.01    39503     0.38     9.25  Proc#call
  1.31   584.86     14.16   410340     0.03     0.03  Fixnum#<
  1.30   599.00     14.14    33784     0.42    13.61  RubyLex#token
  1.30   613.08     14.08   466655     0.03     0.03  String#[]
  1.18   625.85     12.77   402553     0.03     0.03  Fixnum#+
  1.15   638.32     12.47   119803     0.10     0.13  SM::AttrSpan#[]
  1.11   650.33     12.01    46342     0.26     0.51  RDoc::RubyParser#unget_tk
