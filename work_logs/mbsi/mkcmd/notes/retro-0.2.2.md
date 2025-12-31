# Build Self-Sufficiency Retro

## The Problem

Our CLI tool had a "dependency on itself" problem. The build process needed to read source code files at runtime to scaffold out templates. But when we published the package to npm, those source files weren't included - only the compiled JavaScript. This meant the tool couldn't work after installation because it couldn't find the templates it needed.

**Think of it like this:** A chef tries to cook a recipe that calls for "look in the pantry for ingredients," but someone moved the pantry to a different house. The recipe fails because the ingredients aren't where expected.

## The Chosen Solution

Instead of trying to ship source to NPM, we took a "embed the data" approach. We'd pre-generate all the template content and bake it directly into the compiled package. This way, the npm package is self-contained and doesn't depend on external files.

### Step 1: Crawl and Capture (`stringifier.ts`)

First, we built a crawler that walks through the `src/core` directory and reads every TypeScript file. Think of this like taking a snapshot of all our template code.

**Why:** This gives us one place that knows about all templates, making it easy to add new ones later.

### Step 2: Stringify the Code

For each file, we read its content as plain text (not code). The content becomes just a long string of characters.

**Why:** We want to store the file content *as data*, not execute it. This lets us reconstruct the files later.

### Step 3: Make It Safe for Code Storage

Here's the tricky part: we need to put this text into JavaScript code as a string literal. But what if the original code contains quotes or special characters? They'd break the string.

We had to escape all special characters:
- `"` becomes `\"`
- `\` becomes `\\`
- `\n` becomes `\\n`

**Why:** When the code runs, it needs to reconstruct the exact original text. Without escaping, a quote would prematurely close the string or a newline would become an actual line break instead of the characters `\` and `n`.

### Step 4: Wrap in Functions

We wrapped each escaped string in a function that reconstructs the original file using `FileBuilder`. The generated code looks like:

```typescript
const some_file_init = () => {
  const fb = new FileBuilder();
  fb.addLine("export function line(content: string) {");
  fb.addLine("  return ...");
  return fb.build();
};
```

**Why:** Functions give us a clean way to lazily reconstruct files. We only build what we need, when we need it.

### Step 5: Export as Data

All these functions get collected into an array, each paired with their original file path:

```typescript
const core = [
  { location: "./src/core/helpers/file-builder.ts", content: file_builder_init },
  { location: "./src/core/cli.ts", content: cli_init },
  // ...
];
export { core };
```

**Why:** This is our "map" of what exists and how to rebuild it. The scaffolding code just iterates over this array.

### Step 6: Scaffold from Data (`scaffold-core.ts`)

Finally, the scaffolding code imports the `core` array and rebuilds everything:
- For each entry, create the parent directory (with `mkdir -p` equivalent)
- Write the file by calling the content function

**Why:** This is a straightforward loop that doesn't need to know about the source directory structure. It just follows the map.

### Step 7: Prebuild Script

We added a `prebuild` step that runs the stringifier before every build. This ensures the `src/data/core.ts` file is always up-to-date.

**Why:** Developers shouldn't have to remember to manually regenerate the data file. Automation prevents bugs from stale data.

## Why This Approach?

### Self-Contained
The npm package has everything it needs built-in. No external file dependencies, no path resolution issues.

### Simple Consumer
Code that uses these templates (`scaffold-core.ts`) is very simple - just loop through an array and write files. It doesn't need to know about file crawling or path resolution.

### Maintainable
Adding a new template is as easy as dropping a file in `src/core`. The crawler picks it up automatically.

### Type-Safe
Since we generate TypeScript code, we get full type checking. If we change the template format, TypeScript will catch errors at build time.

### Efficient
Only rebuilds on build time, not at runtime.

## Tradeoffs

### Complexity vs. Simplicity
We traded the "simple" approach (read files at runtime) for a more complex build process. But this complexity is worth it because:
- Runtime is simpler and more reliable
- Errors caught at build time, not user runtime
- Package is truly self-contained

### Build Time
Builds take slightly longer because we regenerate data. But this only affects developers, not end users.

## What We Learned

1. **String escaping is harder than it looks** - escaping for code generation requires thinking about two levels: the code we generate, and the code *that* code will generate.
2. **Self-contained packages are worth the upfront work** - the confidence that "it just works" after installation is valuable.
3. **Automate the boring stuff** - the prebuild script ensures we never forget to regenerate data.
