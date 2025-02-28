## JSON Schema

You can contribute in a variety of ways. For a detailed tutorial, read [Scott Addie](https://twitter.com/Scott_Addie)'s [**Community-Driven JSON Schemas in Visual Studio 2015**](https://scottaddie.com/2016/08/02/community-driven-json-schemas-in-visual-studio-2015/) blog post.

1. Submit new JSON schema files
2. Add a JSON schema file to the [catalog](#catalog)
3. Modify/update existing schema files

[Versioning of schema](https://github.com/SchemaStore/schemastore/issues/197#issuecomment-229690162) files are handled by modifying the file name to include
the version number: _myschema-1.2.json_

When uploading a new schema file, make sure it targets a file that is commonly
used or has potential for broad uptake.

Keep single source of truth. Do not copy an external schema here, but point the catalog to the external schema.

If you don't have Visual Studio (using macOS or Linux?), you can check your modifications are fine by running:

```sh
make
```

### <a name="catalog"></a>Adding to catalog

After adding a schema file in `src/schemas`, register them in alphabetical order in the [schema catalog](src/api/json/catalog.json) by adding an entry corresponding to your schema:

```json
{
  "description": "Schema description",
  "fileMatch": ["list of well-known filenames matching schema"],
  "name": "Friendly schema name",
  "url": "https://json.schemastore.org/<schemaName>.json"
}
```

### Best practices

✔️ **Use** the lowest possible schema draft needed, preferably Draft v4, to ensure interoperability with as many supported editors, IDEs and parsers as possible.

✔️ **Use** [`base.json`][base] schema for `draft-07` and [`base-04.json`][base-04] for `draft-04` with some common types for all schemas.

:x: **Don't forget** add test files.

- Always be consistent across your schema: order properties and describe in the one style.
- Always use `$comment` to note about something to developers. You can refer to some issues here.
- Always use `title` when property type is an object to enhance editor experience which use
  this property to show errors (like VS Code).
- Always use `description`, `type`, `additionalProperties`.
  - Always set `additionalProperties` to `false` until documentation permits
    additional properties explicitly. That tool the JSON schema is created for
    can be changed in the future to allow wrong extra properties.
- Always use `minLength`/`maxLength`/`pattern`/etc for property values.
- Don't end `title`/`description` values with colon.
- Always use lower case with spaces between words for `title`-s to make expected object
  values look more like types in programming languages.
- Always explicitly state whether some setting is global for some tool or local
  for a project created with this tool. For instance if some settings is local
  then add `for the current <project-type>` at the end of the `description` like
  `Whether to ignore a theme configuration for the current site` for `Jekyll`.
- Always add documentation url to descriptions when available in the following
  format: `<description>\n<url>` like `"Whether to ignore a theme configuration for the current site\nhttps://jekyllrb.com/docs/configuration/options/#global-configuration"`.
- Don't add undocumented properties or features to the schema.

[base]: https://github.com/SchemaStore/schemastore/blob/master/src/schemas/json/base.json
[base-04]: https://github.com/SchemaStore/schemastore/blob/master/src/schemas/json/base-04.json

### Adding tests (for [local schemas](src/schemas/json) only)

To make sure that files are validated against your schema correctly (we strongly suggest adding at least one before creating a pull request):

1. Create a subfolder in [`src/test`](src/test) named as your schema file
2. Create one or more `.json`, `.yml`, `.yaml` or `toml` files in that folder

#### Adding negative tests

To make sure that invalid files fail to validate against your schema, use a subfolder in [`src/negative_test/`](src/negative_test) instead.

#### Run build locally

- Install [NodeJS](https://nodejs.org) (The minimum required NodeJS version "engines" is defined in [package.json](src/package.json))
- in subdirectory src/ run `npm run build` (to test a single schema, use `npm run build -- --SchemaName=<jsonFileName.json> default`)
- or optionally for macOS or Linux in the project root directory run `make`

If the build succeeds, your changes are valid and you can safely create a PR.

### Self-hosting schemas

If you wish to retain full control over your schema definition, simply register it in the [schema catalog](src/api/json/catalog.json) by providing a `url` pointing to the self-hosted schema file to the [entry](#catalog). Example on how to handle [multiple schema versions.](https://github.com/SchemaStore/schemastore/pull/2057#issuecomment-1024470105)

### How to `$ref` from `schema_x.json` to `schema_y.json`

- Both schemas must exist [locally](src/schemas/json) in SchemaStore.
- Both schemas must have the same draft (example draft-04)
- `schema_y.json` must have `id` or `$id` with this value `"https://json.schemastore.org/schema_y.json"`
- In `schema_x.json`, add ref to `schema_y.json`: `"$ref": "https://json.schemastore.org/schema_y.json#..."`
- In [schema-validation.json](src/schema-validation.json), in `"options": []` list add
  `"schema_x.json": {"externalSchema": ["schema_y.json"]}`

### Recommended Editor Extensions

This project uses [EditorConfig](https://editorconfig.org) to configure editor settings and [Prettier](https://prettier.io) to configure file formatting.
Please install the EditorConfig and Prettier extensions for your IDE or code editor if they are not natively supported.

### Validation mode

SchemaStore supports three types of schema validation mode.

- [Full strict mode](https://ajv.js.org/strict-mode.html) via AJV validator (SchemaStore default mode)
- Not fully strict mode via AJV validator. (The json filename is present in the `ajvNotStrictMode` list in [schema-validation.json](src/schema-validation.json))
- Validation via [tv4](https://github.com/geraintluff/tv4) (The json filename is present in the `tv4test` list in [schema-validation.json](src/schema-validation.json))

### Avoid common PR problems

- git merge conflict in catalog.json because you added the item to the end of the list instead of alphabetically.
- Prettier build server failed because the PR was created/push from an organization and not from your own account.
