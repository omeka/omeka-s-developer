---
title: Registering a new module or theme with omeka.org
---

After you [register your GitHub repository](http://omeka.org/register/) as an Omeka addon we'll publish new versions of your addon if you follow these steps.

First, in your addon's GitHub repository, go to the releases page and click
"Draft a new release". Enter a tag name (it does not have to be your addon
version) and target branch. We recommend entering a release title (e.g. addon
version) and description (e.g. release notes, changelog, etc.), but it's not
required.

Then, on your own computer, create a ZIP archive of the packaged,
ready-for-release addon.

This ZIP file must be distinct from the automatically generated ZIP file that GitHub creates. It should remove things like git and tx settings.

We recommend using Git's `archive` command:

    $ git archive --output={AddonZipName}.zip --prefix={AddonDirName}/ {tagname}

You should include a `.gitattributes` file so the `git archive` will exclude things like language and other git files that should not be included in the archive. In most cases, it will look like this:

```
/.gitattributes export-ignore
/.gitignore export-ignore
/.tx/ export-ignore
/language/*.pot export-ignore
/language/*.po export-ignore
```

Note that the `{AddonDirName}` must match the directory name you initially
registered. The `{AddonZipName}` can be anything, but we recommend that it's the
same as `{AddonDirName}`. Afterwards, attach this ZIP file to the release as a
binary.

Then, to complete the process, click "Publish release". If everything checks
out, we'll register the release shortly. After we have added it to the registry, updates following the same procedure will be automatically updated. If you
subsequently set the release to be a prerelease or draft, we will remove the
release from our registry.

Note that we pull in your GitHub repository's README and publish it alongside
your addon versions. Make sure the README is ready for publication.

Before registring your releases we check that certain things are true:

- Your repository has releases
- The release is not a draft
- The release has an attached binary (asset)
- The release/asset combination is not already registered
- The asset has the .zip extension
- The asset is a ZIP file
- The ZIP file contains only one top-level directory
- The ZIP file top-level directory is named the same as the one you registered
- The ZIP file contains a .jpg thumbnail (themes only)
- The ZIP file contains an INI file at the correct path:
  - classic plugin: /plugin.ini
  - classic theme: /theme.ini
  - S module: /config/module.ini
  - S theme: /config/theme.ini
- The INI file has no parsing errors
- The INI file has an `[info]` section
- The INI file has a `version`

If all these things are true we register and publish the release. Keep in mind
that we do not check if the addon works as expected. It is your responsibility to
set your release to a tag or commit that's in working condition.
