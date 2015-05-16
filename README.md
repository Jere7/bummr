# Bummr
The *Bummr* gem allows you to automatically update all gems which pass your
build in separate commits.

## Installation

```bash
$ gem install bummr
```

Add a file called `.bummr-build.sh` to the root of your git directory.

Here is a suggested `.bummr-build.sh` which will `bundle exec rake` 4 times:

`.bummr-build.sh`

```bash
#!/bin/sh
MAX_TRIES=4
COUNT=0
EXIT=0

while [ $COUNT -lt $MAX_TRIES ] && [ $EXIT -eq 0 ]; do
  git log --pretty=format:'%s' -n 1
  echo "\nRunning test suite... $COUNT of $MAX_TRIES"
  bundle exec rake
  let EXIT=$?
  let COUNT=COUNT+1
done

exit $EXIT
```

Commit this file and merge it to master before running `bummr update`!

## Usage:

- Create a new, clean branch off of master.
- Run `bummr update`

##### `bummr update`

- Finds all your outdated gems
- Updates them each individually, using `bundle update --source #{gemname}`
- Commits each gem update separately, with a commit message like:

`gemname, {0.0.1 -> 0.0.2}`

- Runs `git rebase -i master` to allow you the chance to review and make changes.
- Runs `bummr test`

##### `bummr test`

- Runs your build script (`.bummr-build.sh`).
- If there is a failure, runs `bummr bisect`.

##### `bummr bisect`

- `git bisect`s against master.
- Finds the bad commit and attempts to remove it.
- Logs the bad commit in `log/bummr.log`.
- Runs `bummr test`

## Notes

- Once the build passes, you can push your branch and create a pull-request!
- You may wish to `tail -f log/bummr.log` in a separate terminal window so you
  can see which commits are being removed.
- Bummr may not be able to remove the bad commit due to a merge conflict, in 
  which case you will have to remove it manually, continue the rebase, and 
  run `bummr test` again.

## Contributing

1. Fork it ( https://github.com/lpender/bummr/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Wanted

Here are some things I'd love to add to Bummr:

- Test coverage.
- Configuration options (for test script path, name of master branch, etc)

## Thank you!

Thanks to Ryan Sonnek for the [Bundler
Updater](https://github.com/wireframe/bundler-updater) gem.