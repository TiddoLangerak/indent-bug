Repository demonstrating an off-by-one bug in the indentation of GitHub diffs.

## Bug TL;DR

In diffs on `github.com` the first tab indentation is one space too short. E.g. with the default tab-width of 8, the first
indentation is only 7 spaces from the `+` sign. This causes the code to be aligned differently on GitHub than in most
other IDEs, text editors, or `git` commands. For shorter tab-widths this can also make diffs difficult to read.

## The files
In this repository I have 2 folders, 2-space and 4-space, which contain a sample.txt file and an .editorconfig that sets the tab-width to 2 & 4 respectively (the `sample.txt` files are identical).
The sample.txt files contains 3 lines of text, with a textual column "rulers" to make it easier to see the bug. The exact content is as followed (tabs written explicitly):

```txt
12345678
not indented
12345678
\tsingle tab indent
12345678
\t\tsingle tab indent
```

## The bug
When viewing the code directly, the text is indented as expected. With manually expanded spaces the following should be shown:

[2-space/sample.txt](https://github.com/TiddoLangerak/indent-bug/blob/master/2-space/sample.txt):
```txt
12345678
not indented
12345678
  single tab indent
12345678
    single tab indent
```

[4-space/sample.txt](https://github.com/TiddoLangerak/indent-bug/blob/master/4-space/sample.txt):
```txt
12345678
not indented
12345678
    single tab indent
12345678
        single tab indent
```

As you can clearly see from the ruler, each tab indents the next line of text with the appropriate number of spaces.

However, in the diffs they're one space short. You'll see the following:

[2-space/sample.txt](https://github.com/TiddoLangerak/indent-bug/commit/df82f36fa3c49f4d34a26dd3c467d24a0d43c41d#diff-bdc044374a8c4619f348f79bbfa5bdf3):
```txt
+12345678
+not indented
+12345678
+ single tab indent
+12345678
+   single tab indent
```

[4-space/sample.txt](https://github.com/TiddoLangerak/indent-bug/commit/df82f36fa3c49f4d34a26dd3c467d24a0d43c41d#diff-a0a31aa657e773e50bb8855cb6293912):
```txt
+12345678
+not indented
+12345678
+   single tab indent
+12345678
+       single tab indent
```

As you can see, the indentations are one space shorter in the diff than they're supposed to be.

## Probable cause

The rendered HTML layout for the diffs seems to use actual tab characters for indentation, and sets its size using the CSS property `tab-size`. 
E.g. the "single tab indent" line uses the following HTML (tab written explicitly):

```html
<span class="blob-code-inner">+\tsingle tab indent</span>
```

However, contrary to what the name suggests, the `tab-size` property doesn't set the size of the tab character, but instead sets the distance
between tab *stops* (see [CSS standard](https://drafts.csswg.org/css-text-3/#tab-stop)). 
As a result, the first tab is NOT `tab-size` spaces width, but `[tab-size] - [distance-until-previous-tab-stop]` spaces. 
When there is a `+` as first character then the distance-until-previous-tab-stop is 1, and thus the tab is one space too short.

## Possible fixes:
Obviously I have no idea how your source code looks, so I can't tell how to fix the bug. However,
I did find a few ways that I could get the intended result by manually editing the DOM, so perhaps you will find these
findings useful:

- Expand tabs to spaces server-side
- Put the `+` and `-` signs in a separate `<td>` element.
- Add an extra space after the first tab to compensate for the + sign.
