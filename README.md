tree-sitter-auto
================

A tree-sitter adaptor for `less` that can automagically guess the file type.

Install
-------

Put the script somewhere, make sure it's writiable with `chmod +x
tree-sitter-auto`, and then put something like this in your shell's init script:

```sh
export LESSOPEN="||-/path/to/tree-sitter-auto -- %s"
```

If you are already using lesspipe and don't want to replace it wholesale, then
you can create a wrapper script:

```sh
#!/bin/sh

tree-sitter-auto -- "$1" || lesspipe.sh "$1"
```
And then set `LESSOPEN` in your shell init:
```sh
export LESSOPEN="||-/path/to/wrapper %s"
```

Adding more content detectors
-----------------------------

New checks should be added to the bottom of the `match_content`
function. Remember that exec-ing is slow, so try to do everything possible to
only use bash built-ins and operators. Bash might have more built-in than you
realize—check out [bash parameter expansion syntax][bash-param-exp], [the
`extglob` glob syntax][bash-extglob] (which are enabled here), and [the regex
operator `=~`][bash-regex] for more info.

Bash regexes can be kind of weird, luckily it's pretty easy to test bash
right in your terminal:

```bash
$ chunk=$(head -c 4096 some-test-code.whatever)
$ [[ $chunk =~ test-your-regex-here ]] && echo -n matched || echo nope
```

Then just keep up-arrowing and editing the regexp (and if you're like me,
smashing your head against the desk and then re-consulting the [bash
manual][bash-regex] 😆) until it works.

A regex with a lot of spaces can be real tricky since bash uses spaces to signal
the end of a regex. This means you have to escape the spaces with `\` or put
them in quotes. This can get ugly quick. One trick is to use Bash's `$''` quotes
on a temporary variable and then use it as the regex:

```bash
re=$'\n *fn[ <]|\n *async fn[ <]|\nimpl[ <]'
if [[ $chunk =~ $re ]]; then
...
```

This also useful if you want to search for beginning or end of lines, as shown
here (`^` won't work as it matches the beginning of the `$chunk` string which is
the beginning of the file).

[bash-param-exp]: https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html
[bash-extglob]: https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html
[bash-regex]: https://www.gnu.org/software/bash/manual/html_node/Conditional-Constructs.html

License
-------

Copyright © 2023 David Caldwell <david@porkrind.org>

Licensed under the MIT License (see LICENSE.md for details)
