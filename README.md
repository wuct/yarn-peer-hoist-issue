# yarn-peer-hoist-issue

# The issue

Let's say we are working in a big workspace project and all sub-projects are using react@15. We have the following setting in the root `package.json`:

```
"workspaces": {
  "packages": ["a", "b"]
}
```

After react@16 released, we want to migrate our project incrementally and we want do it for __b__ first, so we update the `workspaces` to:

```
"workspaces": {
  "packages": ["a", "b"],
  "nohoist": ["b/**"]
}
```

This setup can help us avoid getting different version of react in __b__. For example, if we are using react-redux@5 in both __a__ and __b__, the expected result will be:

```
├── node_modules
│   ├── react@15
│   └── react-redux@5
│     
├── a
│   └── node_modules
├── b
│   └── node_modules
│       ├── react@16
│       └── react-redux@5
```

However, the actual result is 

```
├── node_modules
│   ├── react@15
│   └── react-redux@5
│     
├── a
│   └── node_modules
├── b
│   └── node_modules
│       ├── react@16
│       └── react-redux@5
│           └── node_modules
│               └── react@15
```

where `b/node_modules/react-redux/node_modules/react` is wrong.

Notice that the `peerDependencise` in react-redux@5 is `"react": "^0.14.0 || ^15.0.0-0 || ^16.0.0-0"`.

# To reproduce

1. `yarn install`
2. `cat b/node_modules/react-redux/node_modules/react/package.json | grep version` and get `"version": "15.6.2",` 

# Env
yarn: 1.9.2
node: 8.11.2
npm: 5.6.0
os: macOS 10.13.5

