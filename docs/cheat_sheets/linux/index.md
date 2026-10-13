# Bash

## grep

Stop forgetting this...

```
cat -A binary_file | grep 
```

## Text Customizations

### Style String

|Style String|Reset String|Style Delimiter|
|-|-|-|
|`\e[<style>\m`|`\e[0m`|`;`|


### Basic Colors

#### Text

|Color|ANSI Code|HEX|High Intensity|ANSI Code|HEX|
|-|-|-|-|-|-|
|<span style="color: #000000; background-color: white;"> Black|30|#000000|<span style="color: #808080; background-color: white;">Black (Gray)|90|#808080|
|<span style="color: #CD0000;">Red</span>|31|#CD0000|<span style="color: #FF5555;">Red</span>|91|#FF5555|
|<span style="color: #00CD00;">Green</span>|32|#00CD00|<span style="color: #55FF55;">Green</span>|92|#55FF55|
|<span style="color: #CDCD00;">Yellow</span>|33|#CDCD00|<span style="color: #FFFF55;">Yellow</span>|93|#FFFF55|
|<span style="color: #0000EE;">Blue</span>|34|#0000EE|<span style="color: #5555FF;">Blue</span>|94|#5555FF|
|<span style="color: #CD00CD;">Magenta</span>|35|#CD00CD|<span style="color: #FF55FF;">Magenta</span>|95|#FF55FF|
|<span style="color: #00CDCD;">Cyan</span>|36|#00CDCD|<span style="color: #55FFFF;">Cyan</span>|96|#55FFFF|
|<span style="color: #E5E5E5;">White</span>|37|#E5E5E5|<span style="color: #FFFFFF;">White</span>|97|#FFFFFF|

#### Background
|Color|ANSI Code|HEX|High Intensity|ANSI Code|HEX|
|-|-|-|-|-|-|
|<span style="background-color: #000000;"> Black|40|#000000|<span style="background-color: #808080;">Black (Gray)|100|#808080|
|<span style="background-color: #CD0000;">Red</span>|41|#CD0000|<span style="background-color: #FF5555;">Red</span>|101|#FF5555|
|<span style="background-color: #00CD00;">Green</span>|42|#00CD00|<span style="background-color: #55FF55;">Green</span>|102|#55FF55|
|<span style="background-color: #CDCD00;">Yellow</span>|43|#CDCD00|<span style="background-color: #FFFF55;">Yellow</span>|103|#FFFF55|
|<span style="background-color: #0000EE;">Blue</span>|44|#0000EE|<span style="background-color: #5555FF;">Blue</span>|104|#5555FF|
|<span style="background-color: #CD00CD;">Magenta</span>|45|#CD00CD|<span style="background-color: #FF55FF;">Magenta</span>|105|#FF55FF|
|<span style="background-color: #00CDCD;">Cyan</span>|46|#00CDCD|<span style="background-color: #55FFFF;">Cyan</span>|106|#55FFFF|
|<span style="background-color: #E5E5E5;">White</span>|47|#E5E5E5|<span style="background-color: #FFFFFF;">White</span>|107|#FFFFFF|

### Styles
|Text Style|Code|HTML|
|-|-|-|
|**Bold**|1|`<b>`|
|*Italic*|3|`<i>`|
|<span style="text-decoration: underline;">Underline</span>|4|`<span style="text-decoration: underline;">`|
|~~Strikethrough~~|9|`<s>`|

### Examples

|Description|Formatted|Output|
|-|-|-|
|High intensity yellow text|`echo -e \e[93mHello, world\e[0m`|<code style="color: #FFFF55;">Hello,world</code>|
|Red text on a yellow background|`echo -e "\e[31;43mHello, world\e[0m"`|<code style="color: #CD0000; background-color: #CDCD00;">Hello, world</code>|
|Bold green text on a magenta background|`echo -e "\e[1;92;45mHello, world\e[0m"`|<code style="color: #00CD00; background-color: #CD00CD; font-weight: bold;">Hello, world</code>|