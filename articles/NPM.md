# NPM

Some of those updates are major releases. Running npm update won’t update the version of those. Major releases are never updated in this way because they (by definition) introduce breaking changes, and npm want to save you trouble.

To update to a new major version all the packages, install the npm-check-updates package globally:

```bash
npm install -g npm-check-updates
```

then run it:

```bash
ncu -u
```

this will upgrade all the version hints in the package.json file, to dependencies and devDependencies, so npm can install the new major version.

You are now ready to run the update:

```bash
npm update
```

If you just downloaded the project without the node_modules dependencies and you want to install the shiny new versions first, just run

```bash
npm install
```