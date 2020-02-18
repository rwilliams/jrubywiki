Clone and Build JRuby on any Java 7+ JDK following the [BUILDING guide](https://github.com/jruby/jruby/blob/master/BUILDING.md).

Running the base test suite happens via `bin/jruby -S rake test...`. Longer suites are available via rake targets.

* JIT-forced and AOT-forced JRuby suites: `rake test:jruby:jit` or `rake test:jruby:aot`
* JIT-forced and AOT-forced MRI 1.9 suites: `rake test:mri:jit` or `rake test:mri:aot`

Fully using `invokedynamic` is be *off* by default but can be toggled using a command line switch e.g.

```
$ JRUBY_OPTS="-Xcompile.invokedynamic=true -J-Xmx1024M" bin/jruby -S rake test:mri:aot
...
```

Benchmarks are available in the [rubybench project](https://github.com/jruby/rubybench), and can be used to test performance and JIT.