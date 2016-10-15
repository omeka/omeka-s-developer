# Standards

Omeka development aims to adhere to the [Zend Framework 2 Coding Standards](https://zf2-docs.readthedocs.org/en/latest/ref/coding.standard.html) 
and uses the [git-flow](http://nvie.com/posts/a-successful-git-branching-model/) branching model.

Contributions should aim for that standard. 

## Tips for pull requests

We aim to adhere to the Zend Framework Coding Standards, but do not always meet that mark. It is therefore tempting to make style corrections during the process of submitting pull requests. We appreciate them, but those should be distinct from feature changes.

Please resist the (understandable) urge to commit style changes within the same commits that change features or functionality. That will help us differentiate the functionality changes in your submissions from the stylistic changes, and make it more likely that your pull requests will be quickly evaluated and accepted.

We use the `php-cs-fixer` library in our testing suite. After including functional changes, we recommend running that test and applying the suggested changes in a distinct commit for the pull request.