palisade
======

`palisade` is a command line tool for managing GitHub gists.

Based on [gist.rb][gist] by [@defunkt][defunkt], this tool helps you to manage a local copy of your gists.

After publishing files to gist.github.com, this tool will:

- automatically clone the gist repository to local
- index the content of your gist for code search
- fetch meta info (e.g. description, url) of the gist from GitHub and add them to `gists.list`.

[gist]: https://github.com/defunkt/gist
[defunkt]: https://github.com/defunkt

You can also use `palisade` to sync your gists (created and starred)
between gist.github.com and your machine.

Dependencies
------------

- curl
- git
- [gist.rb][gist]
- [jq](http://stedolan.github.io/jq/)

For Linux, BSD, etc, you also need `xclip` or `xsel`.
For Cygwin, you need putclip/getclip provided by cygutils-extra.
(Mas OS X users should be fine with the preinstalled pbcopy/pbpaste.)

Mac OS X users also need GNU versions of `sed` and `date`, a.k.a `gsed` and
`gdate`.

Note: `xsel` users should use `gist.rb` v4.1.2+, since there is [a bug bitting xsel users in previous versions][151].

[151]: https://github.com/defunkt/gist/pull/151

# rf_rgb

- [csearch](https://github.com/google/codesearch)

    To search gists on your local machine.
    If not available, fallbacks to `grep`.

- [legit](https://github.com/kennethreitz/legit)

    If available, invokes `legit sync` to sync gist repository.
    Legit will stash, fetch, rebase/merge, push, and unstash if necessary.

    The `develop` branch of legit allows configuration for merge policy:

        * The default smart merge (rebase when suitable)
        * Always merge, never rebase (since [21bb7ed])
        * Always rebase, never merge (since [252b1eb])
        * Fast forward merge only (since [4782928])

    If legit is not available,
    palisade will report dirty gist repositories (`DIRTY $gist_id`)
    when the environment variable `palisade_AUTO_COMMIT` does not exist,
    and will commit files automatically when `palisade_AUTO_COMMIT` exists.

[21bb7ed]: https://github.com/kennethreitz/legit/commit/21bb7edd081f9e47abec9b970b32f2814104d298
[252b1eb]: https://github.com/kennethreitz/legit/commit/252b1eb2cd1c0a8f223fa8022ed37752bd5d6cec
[4782928]: https://github.com/kennethreitz/legit/commit/478292899831c1da478490970bc5d4f66d117510

Install
-------

Note that the following instructions only install palisade itself.
You need to install its dependencies mentioned before yourself.

    git clone https://github.com/weakish/palisade.git
    cd palisade
    make install

- Edit `config.mk` if you do not want to install it to `/usr/local`.
- Compatible with both GNU and BSD make.

To uninstall:

```sh
; cd palisade
; make uninstall
```

You can also install/uninstall palisade via [basher].

[basher]: https://github.com/basherpm/basher

Usage
-----

### init

For the first time, you need to run `palisade init` to associate your GitHub account and configure the directory to store local copies of your gists.

After that, you may run `palisade sync` to fetch all your gists (created and starred) to local.

Warn: `sync` can only fetch up to 10 million gists for you. If you have more than 10 million gists, you need to modify the source of `palisade` to lift the limit.

### Configuration

`palisade_USE_HTTPS`: If you need to use https for some reason, set the env var `palisade_USE_HTTPS`, but please note this isn't necessarily more secure than ssh, it's just a different option in case your network blocks all traffic other than http/s.

`palisade_AUTO_COMMIT`: If you'd like the `sync` command to automatically commit any local changes you've made before pulling and pushing to gist.github.com, set the `palisade_AUTO_COMMIT` env var to anything.

### publish

Whenever you want to publish a gist, just use

    palisade description file.txt ...

This will create the gist with the provided description, clone the gist repo, and put the gistid to clipborad.

Note: you must provide gist description, otherwise `palisade` will fail.

Hint: `palisade` will pass all arguments to gist as `gist -c -o -d description ...`, so you can use other options that gist understands, e.g. `palisade description -P` will work.

If you've edited your gists at `gist.github.com` or local machine, without pull/push changesets, you can sync all your gists via `palisade sync`.

If you've deleted your gists at `gist.github.com`, after `palisade sync`, the directories of deleted gists at your local machine will be marked with a prefix `_`.

### search

Search all of your gists:

    palisade search regexp

If `codesearch` is installed, `regexp` is RE2 (nearly PCRE).
Otherwise it is ERE, a.k.a `grep -E`.

### export

Export a gist (available at local) to a git repository,
with its full history:

```sh
; cd git-repo-root
; palisade export gist_id sub_directory_name branch_name
```

The content of the gist will be exported to `sub_directory_name`,
and the merging message will use `branch_name`.

### migrate

From version 1.0.0, `palisade` uses a different storage structure.
If you have used `palisade <1.0.0`, then you need to run this command to migrate:

    palisade migrate


Storage
-------

    /path/to/your/gists
    |-- gists.list  # a list of all your gists (including meta info)
    |-- repo # git repositories of your gists
    |-- tree # working directory of your gist repositories
        |-- 123456 # an example of gist
        |-- _123567890 # an example of gist which you have deleted on gist.github.com
        |-- ...
    `-- .csearchindex # code search index (optional)


Contributing
------------

Send pull requests or issues at:

https://github.com/weakish/palisade

### Tips

Setting environment variable `palisade_DEBUG` to `true` (or any non-empty string) will enable debug mode (`set -x`).

