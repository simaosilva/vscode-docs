---
Order: 10
Area: references
TOCTitle: Document Selector
PageTitle: Document Selector
---

# Document Selectors

Extensions can filter their features based on document selectors by language, file type, and location. This topic discusses document selectors, document schemes, and what extensions authors should be aware about.

## Text documents not on disk

Not all text documents are stored on disk, for example, newly created documents. Unless specified, a document selector applies to **all** document types. Use the [DocumentFilter](https://code.visualstudio.com/docs/extensionAPI/vscode-api#DocumentFilter) `scheme` property to narrow down on certain schemes, for example `{ scheme: 'file', language: 'typescript' }` for TypeScript files that are stored on disk.

## Document selectors

The VS Code extension API combines language-specific features, like IntelliSense, with document selectors through the [DocumentSelector](https://code.visualstudio.com/docs/extensionAPI/vscode-api#DocumentSelector) type. They are an easy mechanism to narrow down functionality to a specific language.

The snippet below registers a [HoverProvider](https://code.visualstudio.com/docs/extensionAPI/vscode-api#HoverProvider) for TypeScript files and the document selector is the `typescript` language identifier string.

```ts
vscode.languages.registerHoverProvider('typescript', {
    provideHover(doc: vscode.TextDocument) {
        return new vscode.Hover('For *all* TypeScript documents.');
    }
})
```

A document selector can be more than just a language identifier and more complex selectors can use a [DocumentFilter](https://code.visualstudio.com/docs/extensionAPI/vscode-api#DocumentFilter) to filter based on the `scheme` and file location through a `pattern` path glob-pattern:

```ts
vscode.languages.registerHoverProvider({ pattern: '**/test/**' }, {
    provideHover(doc: vscode.TextDocument) {
        return new vscode.Hover('For documents inside `test`-folders only');
    }
})
```

The next snippet uses the `scheme` filter and combines it with a language identifier. The `untitled` scheme is for new files that have not yet been saved to disk.

```ts
vscode.languages.registerHoverProvider({ scheme: 'untitled', language: 'typescript' }, {
    provideHover(doc: vscode.TextDocument) {
        return new vscode.Hover('For new, unsaved TypeScript documents only');
    }
})
```

## Document scheme

The `scheme` of a document is often overlooked but is an important piece of information. Most documents are saved on disk and extension authors typically assume they are working with a file on disk. For example with a simple `typescript` selector, the assumption is **TypeScript files on disk**. However, there are scenarios where that assumption is too lax and a more explicit selector like `{ scheme: 'file', language: 'typescript' }` should be used.

The importance of this comes into play when features rely on reading/writing files from/to disk. Check out the snippet below:

```ts
vscode.languages.registerHoverProvider('typescript', { // 👎 too lax
    provideHover(doc: vscode.TextDocument) {
        const { size } = fs.statSync(doc.uri.fsPath); // ⚠️ what about 'untitled:/Untitled1.ts' or others?
        return new vscode.Hover(`Size in bytes is ${size}`);
    }
})
```

The hover provider above wants to display the size of a document on disk but it fails to check whether the document is actually stored on disk. For example, it could be newly created and not yet saved. The correct way would be to tell VS Code that the provider can only work with files on disk.

```ts
vscode.languages.registerHoverProvider({ scheme: 'file', language: 'typescript' }, { // 👍 only works with files on disk
    provideHover(doc: vscode.TextDocument) {
        const { size } = fs.statSync(doc.uri.fsPath);
        return new vscode.Hover(`Size in bytes is ${size}`);
    }
})
```

## Summary

Documents are usually stored on the file system, but not always: there are untitled documents, cached documents that Git uses, documents from remote sources like FTP, and so forth. If your feature relies on disk access, make sure to use a document selector with the `file` scheme.

## Next Steps

To learn more about VS Code extensibility model, try these topic:

* [Extension Manifest File](/docs/extensionAPI/extension-manifest.md) - VS Code package.json extension manifest file reference
* [Contribution Points](/docs/extensionAPI/extension-points.md) - VS Code contribution points reference