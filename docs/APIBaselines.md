# API Baselines

This document contains information regarding API baseline files and how to work with them.

## Add baseline files for new projects

When creating a new implementation (i.e. src) project, it's necessary to manually add API baseline files since API baseline are enabled by default. If the project is a non-shipping or test only project, add `<AddPublicApiAnalyzers>false</AddPublicApiAnalyzers>` to the project to disable these checks. To add baseline files to the project:

1. `cp .\eng\PublicAPI.empty.txt {new folder}\PublicAPI.Shipped.txt`
1. `cp .\eng\PublicAPI.empty.txt {new folder}\PublicAPI.Unshipped.txt`
1. Update AspNetCore.sln and relevant `*.slnf` file to include the new project
1. `{directory containing relevant *.slnf}\startvs.cmd`
1. F6 # or whatever your favourite build gesture is
1. Click on a RS0016 (or whatever) error
1. Right click in editor on underscored symbol or go straight to the “quick fix” icon to its left. Control-. also works.
1. Choose “Add Blah to public API” / “Fix all occurrences in … Solution”
1. Click Apply
2. F6 # again to see if the fixer missed anything or if other RS00xx errors show up (not uncommon)
3. Suppress or fix other problems as needed but please suppress (if suppressing) using attributes and not globally or with `#pragma`s because attributes make the justification obvious e.g. for common errors that can't be fixed
    `[SuppressMessage("ApiDesign", "RS0026:Do not add multiple public overloads with optional parameters", Justification = "Required to maintain compatibility")]`
    or
    `[SuppressMessage("ApiDesign", "RS0027:Public API with optional parameter(s) should have the most parameters amongst its public overloads.", Justification = "Required to maintain compatibility")]`

More information available in
-	https://github.com/dotnet/roslyn-analyzers/blob/master/src/PublicApiAnalyzers/PublicApiAnalyzers.Help.md
-	https://github.com/dotnet/roslyn-analyzers/blob/master/src/PublicApiAnalyzers/Microsoft.CodeAnalysis.PublicApiAnalyzers.md

## PublicAPI.Shipped.txt

This file contains APIs that were released in the last major version. This file should only be modified after a major release by the build team and should never be modified otherwise.

## PublicAPI.Unshipped.txt

This file contains new APIs since the last major version.

### New APIs

A new entry needs to be added to the PublicAPI.Unshipped.txt file for a new API. For example:

```
#nullable enable
Microsoft.AspNetCore.Builder.NewApplicationBuilder.New() -> Microsoft.AspNetCore.Builder.IApplicationBuilder!
```

### Removed APIs

A new entry needs to be added to the PublicAPI.Unshipped.txt file for a removed API. For example:

```
#nullable enable
*REMOVED*Microsoft.Builder.OldApplicationBuilder.New() -> Microsoft.AspNetCore.Builder.IApplicationBuilder!
```

### Updated APIs

Two new entry needs to be added to the PublicAPI.Unshipped.txt file for an updated API, one to remove the old API and one for the new API. This also applies to APIs that are now nullable aware. For example:

```
#nullable enable
*REMOVED*Microsoft.AspNetCore.DataProtection.Infrastructure.IApplicationDiscriminator.Discriminator.get -> string!
Microsoft.AspNetCore.DataProtection.Infrastructure.IApplicationDiscriminator.Discriminator.get -> string?
```

## Updating baselines after major releases

This will be performed by the build team using scripts at https://github.com/dotnet/roslyn/tree/master/scripts/PublicApi. The process moves the content of PublicAPI.Unshipped.txt into PublicAPI.Shipped.txt