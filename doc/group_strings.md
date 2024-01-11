# LED matrix configuration via special PIS group strings

## General things that you should know

### Special PIS classes and how they are parsed

The configuration of the matrix is done via the sepcial PIS group strings.
The installed main panel will load either the common special PIS class "LED_BERLIN" or the specific special PIS class with a suffix (e.g. "LED_BERLIN_2014") or both of them.

You can think of the group strings as a configuration file.
If both special PIS classes (the common one and the one with the suffix) are loaded, it is the same as one configuration file with the content of both, where the one with the suffix comes first.

The format used is based on the INI-format.
There are sections defined by a string in square brackets (`[` and `]`) and settings as key value pairs delimited by an equals sign (`=`).
Lines starting with a semicolon (`;`) are comments and are ignored.
One difference is, that everything is case sensitive.
The other difference is the terminus definition in the terminus and terminus template sections. They are given in XML inside those sections and therefore deviate from the key value pair format.

### Available sections

| section | content |
| --- | --- |
| [settings] | Global Settings |
| [local] | Settings for current special PIS only |
| [panel] | Panel settings |
| [font] | Adding custom fonts |
| [fontset] | Sets of fonts that are used on the panels |
| [auto_format:regex] | Automatically replace certain parts of terminus strings |
| [auto_format:rule] | Rules or conditions for automatically formatting the terminus |
| [linemod] | Modify line number and its format |
| [linecolor] | Configure RGB line panel |
| [terminus] | Add terminus that has no code in PIS |
| [terminustemplate] | Terminus template |

### Order of sections

There are some rules concerning the order of the sections that need to be considered.

Generally it is irrelevant whether configurations are defined before they are referenced or after that.
That means, for example, the fonts can be defined before the fontsets or after them.
However, the order of multiple sections of the same name (e.g. multiple `[panel]` definitions) are important, since they are applied only in specified conditions and the first match will be applied.

There are three variants of how exactly this works that are slightly different.

The first applies to the sections `[settings]` and `[local]`.
`[settings]` contains global settings.
The order of them is irrelevant but the first appearance of a setting will be applied. This means, that if there are two special PIS groups and a setting in `[settings]` is defined in both of them, the one of the special PIS with the suffix will be applied, since it comes first.
`[local]` contains settings that are applied only to the special PIS in which they are defined.
Multiple instances of those sections in the same special PIS group are possible, but still, if a setting is given mutliple times, the first one will be applied and all others are ignored.

The second applies to the sections `[auto_format:regex]` and `[auto_format:rule]`.
Each appearance define a regular expression or a rule. The script will work its way through the regular expressions first and then through the rules.
It does that in the same order in which they are defined and applies all of them until there is a parameter in a rule section that tells the script to stop after that rule.

The third variant applies to all other sections.
Those section contain parameters that define for which panel, line number or condition it should be applied or give an ID by which a font or a terminus is identified.
Here the first match will be used.<br>
For example if there is a `[panel]` section for panels with a height of 26 and a width of 48 **after** a `[panel]` section for a height of 26 and all widths, it will never be applied, since the one which is for all widths is also a match for the width of 48.
If you want the 26x48 panels to match the one, that is written for that, it needs to be defined before the other.

## [settings]

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| special_chars | boolean | false | Sets whether the special chars in the base PIS will be ignored (false) or not (true). |
| count_is_id | boolean | false | The panels that are installed in a vehicle are counted and can be identified by that. This can be used in the panel settings, the fontsets and in the terminus definitions to match those things onto a specific panel with the "count" parameter.<br><br>There are two ways how the panels can be counted:<br>1: Straight ahead in the order of the module slots.<br>2: All installed panel sizes separately. So if there are one 32x200, two 26x192 and one 26x48 installed, all panels get the number 1 except the second 26x192 which will get the number 2.<br><br>The second one is the default. Setting this parameter to true, the first one will be used.|
| auto_format | boolean | false | With this set to true, it is possible to automatically modify and format the terminus strings if there is no complex terminus definition.<br>String parts can be replaced or be printed smaller than the other, if the terminus has two lines.<br>The rules for that must be specified in the sections [auto_format:regex] and [auto_format:rule]. |
| start | string | | This can enable another boot process of the panels. Currently the following are available:<br>1: Switch on all LEDs, then print the version and other data.<br>2: Long startup process with cursor and stuff. Only available for panels with a height of 26 or more.<br><br>Default is no start process. |

## [local]

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| complex_fallback | boolean | false | Complex terminus definitions are given only for specific panels. If there is a panel installed in the vehicle that is not represented in a complex terminus definition, it can either be empty (except for the line number) or just use the default strings from the base PIS as fallback.<br>This parameter sets which behavior is used. If it is set to true, the default strings will be used. |
| complex_auto_format | boolean | false | If this is set to true, the default strings used as fallback will be automatically formatted. This only works if complex_fallback is set to true (otherwise there are no strings that can be modified) and the global auto_format is enabled. |

## [panel]

| Parameter | Type | Default | Use | Description |
| --- | --- | --- | --- | --- |
| width | integer | | match | The width of the panel to which these settings should be applied. If it is not set, it will always match. |
| height | integer | | match | The height of the panel to which these settings should be applied. If it is not set, it will always match. |
| content | integer | | match | The content of the panel to which these settings should be applied. If it is not set, it will always match.<br>0: line, 1: line,terminus, 2: terminus |
| type | integer | | match | The type of the panel to which these settings should be applied. If it is not set, it will always match.<br>3: LED white or amber, 4: Panel with RGB line module |
| count | integer | | match | The count or slot position (see [settings] parameter count_is_id) of the panel to which these settings should be applied. If it is not set, it will always match. |
| main_size | string | | match | Only apply these settings if the main module has this size.<br>The size is given as "\<width>x\<height>" without leading zeros (e.g. "240x32"). If it is not set, it will always match. |
| line_mode | integer | 0 | setting | 0: Line gets the space it needs, rest is terminus<br>1: Fixed area for line (and terminus)<br>2: Like fixed but expanded (and terminus reduced) if line does not fit inside |
| line_width | integer | 0 | setting | Width of line.<br>If line mode is 0 it sets the maximum width of the line which is used to select the font and spacing, so it should be set anyway in that case. Otherwise the panel would always use the smallest available line font with its default spacing (or with minimum spacing if line_stretch is true). |
| line_align | string | "left" | setting | Alignment of line inside the defined area.<br>If it is set to anything else than "left" while line mode is set to 0, the line mode will be overwritten with 1.|
| line_offset | integer | 0 | setting | Shifts the line string to the right if alignment is left or to the left is alignment is right.<br>Font selection and spacing both use `line_width - 2 * line_offset` as available space. |
| line_stretch | boolean | false | setting | If set to true, it allows the modification of the spacing. Otherwise the line always uses the default spacing of the font. |
| line_separation | integer | 0 | setting | Space between line and terminus and therefore also the position of the terminus.<br>Available terminus space is calculated as `panel_width - used_line_width - line_separation` with used_line_space depending on the setting of line_mode, line_width and actual line width.<br>If line_mode is 0, this is not used if the actual line width is 0. With line_mode 1 or 2 it is always used. |

## [font]

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| name | string | | The name of the font which is used to reference it in other sections. It must be given, otherwise the section will be ignored.<br>If the given name matches an already existing font, the settings of that font will be overwritten, which is usefull if you just want to modify an existing font without the need of creating a new font with setting all the parameters like user_id and content_id again. |
| user_id | integer | | The user id of the font. |
| content_id | integer | | The content id of the font. |
| spacing | integer | | This is the spacing that is defined in the content tool when creating the font. It will be subtracted from the all spacing settings throughout the whole matrix configuration (group strings, stop strings), so that you can always give the real spacing everywhere, ignoring what is defined in the font in LOTUS. |
| height | integer | | The height of the font. In most cases this will be the height that is defined in the content tool when creating the font.<br>However, this is used in the font selection process and the positioning of the string, so you can influence that by modifing this value. The same goes for the following values. |
| baseline | integer | | The distance of the baseline from the top of the font. |
| mean_line | integer | | The distance of the mean line from the top of the font. |
| cap_line | integer | | The distance of the cap line from the top of the font. |
| min_spacing | integer | | The minimum spacing of the font.<br>If this font is the last one in a set but it still does not fit inside the available space with the default spacing, this is the minimum allowed spacing. |
| default_spacing | integer | | The default spacing. This is used if the spacing is not modified (e.g. in line fonts when line_stretch is set to false) and during the font selection when checking whether the string fits inside the available space. |
| max_spacing | integer | | This is the maximum allowed spacing if there is enough space to stretch the string. |
| descender_string | string | | A list of chars that are lower than the baseline of the font, written without spaces or delimiters (e.g. "gjpq"). |

## [fontset]

| Parameter | Type | Default | Use | Description |
| --- | --- | --- | --- | --- |
| width | integer | | match | The width of the panel to which these settings should be applied. If it is not set, it will always match. |
| height | integer | | match | The height of the panel to which these settings should be applied. If it is not set, it will always match. |
| content | integer | | match | The content of the panel to which these settings should be applied. If it is not set, it will always match.<br>0: line, 1: line,terminus, 2: terminus |
| type | integer | | match | The type of the panel to which these settings should be applied. If it is not set, it will always match.<br>3: LED Mono, 4: Panel with RGB line module |
| count | integer | | match | The count or slot position (see [settings] parameter count_is_id) of the panel to which these settings should be applied. If it is not set, it will always match. |
| main_size | string | | match | Only apply these settings if the main module has this size.<br>The size is given as "\<width>x\<height>" without leading zeros (e.g. "240x32"). If it is not set, it will always match. |
| line | string | | setting | Comma separated list of font names from large to small. |
| terminus1 | string | | setting | Comma separated list of font names from large to small if the terminus has only one line. |
| terminus2 | string | | setting | Comma separated list of font names from large to small if the terminus has two lines. |
| terminus2_small | string | | setting | Comma separated list of font names from large to small if the terminus has two lines and one of them is smaller.<br>This is the list for the smaller one. |
| terminus2_large | string | | setting | Comma separated list of font names from large to small if the terminus has two lines and one of them is smaller.<br>This is the list for the larger one. |

## [auto_format:regex]

| Parameter | Type | Default | Use | Description |
| --- | --- | --- | --- | --- |
| pattern | string | | match | Regular expression that should be replaced. |
| replace | string | | setting | Replacement for the regular expression. This will be applied to all appearances of the pattern in the terminus strings (except the ones from complex terminus definitions). |

## [auto_format:rule]

| Parameter | Type | Default | Use | Description |
| --- | --- | --- | --- | --- |
| string1_pattern | string | | match | Regular expression for the first terminus line. If it is not set, it will always match. |
| string2_pattern | string | | match | Regular expression for the second terminus line. If it is not set, it will always match. |
| if_no_prev_match | boolean | false | setting | If set to true, this rule is only applied, if no other rule was applied before this one. |
| break | boolean | false | setting | If set to true, this rule is the last one that is applied. |
| string1_replace | string | | setting | Replacement for the regular expression of the first terminus line. This is done before all other instructions in this section are applied. If it is not set, nothing will be replaced. |
| string2_replace | string | | setting | Replacement for the regular expression of the second terminus line. This is done before all other instructions in this section are applied. If it is not set, nothing will be replaced. |
| cat_strings | boolean | false | setting | If set to true, terminus line 1 and line 2 are merged into one string by concatenating them. |
| cat_if_enough_space | boolean | false | setting | If set to true, it will only concatenate the strings if there is enough space for the resulting string. The check uses the smallest available font with its default spacing.<br>It only applies if cat_strings is set to true. |
| cat_with_space | boolean | false | setting | If set to true, the strings are concatenated with a space between them.<br>It only applies if cat_strings is set to true. |
| smallerstring | integer | 0 | setting | With this, one string can be printed with a font from the terminus2_small list of the `[fontset]` while the other one will be printed with a font from the terminus2_large list.<br>It only applies, if the terminus has two lines and they are not concatenated into one.<br><br>0: Both string use the same size (terminus2 list of the `[fontset]`).<br>1: The first string uses terminus2_small, the second uses terminus2_large.<br>2: The second string uses terminus2_small, the first uses terminus2_large. |


## [linemod]

| Parameter | Type | Default | Use | Description |
| --- | --- | --- | --- | --- |
| line_pattern | string | | match | Regular expression of the line, to which this modification should apply. If it is not set, it will always match. |
| specialchar_pattern | string | | match | Regular expression of the special char code, to which this modification should apply. If it is not set, it will always match. |
| line_replace | string | | setting | Replacement for the regular expression of the line. |
| override_terminus | string | | setting | `id` of a [terminus] that will be used instead of the terminus that is received from the bord computer. |
| shift_terminus | integer | 0 | setting | Only applies with `line_mode = 1` in the `[panel]` section.<br>This shifts the whole terminus. Negative values will shift it to the left, positive values to the right.<br>It does not grant more space to the terminus, it just shifts it, as it is. |
| format_\<height*>\<type**>\_align | string | "left" | setting | This overwrites `line_align` of the `[panel]` section. |
| format_\<height*>\<type**>\_offset | integer | 0 | setting | This overwrites `line_offset` of the `[panel]` section. |
| format_\<height*>\<type**>\_spacing | integer | | setting | This disables `line_stretch` of the `[panel]` section and sets a fixed spacing. |

\* height = panel height<br>
\** type = panel shows line and terminus (f) or only line (l)<br>
example: **format_26l_align**, if the panel height is 26 and it shows only the line.

## [linecolor]

used by a module that is not yet released ;-)

| Parameter | Type | Default | Use | Description |
| --- | --- | --- | --- | --- |
| line_pattern | string | | match | Regular expression of the line, to which this configuration should apply. If it is not set, it will always match. |
| specialchar_pattern | string | | match | Regular expression of the special char code, to which this configuration should apply. If it is not set, it will always match. |
| color | string | 255 | setting | Color of the line string. This string must be a comma separated list of one to four integers from 0 to 255.<br>Examples:<br>"128": This sets the color to 128,128,128 with an alpha of 255.<br>"128,255": This does the same thing.<br>"128,128,255": This will set the color to 128,128,255 with an alpha of 255.<br>"128,128,128,255": This does the same thing as the first two.<br>The alpha will always be rounded to 0 or 255.|
| background_color | string | 0 | setting | The background color of the line field. It uses the same format as `color`. |
| font | string | | setting | This is a `name` of a `[font]`. It will be used together with `font_string_b` and `font_string_f` to set a background or a foreground image.<br>The font should have a height of 19px and the characters should have a maximum width of 32px. This fills the whole line field. |
| font_string_b | string | | setting | A single character or a string that contains the background image from the `font`. This will be printed behind the line string. |
| font_string_f | string | | setting | A single character or a string that contains the foreground image from the `font`. This will be printed in front of the line string. |
| hide_line | boolean | false | setting | Hide the line string. This is the same as if the alpha of the `color` is set to 0. |
| hide_background | boolean | false | setting | Hide the background. This is the same as if the alpha of the `background_color` is set to 0 and no `font_string_b` is set. |

## [terminus] and [terminustemplate]

Those have only the one parameter "id" which is a string, that identifies the terminus or the template.

This paramter is then followed by a terminus definition in the following 3 or more lines,
where the first line is `<terminus>` and the last one is `</terminus>`.
Those opening and closing tags of the terminus definition must be in separate lines, so that the group string parser can identify the section of the groups strings, where the definition starts and ends.

The difference between a terminus and a terminus template is, that the template can contain strings that are replaced when it is used.

Instead of writing a fixed string like "S+U Alexanderplatz", you can use a string like `$replaceme$`. The "\$" at the beginning and the end are mandatory. Those identify this as the placeholder that will be replaced, when referencing this template.
A placeholder with the same name can be used multiple times. They all will be replaced.
Multiple placeholders with different names are possible, too, of course (e.g. `$replaceme1$` and `$replaceme2$`).

The template is then used as

```xml
<terminus>
  <template id="mytemplate">
    <string id="replaceme1">S+U Alexanderplatz</string>
    <string id="replaceme2">via Potsdamer Platz</string>
  </template>
</terminus>
```

where `mytemplate` is the id of the template.