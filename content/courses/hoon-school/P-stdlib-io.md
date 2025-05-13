# 15. Text Processing II

_This module will elaborate on text representation in Hoon, including formatted text and `%ask` [generators](urbit-docs/glossary/generator).  It may be considered optional and skipped if you are speedrunning Hoon School._


##  Text Conversions

We frequently need to convert from text to data, and between different text-based representations.  Let's examine some specific [arms](urbit-docs/glossary/arm):

- How do we convert text into all lower-case?
    - [++cass](urbit-docs/language/hoon/reference/stdlib/4b#cass)

- How do we turn a `cord` into a [tape](urbit-docs/glossary/tape)?
    - [++trip](urbit-docs/language/hoon/reference/stdlib/4b#trip)

- How can we make a [list](urbit-docs/glossary/list) of a null-terminated tuple?
    - [++le:nl](urbit-docs/language/hoon/reference/stdlib/2m#lenl)

- How can we evaluate [Nock](urbit-docs/glossary/nock) expressions?
    - [++mink](urbit-docs/language/hoon/reference/stdlib/4n#mink)

(If you see a `|*` [bartar](urbit-docs/language/hoon/reference/rune/bar#-bartar) rune in the code, it's similar to a `|=` [bartis](urbit-docs/language/hoon/reference/rune/bar#-bartis), but produces what's called a [_wet gate_](urbit-docs/courses/hoon-school/R-metals).)

The `++html` core of the standard libary contains some additional important tools for working with web-based data, such as [MIME types](https://en.wikipedia.org/wiki/Media_type) and [JSON strings](https://en.wikipedia.org/wiki/JSON).

- To convert a `@ux` hexadecimal value to a `cord`:

    ```hoon
    > (en:base16:mimes:html [3 0x12.3456])  
    '123456'
    ```

- To convert a `cord` to a `@ux` hexadecimal value:

    ```hoon
    > `@ux`q.+>:(de:base16:mimes:html '123456')
    0x12.3456
    ```

- There are tools for working with Bitcoin wallet base-58 values, JSON strings, XML strings, and more.

    ```hoon
    > (en-urlt:html "https://hello.me")
    "https%3A%2F%2Fhello.me"
    ```


##  Formatted Text

Hoon produces messages at the [Dojo](urbit-docs/glossary/dojo) (or otherwise) using an internal formatted text system, called `tank`s.  A `+$tank` is a formatted print tree. Error messages and the like are built of `tank`s.  `tank`s are defined in `hoon.hoon`:

```hoon
::  $tank: formatted print tree
::
::    just a cord, or
::    %leaf: just a tape
::    %palm: backstep list
::           flat-mid, open, flat-open, flat-close
::    %rose: flat list
::           flat-mid, open, close
::
+$  tank
  $~  leaf/~
  $@  cord
  $%  [%leaf p=tape]
      [%palm p=(qual tape tape tape tape) q=(list tank)]
      [%rose p=(trel tape tape tape) q=(list tank)]
  ==
+$ tang (list tank) :: bottom-first error
```

The [++ram:re](urbit-docs/language/hoon/reference/stdlib/4c#ramre) arm is used to convert these to actual formatted output as a [tape](urbit-docs/glossary/tape), e.g.

```hoon
> ~(ram re leaf+"foo")
"foo"
> ~(ram re [%palm ["|" "(" "!" ")"] leaf+"foo" leaf+"bar" leaf+"baz" ~])
"(!foo|bar|baz)"
> ~(ram re [%rose [" " "[" "]"] leaf+"foo" leaf+"bar" leaf+"baz" ~])
"[foo bar baz]"
```

Many [generators](urbit-docs/glossary/generator) build sophisticated output using `tank`s and the short-format [cell](urbit-docs/glossary/cell) builder `+`, e.g. in `/gen/azimuth-block/hoon`:

```hoon
[leaf+(scow %ud block)]~
```

which is equivalent to

```hoon
~[[%leaf (scow %ud block)]]
```

`tank`s are the primary output mechanism for more advanced generators. Even if you don't end up writing them much, you will encounter them as you delve into the Urbit codebase.

#### Tutorial:  Deep Dive into `ls.hoon`

The [+ls](urbit-docs/manual/os/dojo-tools#ls) generator shows the contents at a particular path in [Clay](urbit-docs/glossary/clay):

```hoon
> +cat /===/gen/ls/hoon
/~nec/base/~2022.6.22..17.25.54..1034/gen/ls/hoon
::  LiSt directory subnodes
::
::::  /hoon/ls/gen
  ::
/?    310
/+    show-dir
::
::::
  ::
~&  %
:-  %say
|=  [^ [arg=path ~] vane=?(%g %c)]
=+  lon=.^(arch (cat 3 vane %y) arg)
tang+[?~(dir.lon leaf+"~" (show-dir vane arg dir.lon))]~
```

Let's go line by line:

```hoon
/?    310
/+    show-dir
```

The first line `/?` faswut represents now-future functionality which will allow the version number of the kernel to be pinned.  It is currently non-functioning but you will see it in many Urbit-shipped files.

Then the `show-dir` library is imported.

```hoon
~&  %
```

A separator `%` is printed.

```hoon
:-  %say
```

A `%say` [generator](urbit-docs/glossary/generator) is a cell with a metadata tag `%say` as the head and the [gate](urbit-docs/glossary/gate) as the tail.

```hoon
|=  [^ [arg=path ~] vane=?(%g %c)]
```

This generator requires a path argument in its sample and optionally accepts a [vane](urbit-docs/glossary/vane) tag (`%g` [Gall](urbit-docs/glossary/gall) or `%c` [Clay](urbit-docs/glossary/clay)).  Most of the time, [+cat](urbit-docs/manual/os/dojo-tools#cat) is used with Clay, so `%c` as the last entry in the type union serves as the [bunt](urbit-docs/glossary/bunt) value.

```hoon
=+  lon=.^(arch (cat 3 vane %y) arg)
```

We saw `.^` [dotket](urbit-docs/language/hoon/reference/rune/dot#-dotket) for the first time in [the previous module](urbit-docs/courses/hoon-school/O-subject), where we learned that it performs a _peek_ or [scry](urbit-docs/glossary/scry) into the state of an Arvo [vane](urbit-docs/glossary/vane).  Most of the time this functionality is used to ask `%c` [Clay](urbit-docs/glossary/clay) or `%g` [Gall](urbit-docs/glossary/gall) for information about a path, [desk](urbit-docs/glossary/desk), [agent](urbit-docs/glossary/agent), etc.  In this case, `(cat 3 %c %y)` is a fancy way of collocating the two `@tas` terms into `%cy`, a Clay file or directory lookup.  The type of this lookup is `+$arch`, and the location of the file or directory is given by `arg` from the sample.

```hoon
tang+[?~(dir.lon leaf+"~" (show-dir vane arg dir.lon))]~
```

The result of the lookup on the previous line is adapted into a formatted text block with a head of `%tang` and different results depending on whether the request was `~` null or not.

#### Tutorial:  Deep Dive into `cat.hoon`

For instance, how does [+cat](urbit-docs/manual/os/dojo-tools#cat) work?  Let's look at the structure of `/gen/cat/hoon`:

<details>
<summary>/gen/cat.hoon</summary>

```hoon
::  ConCATenate file listings
::
::::  /hoon/cat/gen
  ::
/?    310
/+    pretty-file, show-dir
::
::::
  ::
:-  %say
|=  [^ [arg=(list path)] vane=?(%g %c)]
=-  tang+(flop `tang`(zing -))
%+  turn  arg
|=  pax=path
^-  tang
=+  ark=.^(arch (cat 3 vane %y) pax)
?^  fil.ark
  ?:  =(%sched -:(flop pax))
    [>.^((map @da cord) (cat 3 vane %x) pax)<]~
  [leaf+(spud pax) (pretty-file .^(noun (cat 3 vane %x) pax))]
?-     dir.ark                                          ::  handle ambiguity
    ~
  [rose+[" " `~]^~[leaf+"~" (smyt pax)]]~
::
    [[@t ~] ~ ~]
  $(pax (welp pax /[p.n.dir.ark]))
::
    *
  =-  [palm+[": " ``~]^-]~
  :~  rose+[" " `~]^~[leaf+"*" (smyt pax)]
      `tank`(show-dir vane pax dir.ark)
  ==
==
```

</details>

- What is the top-level structure of the [generator](urbit-docs/glossary/generator)?  (A [cell](urbit-docs/glossary/cell) of `%say` and the [gate](urbit-docs/glossary/gate), what Dojo recognizes as a `%say` generator.)

- Some points of interest include:
  - `/?` faswut pins the expected Arvo [kelvin version](urbit-docs/glossary/kelvin); right now it doesn't do anything.
  - `.^` [dotket](urbit-docs/language/hoon/reference/rune/dot#-dotket) loads a value from Arvo (called a ["scry"](urbit-docs/glossary/scry)).
  - [++smyt](urbit-docs/language/hoon/reference/stdlib/4m#smyt) pretty-prints a path.
  - `=-` [tishep](urbit-docs/language/hoon/reference/rune/tis#--tishep) combines a [faced](urbit-docs/glossary/face) noun with the [subject](urbit-docs/glossary/subject), inverted relative to `=+` [tislus](urbit-docs/language/hoon/reference/rune/tis#-tislus)/`=/` [tisfas](urbit-docs/language/hoon/reference/rune/tis#-tisfas).

You can see how much of the generator is concerned with formatting the content of the file into a formatted text `tank` by prepending `%rose` tags and so forth.

- Work line-by-line through the file and clarify parts that are muddy to you at first glance.

### Producing Error Messages

Formal error messages in Urbit are built of tanks.  “A `tang` is a [list](urbit-docs/glossary/list) of `tank`s, and a `tank` is a structure for printing data.  There are three types of `tank`: `leaf`, `palm`, and `rose`.  A `leaf` is for printing a single noun, a `rose` is for printing rows of data, and a `palm` is for printing backstep-indented lists.”

One way to include an error message in your code is the `~_` [sigcab](urbit-docs/language/hoon/reference/rune/sig#_-sigcab) rune, described as a “user-formatted tracing printf”, or the `~|` [sigbar](urbit-docs/language/hoon/reference/rune/sig#-sigbar) rune, a “tracing printf”.  What this means is that these print to the stack trace if something fails, so you can use either [rune](urbit-docs/glossary/rune) to contribute to the error description:

```hoon
|=  a=@ud
~_  leaf+"This code failed"
!!
```

When you compose your own library functions, consider including error messages for likely failure points.


##  `%ask` Generators

Previously, we introduced the concept of a `%say` [generator](urbit-docs/glossary/generator) to produce a more versatile form of standalone single computation than a simple naked generator ([gate](urbit-docs/glossary/gate)) allowed. Another elaboration, the `%ask` generator, takes things further.

We use an `%ask` generator when we want to create an interactive program that prompts for inputs as it runs, rather than expecting arguments to be passed in at the time of initiation.

This section will briefly walk through an `%ask` generator to give you a taste of how they work.  The [CLI app guide](urbit-docs/userspace/apps/guides/cli-tutorial) walks through the libraries necessary for working with `%ask` generators in greater detail.  We also recommend reading [~wicdev-wisryt's “Input and Output in Hoon”](https://urbit.org/blog/io-in-hoon) for an extended consideration of relevant input/output issues.

##### Tutorial:  `%ask` Generator

The code below is an `%ask` [generator](urbit-docs/glossary/generator) that checks if the user inputs `"blue"` when prompted [per a classic Monty Python scene](https://www.youtube.com/watch?v=L0vlQHxJTp0).  Save it as `/gen/axe.hoon` in your `%base` [desk](urbit-docs/glossary/desk).

```hoon
/-  sole
/+  generators
=,  [sole generators]
:-  %ask
|=  *
^-  (sole-result (cask tang))
%+  print    leaf+"What is your favorite color?"
%+  prompt   [%& %prompt "color: "]
|=  t=tape
%+  produce  %tang
?:  =(t "blue")
  :~  leaf+"Oh. Thank you very much."
      leaf+"Right. Off you go then."
  ==
:~  leaf+"Aaaaagh!"
    leaf+"Into the Gorge of Eternal Peril with you!"
==
```

Run the generator from the [Dojo](urbit-docs/glossary/dojo):

```hoon
> +axe

What is your favorite color?
: color:
```

Something new has happened.  Instead of simply returning something, your Dojo's prompt changed from `~your-urbit:dojo>` to `~your-urbit:dojo: color:`, and now expects additional input.  Let's give it an answer:

```hoon
: color: red
Into the Gorge of Eternal Peril with you!
Aaaaagh!
```

Let's go over what exactly is happening in this code.

```hoon
/-  sole
/+  generators
=,  [sole generators]
```

Here we bring in some of the types we are going to need from `/sur/sole` and gates we will use from `/lib/generators`. We use some special [runes](urbit-docs/glossary/rune) for this.

- `/-` [fashep](urbit-docs/language/hoon/reference/rune/fas#--fashep) is a Ford rune used to import types from `/sur`.
- `/+` [faslus](urbit-docs/language/hoon/reference/rune/fas#-faslus) is a Ford rune used to import libraries from `/lib`.
- `=,` [tiscom](urbit-docs/language/hoon/reference/rune/tis#-tiscol) is a rune that allows us to expose a namespace. We do this to avoid having to write `sole-result:sole` instead of `sole-result` or `print:generators` instead of `print`.

```hoon
:-  %ask
|=  *
```

This code might be familiar. Just as with their `%say` cousins, `%ask` generators need to produce a `cell`, the head of which specifies what kind of generator we are running.

With `|= *`, we create a [gate](urbit-docs/glossary/gate) and ignore the standard arguments we are given, because we're not using them.

```hoon
^-  (sole-result (cask tang))
```

`%ask` [generators](urbit-docs/glossary/generator) need to have the second half of the [cell](urbit-docs/glossary/cell) be a gate that produces a `sole-result`, one that in this case contains a `cask` of `tang`.  We use the `^-` [kethep](urbit-docs/language/hoon/reference/rune/ket#--kethep) rune to constrain the generator's output to such a `sole-result`.

A `cask` is a pair of a [mark](urbit-docs/glossary/mark) name and a [noun](urbit-docs/glossary/noun).  We previously described a `mark` as a kind of complicated [mold](urbit-docs/glossary/mold); here we add that a `mark` can be thought of as an Arvo-level [MIME](https://en.wikipedia.org/wiki/MIME) type for data.

A `tang` is a [list](urbit-docs/glossary/list) of `tank`, and a `tank` is a structure for printing data, as described above.  There are three types of `tank`: `leaf`, `palm`, and `rose`.  A `leaf` is for printing a single noun, a `rose` is for printing rows of data, and a `palm` is for printing backstep-indented lists.

```hoon
%+  print    leaf+"What is your favorite color?"
%+  prompt   [%& %prompt "color: "]
|=  t=tape
%+  produce  %tang
```

Because we imported [generators](urbit-docs/glossary/generator), we can access its contained gates, three of which we use in `axe.hoon`: `++print`, `++prompt`, and `++produce`.

- `print` is used for printing a `tank` to the console.

    In our example, `%+` [cenlus](urbit-docs/language/hoon/reference/rune/cen#-cenlus) is used to call the gate `++print`, with two arguments. The first argument is a `tank` to print.  The `+` here is syntactic sugar for `[%leaf "What is your favorite color?"]` that just makes it easier to write. The second argument is the output of the call to `++prompt`.

- `prompt` is used to construct a prompt for the user to provide input. The first argument is a tuple. The second argument is a gate that returns the output of a call to `++produce`. Most `%ask` generators will want to use the `++prompt` gate.

    The first element of the `++prompt` tuple/sample is a flag that indicates whether what the user typed should be echoed out to them or hidden. `%&` will produce echoed output and `%|` will hide the output (for use in passwords or other secret text).

    The second element of the `++prompt` sample is intended to be information for use in creating autocomplete options for the prompt. This functionality is not yet implemented.

    The third element of the `++prompt` sample is the [tape](urbit-docs/glossary/tape) that we would like to use to prompt the user. In the case of our example, we use `"color: "`.

- `produce` is used to construct the output of the generator. In our example, we produce a `tang`.

```hoon
|=  t=tape
```

Our gate here takes a `tape` that was produced by `++prompt`.  If we needed another type of data we could use `++parse` to obtain it.

The rest of this generator should be intelligible to those with Hoon knowledge at this point.

One quirk that you should be aware of, though, is that `tang` prints in reverse order from how it is created.  The reason for this is that `tang` was originally created to display stack trace information, which should be produced in reverse order.  This leads to an annoyance: we either have to specify our messages backwards or construct them in the order we want and then [++flop](urbit-docs/language/hoon/reference/stdlib/2b#flop) the `list`.
