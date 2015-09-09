# git-guppy [![NPM version](https://badge.fury.io/js/git-guppy.svg)](http://badge.fury.io/js/git-guppy) [![Build Status](https://travis-ci.org/therealklanni/git-guppy.svg?branch=master)](https://travis-ci.org/therealklanni/git-guppy) [![codecov.io](http://codecov.io/github/therealklanni/git-guppy/coverage.svg?branch=master)](http://codecov.io/github/therealklanni/git-guppy?branch=master) [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/therealklanni/git-guppy?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

> Simple git-hook integration for your gulp workflows.

guppy streamlines and extends your git-hooks by integrating them with your
[gulp](http://gulpjs.com) workflow. This enables you to have **gulp tasks that
run when triggered by a git-hook**, which means you can do cool things like
abort a commit if your tests are failing - using the same gulp tasks you already
have.

## Install

```bash
npm i git-guppy --save-dev
```

## Usage

### 1. Declaring hooks as gulp tasks

The first step is the most common to gulp users: you should define a new gulp
task which name matches whichever git-hook you want to be triggerable by git. For
instance, to define a *pre-commit* hook that runs your unit tests and then some
custom code you could do as follows:

```js
var gulp = require('gulp');

gulp.task('pre-commit', ['unit'], function () {
  // your pre-commit task's code.
});
```

### 2. Installing git-hooks

To define that your project implements a git-hook you can simply add the hook's
name to the introduced *package.json*'s property *guppy-hooks*, which should
always be an array such as follows:

```json
{
  "guppy-hooks": [
    "pre-commit",
    "post-commit"
  ]
}
```

After that, you can install *git-guppy* as a dependency of your project
(`npm install git-guppy --save-dev`) and have it's *postinstall* script automatically create the proper hook files inside *.git/hooks* directory.

> If you already had *git-guppy* installed as a dependency of your project
before this step, make sure to remove the package from the node_modules
directory and running `npm install` once again so to force NPM to run
*git-guppy*'s postinstall script, whcih is responsible for creating the hook
files mentioned above.


## Advanced usage

### gulp integration

guppy exposes a few simple methods to help you superpower your git-hooks with
gulp tasks. This is useful when you need to access some meta information from
a git process - such as files being commited, or the commiting message.

To access these methods, you need to initialize guppy by passing in your gulp
reference as follows:

```js
var gulp = require('gulp');
var guppy = require('git-guppy')(gulp);
```

#### guppy.src(*hookName*)

> Supported hooks: `applypatch-msg`, `commit-msg`, `pre-applypatch`, `pre-commit`,
`prepare-commit-msg`

Pass in the name of the desired git-hook and get back the related filenames.
This allows you to work with the source file directly, for example to modify a
commit-msg programmatically or lint changed files.

*Note for pre-commit and pre-applypatch this will give you the ***working-copy***,
not the indexed (staged) changes. If you want the indexed changes, use
`guppy.stream()` instead.*

```js
// contrived example
gulp.task('pre-commit', function () {
  return gulp.src(guppy.src('pre-commit'))
    .pipe(gulpFilter(['*.js']))
    .pipe(jshint())
    .pipe(jshint.reporter(stylish))
    .pipe(jshint.reporter('fail'));
});
```

#### guppy.src(*hookName, fn*)

> Supported hooks: all

If you pass the optional `fn` argument, it will be passed to `gulp.task()` as the
task callback, but the first argument will be the related filenames (or `null`,
if none) and a second optional argument may also be supplied (when applicable)
with any additional arguments received from the git-hook as an array. gulp will
provide its callback as the last argument.

```js
// less contrived example
gulp.task('pre-commit', guppy.src('pre-commit', function (filesBeingCommitted) {
  return gulp.src(filesBeingCommitted)
    .pipe(gulpFilter(['*.js']))
    .pipe(jshint())
    .pipe(jshint.reporter(stylish))
    .pipe(jshint.reporter('fail'));
}));

// another contrived example
gulp.task('pre-push', guppy.src('pre-push', function (files, extra, cb) {
  var branch = execSync('git rev-parse --abbrev-ref HEAD');

  if (branch === 'master') {
    cb('Don\'t push master!')
  } else {
    cb();
  }
}));
```

#### guppy.stream(*hookName*)

> Supported hooks: `applypatch-msg`, `commit-msg`, `pre-applypatch`, `pre-commit`,
`prepare-commit-msg`

Pass in the name of the git-hook to produce a stream of the related files.

*Note that depending on the git-hook, you may be acting on files that differ from
your working copy, such as those staged for commit (as with 'pre-commit' for
example), rather than the working copy. If you need to act on the working-copy
files, use `guppy.src()` instead.*

```js
gulp.task('pre-commit', function () {
  return guppy.stream('pre-commit')
    .pipe(gulpFilter(['*.js']))
    .pipe(jshint())
    .pipe(jshint.reporter(stylish))
    .pipe(jshint.reporter('fail'));
});
```

### Additional notes

For details on what arguments each git-hook receives and what result a non-zero
exit status would have, check the [git-scm docs](https://git-scm.com/docs/githooks).

## Author

**Kevin Lanni**

+ [github/therealklanni](https://github.com/therealklanni)
+ [twitter/therealklanni](http://twitter.com/therealklanni)

## License

MIT © Kevin Lanni
![](https://ga-beacon.appspot.com/UA-62782014-1/git-guppy/1.0?pixel)
