Some tips to control how much memory JRuby will consume:

* Use a 32-bit VM if you want to lower your memory consumption
* Use the client VM if possible
* Use both -J-Djruby.memory.max=NNNm and -J-XmxNNNm to set the maximum heap size
* Use a different GC, one that consumes less memory