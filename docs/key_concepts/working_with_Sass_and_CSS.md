---
title: Working with Sass and CSS
---


Omeka S's CSS is generated using Gulp and other Node packages. The necessary packages are installed with npm. From the top-level Omeka S directory:

```
npm install
```

For convenience, you may also want to install Gulp's CLI globally:

```
npm install -g gulp-cli
```

## Editing the Styles

**Do not edit the files in the `css` directory manually!** They are automatically generated and your changes will be overwritten. 

The base directory for the styles and other assets is `application/Omeka/asset`. The SCSS sourcefiles themselves are in the `sass` subdirectory, while the generated CSS appears in `css`. Other static assets like Javascript and local web fonts live in the assets directory also.

When you edit the SCSS sources, you need to compile them into actual CSS for the styles to be updated and applied. There are two main ways of doing this, both are commands run from the top-level Omeka S directory.

To compile the styles a single time, after making a change:

```
gulp css
```

Alternatively, you can have Gulp continually run and check for changes to the SCSS and recompile automatically:

```
gulp css:watch
```

## See Also

[Javascript Events](../reference/javascript_events.md)







