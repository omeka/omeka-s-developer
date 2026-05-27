# Contributing

## Security

The Omeka Team takes security very seriously. If you encounter a security issue,
please contact us at [security@omeka.org](mailto:security@omeka.org).

## Code Style

Omeka development aims to adhere to the [PSR-2 style guide](http://www.php-fig.org/psr/psr-2), and uses the [git-flow](http://nvie.com/posts/a-successful-git-branching-model) branching model.

Contributions should aim for that standard.

Omeka S contains a `.php_cs` file that you can use with [PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) to clean up code both in core and modules. When using it with modules, you will want to add `.php_cs.cache` to your `.gitignore` file. When submitting contributions, we encourage you to fix style issues in a single distinct commit in your pull request.

## Accessibility

The Omeka Team strives to adhere to the principles and guidelines for accessibility that are articulate in [Web Content Accessibility Guidelines 2.1](https://www.w3.org/WAI/standards-guidelines/wcag/). Contributions to the core and addons should also uphold those standards.

## Tips for pull requests

Generally, we are more likely to accept small, focused pull requests that address a single concern. Your code should be [self-documenting](https://en.wikipedia.org/wiki/Self-documenting_code). You should include comments where your code isn't self-evident. Your commit messages should be intelligible. You should accompany your pull request with a detailed account on what you're trying to achieve, what approach you're taking, and any other messages that you think would be helpful to us, the reviewers. If we accept your changes we must also maintain them, so it's important that your pull request is high quality and compelling.

We aim to adhere to the [Laminas Coding Style Guide](https://docs.laminas.dev/laminas-coding-standard/v2/coding-style-guide/), but do not always meet that mark. It is therefore tempting to make style corrections during the process of submitting pull requests. We appreciate them, but those should be distinct from feature changes.

Please resist the (understandable) urge to commit style changes within the same commits that change features or functionality. That will help us differentiate the functionality changes in your submissions from the stylistic changes, and make it more likely that your pull requests will be quickly evaluated and accepted.

We use the `php-cs-fixer` library in our testing suite. After including functional changes, we recommend running that test with `gulp test` (or `gulp test:cs` to just run the code style tests) and applying the suggested changes in a distinct commit for the pull request.

## Helping with documentation

Documentation inevitably lags behind the actual code in some places. We welcome questions, corrections, and pull requests to the [documentation](https://github.com/omeka/omeka-s-developer/issues).
