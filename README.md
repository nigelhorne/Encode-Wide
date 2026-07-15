# NAME

Encode::Wide - Convert wide characters (Unicode, UTF-8, etc.) into HTML or XML-safe ASCII entities

# VERSION

0.06

# SYNOPSIS

    use Encode::Wide qw(wide_to_html wide_to_xml);

    my $html = wide_to_html(string => "Café déjà vu – naïve façade");
    # returns: 'Caf&eacute; d&eacute;j&agrave; vu &ndash; na&iuml;ve fa&ccedil;ade'

    my $xml = wide_to_xml(string => "Café déjà vu – naïve façade");
    # returns: 'Caf&#xE9; d&#xE9;j&#xE0; vu &#x2013; na&#xEF;ve fa&#xE7;ade'

# DESCRIPTION

Encode::Wide provides functions for converting wide (Unicode) characters into ASCII-safe
formats suitable for embedding in HTML or XML documents.
It is especially useful when dealing with text containing accented or typographic characters that need
to be safely represented in markup.

Other modules exist to do this,
however they tend to have assumptions on the input,
whereas this should work with UTF-8, Unicode, or anything that's common.

The module offers two exportable functions:

- `wide_to_html(string =` $text)>

    Converts all non-ASCII characters in the input string to their named HTML entities if available,
    or hexadecimal numeric entities otherwise.
    Common characters such as \`é\`, \`à\`, \`&\`, \`<\`, \`>\` are
    converted to their standard HTML representations like \`&amp;eacute;\`, \`&amp;agrave;\`, \`&amp;amp;\`, etc.

- `wide_to_xml(string =` $text)>

    Converts all non-ASCII characters in the input string to hexadecimal numeric entities.
    Unlike HTML, XML does not support many named entities, so this function ensures compliance
    by using numeric representations such as \`&amp;#xE9;\` for \`é\`.

# PARAMETERS

Both functions accept a named parameter:

- `string` — The Unicode string to convert.

# ENCODING

Input strings are expected to be valid UTF-8 or Unicode.
If a byte string is passed, the module will attempt to decode it appropriately.
Output is guaranteed to be pure ASCII.

# EXPORT

None by default.

Optionally exportable:

    wide_to_html
    wide_to_xml

## wide\_to\_html

### Purpose

Convert a Unicode / UTF-8 string into a pure-ASCII HTML fragment by replacing
every non-ASCII character with its named HTML entity (e.g. `&eacute;`) where
one exists, or a hexadecimal numeric entity (e.g. `&#xNNNN;`) otherwise.
Bare ampersands, angle brackets, and double-quotes are also escaped so the
result is safe to embed in an HTML attribute or body.

### Arguments

Named parameters (or a single positional string as the first argument):

- `string` (required)

    The text to encode.  May be a plain scalar or a reference to a scalar.
    Must be defined; passing `undef` causes the function to `die`.

- `keep_hrefs` (optional, default 0)

    When true, `<`, `>`, and `"` are **not** escaped,
    so that literal HTML hyperlinks embedded in the input survive intact.

- `keep_apos` (optional, default 0)

    When true, apostrophes (`'`) are **not** converted to `&apos;`.
    Useful when the result will appear inside a JavaScript string literal.

- `complain` (optional)

    A code reference invoked with a descriptive message if the pipeline
    encounters a character it cannot encode.  The function then `die`s with
    a `BUG:` prefix regardless.

### Returns

A defined scalar string containing only ASCII characters (code points 0x00–0x7F).

### Side Effects

Prints diagnostic information to STDERR and invokes the `complain` callback
when an unhandled character is detected.  Dies with `"BUG: wide_to_html(...)"`
in that case.

### Usage Example

    use Encode::Wide qw(wide_to_html);

    my $safe = wide_to_html(string => "na\x{EF}ve caf\x{E9}");
    # => 'na&iuml;ve caf&eacute;'

    my $href_safe = wide_to_html(string => '<a href="x">caf\x{E9}</a>', keep_hrefs => 1);
    # => '<a href="x">caf&eacute;</a>'

### API SPECIFICATION

#### Input

    {
        string     => { type => SCALAR | SCALARREF, required => 1, defined => 1 },
        keep_hrefs => { type => BOOLEAN, optional => 1, default => 0 },
        keep_apos  => { type => BOOLEAN, optional => 1, default => 0 },
        complain   => { type => CODEREF,  optional => 1 },
    }

#### Output

    { type => SCALAR, constraint => sub { $_[0] !~ /[^[:ascii:]]/ } }

### FORMAL SPECIFICATION

Let S be the input string, S' the output string.

    ∀ c ∈ S' : ord(c) ≤ 0x7F                          (ASCII guarantee)
    S = ""  ⟹  S' = ""                                 (empty pass-through)
    keep_hrefs = 0 ⟹ "<" ∉ S' ∧ ">" ∉ S' ∧ ∄ bare " in S'
    keep_apos = 0  ⟹ ∄ bare apostrophe in S'
    ¬∃ bare & in S' (& only appears as part of a valid entity)
    string = undef ⟹ die("Usage: wide_to_html() string not set")

## wide\_to\_xml

### Purpose

Convert a Unicode / UTF-8 string into a pure-ASCII XML fragment by replacing
every non-ASCII character with a hexadecimal numeric entity (e.g. `&#x00E9;`).
Unlike HTML, XML does not support most named entities, so only numeric forms
are used.  Em-dashes and en-dashes are folded to a plain ASCII hyphen (`-`).
Bare ampersands, angle brackets, and double-quotes are escaped for XML safety.

### Arguments

Named parameters (or a single positional string as the first argument):

- `string` (required)

    The text to encode.  May be a plain scalar or a reference to a scalar.
    Must be defined; passing `undef` causes the function to `die`.

- `keep_hrefs` (optional, default 0)

    When true, `<`, `>`, and `"` are **not** escaped.

- `complain` (optional)

    A code reference invoked with a descriptive message if an unhandled character
    is encountered.  The function then `die`s with a `BUG:` prefix regardless.

### Returns

A defined scalar string containing only ASCII characters (code points 0x00–0x7F).

### Side Effects

Prints diagnostic information to STDERR and invokes the `complain` callback
when an unhandled character is detected.  Dies with `"BUG: wide_to_xml(...)"`
in that case.

### Usage Example

    use Encode::Wide qw(wide_to_xml);

    my $safe = wide_to_xml(string => "SURN \x{017D}ganjar");
    # => 'SURN &#x17D;ganjar'

    my $href_safe = wide_to_xml(string => '<tag attr="val">', keep_hrefs => 1);
    # => '<tag attr="val">'

### API SPECIFICATION

#### Input

    {
        string     => { type => SCALAR | SCALARREF, required => 1, defined => 1 },
        keep_hrefs => { type => BOOLEAN, optional => 1, default => 0 },
        complain   => { type => CODEREF,  optional => 1 },
    }

#### Output

    { type => SCALAR, constraint => sub { $_[0] !~ /[^[:ascii:]]/ } }

### FORMAL SPECIFICATION

Let S be the input string, S' the output string.

    ∀ c ∈ S' : ord(c) ≤ 0x7F                          (ASCII guarantee)
    S = ""  ⟹  S' = ""                                 (empty pass-through)
    keep_hrefs = 0 ⟹ "<" ∉ S' ∧ ">" ∉ S' ∧ ∄ bare " in S'
    U+2013 ∈ S ⟹ "-" ∈ S' ∧ "–" ∉ S'                (en-dash collapsed)
    U+2014 ∈ S ⟹ "-" ∈ S' ∧ "—" ∉ S'                (em-dash collapsed)
    ¬∃ bare & in S' (& only appears as part of a valid entity)
    string = undef ⟹ die("Usage: string not set")

# SEE ALSO

- [Test Dashboard](https://nigelhorne.github.io/Encode-Wide/coverage/)
- [HTML::Entities](https://metacpan.org/pod/HTML%3A%3AEntities)
- [Encode](https://metacpan.org/pod/Encode)
- [XML::Entities](https://metacpan.org/pod/XML%3A%3AEntities)
- [Unicode::Escape](https://metacpan.org/pod/Unicode%3A%3AEscape)
- [https://www.compart.com/en/unicode/](https://www.compart.com/en/unicode/)

# SUPPORT

This module is provided as-is without any warranty.

Please report any bugs or feature requests to `bug-encode-wide at rt.cpan.org`,
or through the web interface at
[http://rt.cpan.org/NoAuth/ReportBug.html?Queue=Encode-Wide](http://rt.cpan.org/NoAuth/ReportBug.html?Queue=Encode-Wide).
I will be notified, and then you'll
automatically be notified of progress on your bug as I make changes.

# AUTHOR

Nigel Horne, `<njh at nigelhorne.com>`

# LICENCE AND COPYRIGHT

Copyright 2025-2026 Nigel Horne.

Usage is subject to the GPL2 licence terms.
If you use it,
please let me know.
