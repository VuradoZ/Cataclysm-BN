# Translating Cataclysm: BN

* [Translators](#translators)
  * [Getting Started](#getting-Started)
  * [Grammatical gender](#grammatical-gender)
  * [Tips](#tips)
* [Developers](#developers)
  * [Translation Functions](#translation-functions)
  * [`translation`](#translation)
  * [Recommendations](#recommendations)
* [Maintainers](#maintainers)

## Translators

The official location for translating Cataclysm: BN is the
[Transifex translations project][1].

Some of the currently supported languages are:

* Arabic
* Bulgarian
* Chinese (Simplified)
* Chinese (Traditional)
* Dutch
* Esperanto
* French
* German
* Italian (Italy)
* Japanese
* Korean
* Polish
* Portuguese (Brazil)
* Russian
* Serbian
* Spanish (Argentina)
* Spanish (Spain)
* Turkish

Don't see your language in the list above? You can add it into the project at
Transifex!

### Getting Started

To begin translating, head over the [translation project][1] and click on the
"Help Translate Cataclysm: BN" button.
This should take you to a page where you can either create a free account on
Transifex, or login using GitHub, Google+ or LinkedIn.

![Start translating](img/translating-start.png)

After you've created your account, return to the [translation project][1] and
click on the "Join team" button.
This will open a window where you can choose the language you are interested on
translating, so pick one and click the "Join" button.

![Join project](img/translating-join.png)

After this, the most straightforward thing to do is to reload the page,
which should redirect you to the translation project's dashboard.
Here, you can click the "Languages" link on the sidebar to see the list of
supported languages and the current progress of the translation effort.

Note that you can request for the inclusion of additional languages,
if the one you are interested in is not available on the list.

![Language list](img/translating-list.png)

From this list, you can click on the language of your choice, and then click on
the "Translate" to get started right away. Otherwise, you can click on any
other language and click on the "Join team" button, if you are interested in
translating for that language as well.

After clicking on the "Translate" button, you will be taken to the web editor.
To begin, you need to choose a resource to translate. Most of the in-game text
is contained in the `cataclysm-bright-nights` resource, so click on it to start.

![Choose a resource](img/translating-resource.png)

At this point, the editor should show you the list of text available for
translation, now you only need to click on the string you want to translate and
type your translation on the translation area on the right side of the screen.
Click on the "Save" button when you are satisfied with your translation.

![Web editor](img/translating-editor.png)

See [Transifex's documentation][2] for more information.

### Grammatical gender

For NPC dialogue (and potentially other strings) some languages may wish to
have alternate translations depending on the gender of the conversation
participants.  This two pieces of initial configuration.

1. The dialogue must have the relevant genders listed in the json file defining
   it.  See [the NPC docs](NPCs.md).
2. Each language must specify the genders it wishes to use via the translation
   of `grammatical gender list`.  This should be a space-separated list of
   genders used in this language for such translations.  Don't add genders here
   until you're sure you will need them, because it will make more work for
   you.  If you need different genders than are currently supported you must
   add them to the `all_genders` lists in `lang/extract_json_strings.py` and
   `src/translations.cpp`.

Having done this, the relevant dialogue lines will appear multiple times for
translation, with different genders specified in the message context.  For
example, a context of `npc:m` would indicate that the NPC participant in the
conversation is male.

Because of technical limitations, all supported genders will appear as
contexts, but you only need to provide translations for the genders listed in
`grammatical gender list` for your language.

Other parts of the game have various ad hoc solutions to grammatical gender, so
don't be surprised to see other contexts appearing for other strings.

### Tips

There are issues specific to Cataclysm: BN which translators should be aware of.
These include the use of terms like `%s` and `%3$d` (leave them as they are),
and the use of tags like `<name>`, which shouldn't be translated.

Information about these and any other issues specific to individual languages,
can be found in Cataclysm: BN's [language notes folder][3].

General notes for all translators are in `README_all_translators.txt`,
and notes specific to a language may be stored as `<lang_id>.txt`,
for example `de.txt` for German.

Cataclysm: BN has more than 14000 translatable strings, but don't be discouraged.
The more translators there are, the easier it becomes 😄.

## Developers

Cataclysm: BN uses [GNU gettext][4] to display translated texts.

Using `gettext` requires two actions:

* Marking strings that should be translated in the source code.
* Calling translation functions at run time.

Marking translatable string allows for their automatic extraction.
This process generates a file that maps the original string (usually in English)
as it appears in the source code to the translated string.
These mappings are used at run time by the translation functions.

Note that only extracted strings can get translated, since the original string
is acting as the identifier used to request the translation.
If a translation function can't find the translation, it returns the original
string.

### Translation Functions

In order to mark a string for translation and to obtain its translation at
runtime, you should use one of the following functions and classes.

String *literals* that are used in any of these functions are automatically
extracted. Non-literal strings are still translated at run time, but they won't
get extracted.

#### `_()`

This function is appropriate for use on simple strings, for example:

```c++
const char *translated = _( "text marked for translation" )
```

It also works directly:

```c++
add_msg( _( "You drop the %s." ), the_item_name );
```

Strings from the JSON files are extracted by the `lang/extract_json_strings.py`
script, and can be translated at run time using `_()`. If translation context
is desired for a JSON string, `class translation` can be used instead, which is
documented below.

#### `pgettext()`

This function is useful when the original string's meaning is ambiguous in
isolation. For example, the word "blue", which can mean either a color or an
emotion.

In addition to the translatable string, `pgettext` receives a context which is
provided to the translators, but is not part of the translated string itself.
This function's first parameter is the context, the second is the string to be
translated:

```c++
const char *translated = pgettext( "The color", "blue" );
```

#### `vgettext()`

Some languages have complex rules for plural forms. `vgettext` can be used to
translate these plurals correctly. Its first parameter is the untranslated
string in singular form, the second parameter is the untranslated string in
plural form and the third one is used to determine which one of the first two
should be used at run time:

```c++
const char *translated = vgettext( "%d zombie", "%d zombies", num_of_zombies );
```

#### `vpgettext()`

Same as `vgettext`, but allows to specify translation context.

```c++
const char *translated = vpgettext( "water source, not time of year", "%d spring", "%d springs", num_of_springs );
```

### `translation`

There are times when you want to store a string for translation, maybe with
translation context; Sometimes you may also want to store a string that needs no
translation or has plural forms. `class translation` in `translations.h|cpp`
offers these functionalities in a single wrapper:

```c++
const translation text = to_translation( "Context", "Text" );
```

```c++
const translation text = to_translation( "Text without context" );
```

```c++
const translation text = pl_translation( "Singular", "Plural" );
```

```c++
const translation text = pl_translation( "Context", "Singular", "Plural" );
```

```c++
const translation text = no_translation( "This string will not be translated" );
```

The string can then be translated/retrieved with the following code

```c++
const std::string translated = text.translated();
```

```c++
// this translates the plural form of the text corresponding to the number 2
const std::string translated = text.translated( 2 );
```

`class translation` can also be read from JSON. The method `translation::deserialize()`
handles deserialization from a `JsonIn` object, so translations can be read from
JSON using the appropriate JSON functions. The JSON syntax is as follows:

```JSON
"name": "bar"
```

```JSON
"name": { "ctxt": "foo", "str": "bar", "str_pl": "baz" }
```

or

```JSON
"name": { "ctxt": "foo", "str_sp": "foo" }
```

In the above code, `"ctxt"` and `"str_pl"` are both optional, whereas `"str_sp"`
is equivalent to specifying `"str"` and `"str_pl"` with the same string. Additionally,
`"str_pl"` and `"str_sp"` will only be read if the translation object is constructed using
`plural_tag` or `pl_translation()`, or converted using `make_plural()`. Here's
an example:

```c++
translation name{ translation::plural_tag() };
jsobj.read( "name", name );
```

If neither "str_pl" nor "str_sp" is specified, the plural form defaults to the
singular form + "s".

You can also add comments for translators by writing it like below (the order
of the entries does not matter):

```JSON
"name": {
    "//~": "as in 'foobar'",
    "str": "bar"
}
```

Do note that currently the JSON syntax is only supported for some JSON values,
which are listed below. If you want other json strings to use this format,
refer to `translations.h|cpp` and migrate the corresponding code. Afterwards
you may also want to test `update_pot.sh` to ensure that the strings are
correctly extracted for translation, and run the unit test to fix text styling
issues reported by the `translation` class.

| Supported JSON values
|---
| Effect names
| Item action names
| Item category names
| Activity verbs
| Gate action messages
| Spell names and descriptions
| Terrain/furniture descriptions
| Monster melee attack messages
| Morale effect descriptions
| Mutation names/descriptions
| NPC class names/descriptions
| Tool quality names
| Score descriptions
| Skill names/descriptions
| Bionic names/descriptions
| Terrain bash sound descriptions
| Trap-vehicle collision sound descriptions
| Vehicle part names/descriptions
| Skill display type names
| NPC dialogue u_buy_monster unique names
| Spell messages and monster spell messages
| Martial art names and descriptions
| Mission names and descriptions
| Fault names and descriptions
| Plant names in item seed data
| Transform use action messages and menu text
| Template NPC names and name suffixes
| NPC talk response text
| Relic name overrides
| Speech text
| Tutorial messages
| Vitamin names
| Recipe blueprint names
| Recipe group recipe descriptions
| Item names (plural supported) and descriptions
| Recipe descriptions
| Inscribe use action verbs/gerunds
| Monster names (plural supported) and descriptions
| Snippets
| Bodypart names
| Keybinding action names
| Field level names

### Recommendations

In Cataclysm: BN, some classes, like `itype` and `mtype`, provide a wrapper
for the translation functions, called `nname`.

When an empty string is marked for translation, it is always translated into
debug information, rather than an empty string.
On most cases, strings can be considered to be never empty, and thus always
safe to mark for translation, however, when handling a string that can be empty
and *needs* to remain empty after translation, the string should be checked for
emptiness and only passed to a translation function when is non-empty.

Error and debug messages must not be marked for translation.
When they appear, the player is expected to report them *exactly* as they are
printed by the game.

See the [gettext manual][5] for more information.

## Maintainers

Several steps need to be done in the correct order to correctly merge and maintain the translation files.

There are scripts available for these, so usually the process will be as follows:

1. Download the translations in `.po` format.
2. Put them in `lang/incoming/`, ensuring they are named consistently with the files in `lang/po/`.
3. Run `lang/update_pot.sh` to update `lang/po/cataclysm-BN.pot`.
4. Run `lang/merge_po.sh` to update `lang/po/*.po`. (This is only used to test translations locally as the project now uses Transifex for translation)

    This will also merge the translations from `lang/incoming/`.

These steps should be enough to keep the translation files up-to-date.

To compile the .po files into `.mo` files for use, run `lang/compile_mo.sh`. It will create a directory in `lang/mo/` for each language found.

Also note that both `lang/merge_po.sh` and `lang/compile_mo.sh` accept arguments specifying which languages to merge or compile. So to compile only the translation for, say, Traditional Chinese (zh_TW), one would run `lang/compile_mo.sh zh_TW`.

After compiling the appropriate .mo file, if your system is using that language, the translations will be automatically used when you run cataclysm.

If your system locale is different from the one you want to test, the easiest way to do so is to find out your locale identifier, compile the translation you want to test, then rename the directory in `lang/mo/` to your locale identifier.

So for example if your local language is New Zealand English (en_NZ), and you want to test the Russian (ru) translation, the steps would be `lang/compile_mo.sh ru`, `mv lang/mo/ru lang/mo/en_NZ`, `./cataclysm`.


[1]: https://www.transifex.com/bn-team/cataclysm-bright-nights
[2]: https://docs.transifex.com/
[3]: ../lang/notes
[4]: https://www.gnu.org/software/gettext/
[5]: https://www.gnu.org/software/gettext/manual/index.html
