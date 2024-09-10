# URI Dereferencer

## Custom linked data services

Modules can extend this module to add their own linked data services.

You can create your own linked data service objects by writing them against the
following interface and adding them to the global `UriDereferencer` object.

### Service interface

```js
{
    /**
     * Get the name of this service.
     *
     * @return {string}
     */
    getName() {},
    /**
     * Get options for this service.
     *
     * @return {object}
     */
    getOptions() {},
    /**
     * Does this service recognize the URI?
     *
     * @param {string} uri The URI
     * @return {bool}
     */
    isMatch(uri) {},
    /**
     * Get the resource URL of a URI.
     *
     * The resource URL typically dereferences to an RDF file (JSON-LD, XML,
     * etc.) that represents the URI. Note that the service must have cross-
     * origin resource sharing (CORS) enabled.
     *
     * @param {string} uri The URI
     * @param {string} lang The requested language (optional)
     * @return {string}
     */
    getResourceUrl(uri, lang) {},
    /**
     * Get information about the resource in HTML markup.
     *
     * @param {string} uri The URI
     * @param {string} text The response text of the resource
     * @param {string} lang The requested language (optional)
     * @return {string}
     */
    getMarkup(uri, text, lang) {}
}
```

### Service options

Services can modify their options if they require special handling.

- `useProxy: {bool}`: (default `false`) Use the local proxy server if configured? This
is useful if the linked data service doesn't allow cross-origin HTTP requests (CORS).
Note that the global `UriDereferencer` object must have set a valid `proxyUrl`.

### Service implementation example

```js
UriDereferencer.addService({
    getName() {
        return 'My Linked Data Service';
    },
    getOptions() {
        // This example assumes this service requires a proxy server
        return {useProxy: true};
    }
    isMatch(uri) {
        return (null !== this.getMatch(uri));
    },
    getResourceUrl(uri, lang) {
        // This example assumes a JSON-LD resource. There are many possible
        // ways to dereference a URL, depending on the service (e.g. RDF/XML,
        // SPARQL endpoint). Use lang only for services that accept a language
        // paramater via the URL.
        const match = this.getMatch(uri);
        lang = lang ?? 'en';
        return `http://example.com/vocab/${match[1]}.jsonld?lang=${lang}`;
    },
    getMarkup(uri, text, lang) {
        // This example assumes JSON response text. You may need to parse the
        // response text differently (e.g. XPath for XML). Use lang only for
        // services that provide translations in their response content.
        const json = JSON.parse(text);
        return `
        <dl>
            <dt>Label</dt>
            <dd>${json['label']}</dd>
            <dt>Description</dt>
            <dd>${json['description']}</dd>
        </dl>`;
    },
    // Get the URI match (not part of the interface, but useful anyway).
    getMatch(uri) {
        return uri.match(/^https?:\/\/example\.com\/vocab\/(.+)$/);
    }
});
```
