# rush-pnpm-patch-demo

This is a demo repo for `rush-pnpm patch` and `rush-pnpm patch-commit`

# Prerequisite

- Specify "pnpmVersion" >= "7.4.0" in rush.json.
- Move the content of "pnpmOptions" in rush.json to **common/config/rush/pnpm-config.json** and remove "pnpmOptions" in rush.json.

# Package Dependencies

- demo-a depends on `is-odd@3.0.1`
- demo-b depends on `is-odd@3.0.1`
- demo-c depends on `is-odd@2.0.0`

# Using pnpm patch in Rush.js

1. Start patch `is-odd@3.0.1` module 

Run `rush-pnpm patch is-odd@3.0.1`. It prints

```
You can now edit the following folder: /tmp/0aa7035fc4b05681226c6a4334b026c1/user
```

> Note: the folder path is different in different machine.

2. "patch" the `is-odd@3.0.1`

Open the temp folder, modify the code...

For example: `/tmp/0aa7035fc4b05681226c6a4334b026c1/user/index.js`

```javascript
// ...
module.exports = function isOdd(value) {
  console.log('123'); // <-- logout here
  const n = Math.abs(value);
  if (!isNumber(n)) {
    throw new TypeError('expected a number');
  }
  if (!Number.isInteger(n)) {
    throw new Error('expected an integer');
  }
  if (!Number.isSafeInteger(n)) {
    throw new Error('value exceeds maximum safe integer');
  }
  return (n % 2) === 1;
};
```

3. Commit the patches

Move to `demo-a` project folder, and run `patch-commit`

```shell
cd packages/demo-a
rush-pnpm patch-commit /tmp/0aa7035fc4b05681226c6a4334b026c1/user
```

> NOTE: patch-commit should be run under a project folder.  
> NOTE: replace the temp folder to your own folder path

The common/config/rush/pnpm-config.json, common/config/rush/pnpm-lock.yaml and common/pnpm/patches should be refreshed

4. Check it works

Run `rush rebuild --verbose`

> NOTE: patched dependency has the exactly same integrity of the raw dependency. It confuses rush build cache, so it's essential to invoke `rebuild` here to see the correct log output.

a. demo-a.build.log says

```
Invoking: node index.js 
123
demo-a true
```

It works for `is-odd@3.0.1` in `demo-a` ✅

b. demo-b.build.log says

```
Invoking: node index.js 
123
demo-b true
```

Because `demo-b` depends on `is-odd@3.0.1`, the patched `is-odd` is used by `demo-b` ✅

c. demo-c.build.log says

```
Invoking: node index.js 
demo-c true
```

`is-odd@2.0.0` in `demo-c` is irrelevant with the patched `is-odd` ✅

DONE. 👏 Remember to commit this change to Git.