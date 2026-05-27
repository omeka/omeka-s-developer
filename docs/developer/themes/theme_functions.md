# Theme Functions

You may have functionality that is specific to your theme and should not exist separately as a module. In these cases, you can write helper functions. To set up your helpers, you must define them in your `config/theme.ini`. They should be named using Pascal case and added to the `helpers` array.

```
[info]
name = "Theme Name"

...

helpers[] = "FirstHelper"
helpers[] = "SecondHelper"
```

Each helper function is defined in its own `.php` file within your theme:

```
- asset/
- config/
- helper/
    - FirstHelper.php
    - SecondHelper.php
- view/
- README.md
- theme.jpg
```

Within those files, you can define your functionality using the following template.

```
<?php 
namespace OmekaTheme\Helper;

use Laminas\View\Helper\AbstractHelper;

class FirstHelper extends AbstractHelper
{
    public function __invoke() 
    {
        // your code here

    }
}
```