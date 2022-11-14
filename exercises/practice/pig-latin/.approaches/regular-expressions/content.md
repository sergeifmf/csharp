# Regular expressions

```csharp
using System.Text.RegularExpressions;

public static class PigLatin
{
    private static readonly Regex VowelRegex = new(@"(?<begin>^|\s+)(?<vowel>a|e|i|o|u|xr|yt)(?<rest>\w+)", RegexOptions.Compiled);
    private static readonly Regex ConsonantRegex = new(@"(?<begin>^|\s+)(?<consonant>ch|qu|thr|th|rh|sch|yt|\wqu|\w)(?<rest>\w+)", RegexOptions.Compiled);

    private const string VowelReplacement = "${begin}${vowel}${rest}ay";
    private const string ConsonantReplacement = "${begin}${rest}${consonant}ay";

    public static string Translate(string sentence) =>
        VowelRegex.IsMatch(sentence)
            ? VowelRegex.Replace(sentence, VowelReplacement)
            : ConsonantRegex.Replace(sentence, ConsonantReplacement);
}
```

This approach uses [regular expressions][regular-expressions] to match _and_ rewrite the input sentence.
So how do we do that and is the code still readable?
Let's find out!

If we examine the exercise's rules, there are basically two types of sentence:

1. Those with vowel sounds, where "ay" is added to the end of the word
2. Non consonant sounds, where first the consonant is moved to the end of the word and then "ay" is added to the end of the word

## Vowel sounds

Let's start out simple and try implement rule number 1:

> - **Rule 1**: If a word begins with a vowel sound, add an "ay" sound to the end of the word. Please note that "xr" and "yt" at the beginning of a word make vowel sounds (e.g. "xray" -> "xrayay", "yttria" -> "yttriaay").

Let's focus on "If a word begins with a vowel sound" first.
We can detect the vowels using the following pattern: `(a|e|i|o|u)`.
Next up is detect if the word _begins_ with a vowel sounds.
There are two ways in which a word can "begin":

1. It is the first word, which means that it matches directly after the beginning of the input ([`^` anchor][regex-anchors])
2. It is _not_ the first word, which means that it is preceded by one or more (`+` quantifier) whitespace characters ([`\s`][regex-whitespace-character-group]).

Finally, we need to capture the rest of the word (the letters after the vowel), which are one or more letters ([`\w`][regex-word-character-group]).

Putting this all together, we get:

```csharp
(^|\s+)(a|e|i|o|u)(\w+)
```

It is then easy to augment this with the additional requirement that consonant words can also start with `"xr"` or `"yt"`:

```csharp
(^|\s+)(a|e|i|o|u|xr|yt)(\w+)
```

<!--
- **Rule 4**: If a word contains a "y" after a consonant cluster or as the second letter in a two letter word it makes a vowel sound (e.g. "rhythm" -> "ythmrhay", "my" -> "ymay").

- **Rule 2**: If a word begins with a consonant sound, move it to the end of the word and then add an "ay" sound to the end of the word. Consonant sounds can be made up of multiple consonants, a.k.a. a consonant cluster (e.g. "chair" -> "airchay").
- **Rule 3**: If a word starts with a consonant sound followed by "qu", move it to the end of the word, and then add an "ay" sound to the end of the word (e.g. "square" -> "aresquay"). -->

[regular-expressions]: https://learn.microsoft.com/en-us/dotnet/standard/base-types/regular-expression-language-quick-reference
[regex-ismatch]: https://learn.microsoft.com/en-us/dotnet/api/system.text.regularexpressions.regex.ismatch
[regex-replace]: https://learn.microsoft.com/en-us/dotnet/api/system.text.regularexpressions.regex.replace
[regex-character-classes]: https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-classes-in-regular-expressions
[regex-word-character-group]: https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-classes-in-regular-expressions#word-character-w
[regex-whitespace-character-group]: https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-classes-in-regular-expressions#whitespace-character-s
[regex-anchors]: https://learn.microsoft.com/en-us/dotnet/standard/base-types/anchors-in-regular-expressions
