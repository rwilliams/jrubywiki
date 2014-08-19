JRuby uses [LoadService](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/runtime/load/LoadService.java) as the main mechanism of loading files for Kernel.require and Kernel.load.

1) Kernel.require is handled by [LoadService.require](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/runtime/load/LoadService.java#L391) method and Kernel.load is handled by [LoadService.load](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/runtime/load/LoadService.java#L319) method.

2) Both methods create a [SearchState](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/runtime/load/LoadService.java#L812) object to reflect the search strategy (which boils down to which suffixes ought to be used for the search) and try to find a matching Library with [findLibraryBySearchState](https://b2.corp.google.com/issues/17023210).

3) findLibraryBySearchState primarily relies on LibrarySearcher.findBySearchState to resolve paths, but we also use findLibraryWithClassloaders](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/runtime/load/LoadService.java#L978) until file resources fully support classpath:/ based paths.

4) findBySearchState attempts to locate a library by iterating over suffixes from the searchState and attempting to load one of three following libraries:

1. Builtin libraries, which are registered with `LoadService` using `addBuiltinLibrary`.
2. A [ResourceLibrary](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/runtime/load/LibrarySearcher.java#L200) that is backed by a [FileResource](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/util/FileResource.java). See next section for details.
3. [ClassExtensionLibrary](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/runtime/load/LibrarySearcher.java#L105) that looks for FooService java class that implements BasicLibraryService for loading.

If a match is found, it is immediately returned.

5) If a library is found, it gets loaded via its `load` method.

## Handling libraries backed by resources.

ResourceLibrary is located by the [findResourceLibrary](https://github.com/jruby/jruby/blob/master/core/src/main/java/org/jruby/runtime/load/LibrarySearcher.java#L114) method. This is the flow that handles the 'regular' ruby loading semantics, so we have to consider path special cases as well as LOAD_PATH:

* If path starts with ./ or ../, starts with a schema (e.g. file: or jar:) or java.lang.File.isAbsolute returns true for the path, we bypass load paths and treat the path as-is.
* If path starts with ~/, we replace ~/ with value of ENV["HOME"] and bypass loads paths.
* Any other cases we iterate over the LOAD_PATH array and look for (LOAD_PATH_ENTRY + "/" + path) path. 

After the path mangling step, we create a resource with JRubyFile.createResource. If the return resource actually exists, we return ResourceLibrary that handles the specific of the load or return null if resource points to a non-existent resource.

There are three load strategies for loading libraries backed by a resource:

1. If path ends with ".jar" (e.g. /foo/bar/lib.jar) we handle it as a 'jar' load. That means adding the path to the jar to the classloader classpath and also attempting to load accompanying `BasicLibraryService` (e.g. /LibService.class).

2. If path ends with ".class" we treat it as a compiled script, and use `CompiledScriptLoader` to load.

3. In any other cases, we load the script with `Ruby.loadFile`.