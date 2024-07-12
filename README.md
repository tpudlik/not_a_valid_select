# Spurious "is not a valid select() condition" errors

The `//:library_with_select` is incompatible with platform
`//:platform_a_is_incompatible_with`. This is because it depends on library
`:a`, which is `target_compatible_with` a `constraint_value` that
`//:platform_a_is_incompatible_with` does not possess.

So, when I build,

```
bazelisk build --platforms=//:platform_a_is_incompatible_with :library_with_select
```

I hope for the following output:

```
ERROR: Analysis of target '//:library_with_select' failed; build aborted: Target //:library_with_select is incompatible and cannot be built, but was explicitly requested.
Dependency chain:
    //:library_with_select (c037fd)
    //:label_flag (c037fd)
    //:a (c037fd)   <-- target platform (//:platform_a_is_incompatible_with) didn't satisfy constraint //:constraint_value_a
```

Instead, I get the following output:

```
ERROR: /usr/local/google/home/tpudlik/src/not_a_valid_select/BUILD.bazel:28:11: //:flag_points_to_a is not a valid select() condition for //:library_with_select.
To inspect the select(), run: bazel query --output=build //:library_with_select.
For more help, see https://bazel.build/reference/be/functions#select.
```

This is caused by lines 35--35 in `BUILD.bazel`:

```
    includes = select({
        ":flag_points_to_a": [],
        "//conditions:default": ["public"],
    }),
```

These lines are in fact correct, and would work fine if the `deps` of
`library_with_select` did not include an incompatible target.

Even worse, incompatible target skipping is broken by this! That is, if I run,

```
bazelisk build --platforms=//:platform_a_is_incompatible_with //...
```

I expect the build to succeed, because `:library_with_select` is incompatible
with the target platform. Instead, I get the same `//:flag_points_to_a is not a
valid select() condition for //:library_with_select` error as above.
