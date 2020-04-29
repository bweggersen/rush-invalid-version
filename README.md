# Repro of issue with invalid peer dependency version

## Usage

```
git clone
rush update
```

It will fail with:

```
ERROR: Internal Error: The version '*' of peer dependency 'typescript' is invalid
You have encountered a software defect. Please consider reporting the issue to the maintainers of this application.
```

The dependencies of foo are:

```
"dependencies": {
  "ts-loader": "*",
  "typescript": "npm:@msfast/typescript-platform-resolution@3.8.2"
}
```

`ts-loader` has `typescript` as a peer dependency. Rush is not unpacking `npm:@msfast/typescript-platform-resolution@3.8.2` to its true version, which is `3.8.2` and using `semver.valid()` it's not considering `npm:@msfast/typescript-platform-resolution@3.8.2` to be valid.

## Debug

- Run `$ where rush` to figure out where Rush is installed. When I run it, I get `/usr/local/bin/rush`.
- Open `/usr/local/lib/node_modules/@microsoft/rush/node_modules/@microsoft/rush-lib/lib/logic/pnpm/PnpmProjectDependencyManifest.js` and set breakpoints on line 95 (I'm on version 5.23.0 of Rush).
- Run `$ node --inspect-brk /usr/local/bin/rush update` to run `rush update` with a debugger
- Attach to the node debugger process (this repo has been configured for debugging in VS Code, see config below).
- Observe how `dependencySpecifier.versionSpecifier` is set to `npm:@msfast/typescript-platform-resolution@3.8.2` which fails in `semver.valid`.

`lanuch.json` config in VS Code to attach to a node debug process.

```
{
  "version": "0.1.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach by Process ID",
      "processId": "${command:PickProcess}"
    }
}
```
