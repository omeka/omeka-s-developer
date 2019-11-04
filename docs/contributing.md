---
title: Contributing to Omeka S
---
## Code Style

Omeka development aims to adhere to the [PSR-2 style guide](http://www.php-fig.org/psr/psr-2), and uses the [git-flow](http://nvie.com/posts/a-successful-git-branching-model) branching model.

Contributions should aim for that standard.

Omeka S contains a `.php_cs` file that you can use with [PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) to clean up code both in core and modules. When using it with modules, you will want to add `.php_cs.cache` to your `.gitignore` file. When submitting contributions, we encourage you to fix style issues in a single distinct commit in your pull request.

## Tips for pull requests

We aim to adhere to the Zend Framework Coding Standards, but do not always meet that mark. It is therefore tempting to make style corrections during the process of submitting pull requests. We appreciate them, but those should be distinct from feature changes.

Please resist the (understandable) urge to commit style changes within the same commits that change features or functionality. That will help us differentiate the functionality changes in your submissions from the stylistic changes, and make it more likely that your pull requests will be quickly evaluated and accepted.

We use the `php-cs-fixer` library in our testing suite. After including functional changes, we recommend running that test with `gulp test` (or `gulp test:cs` to just run the code style tests) and applying the suggested changes in a distinct commit for the pull request.

## Helping with documentation

Documentation inevitably lags behind the actual code in some places. We welcome questions, corrections, and pull requests to the [documentation](https://github.com/omeka/omeka-s-developer/issues). 
