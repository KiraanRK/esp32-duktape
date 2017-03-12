=================
Release checklist
=================

Checklist for ordinary releases
===============================

* Git branch naming note

  - ``vN.N.N-release-prep``: use this naming for bumping version number, etc.
    Merge to master before tagging release.

  - ``vN.N.N-release-post``: use this naming for bumping version number after
    release, checklist fixes after release, etc.

* Git maintenance

  - ensure git commits are up-to-date

  - ensure branches are merged, unused branches deleted (also remotely)

  - ensure branches are rebased where applicable

  - git fsck --full

  - git gc --aggressive

* Finalize DUK_VERSION

  - Change previous development version (with patch level 99) to release
    version

  - Verify by running Duktape cmdline and evaluating ``Duktape.version``

* Check dist-files/README.rst

* Ensure LICENSE.txt is up-to-date

  - Check year range

  - Also check ``util/create_spdx_license.py``

* Ensure RELEASES.rst is up-to-date

  - New release is in place

  - Release date is in place

* Ensure tests/api/test-all-public-symbols.c is up-to-date

  - Must add all new API calls

* Compilation tests:

  - Clean compile for command line tool with (a) no options and (b) common
    debug options (DUK_USE_DEBUG, DUK_USE_DEBUG_LEVEL=0, DUK_USE_DEBUG_PRINT=...,
    DUK_USE_SELF_TESTS, DUK_USE_ASSERTIONS)

  - Compile both from ``src`` and ``src-separate``.

  - Run ``mandel.js`` to test the the command line tool works.

  - Check that ``duk_tval`` is packed by default on x86 and unpacked on
    x64

  - util/checklist_compile_test.sh: linux compiler/arch combinations,
    run in dist, check output manually

  - Platform / compiler combinations (incomplete, should be automated):

    + Linux x86-64 gcc

    + Linux x86-64 gcc + -m32

    + Linux x86-64 clang

    + Linux x86-64 clang + -m32

    + FreeBSD clang

    + FreeBSD clang + -m32

    + Windows MinGW

    + Windows MinGW-w64

    + Windows MSVC (cl) x86

    + Windows MSVC (cl) x64

    + Windows Cygwin 32-bit

    + Windows Cygwin 64-bit

    + Linux MIPS gcc

    + Linux ARMEL gcc (little endian)

    + Linux gcc on some mixed endian ARM platform

    + Linux SH4 gcc

  - Check ``make duk-clang``, covers ``-Wcast-align``

* Test genconfig manually using metadata from the distributable

  - Ensure that Duktape compiles with e.g. ``-DDUK_USE_FASTINT`` genconfig
    argument

* Ecmascript testcases

  - **FIXME: semiautomate test running for various configurations**

  - On x86-64 (exercise 16-byte duk_tval):

    - make qecmatest   # quick test

    - VALGRIND_WRAP=1 make qecmatest  # valgrind test

  - On x86-32 (exercise 8-byte duk_tval)

    - make qecmatest   # quick test

  - Run testcases on all endianness targets

  - Run with assertions enabled at least on x86-64

* Run testcases with torture options

  - DUK_USE_GC_TORTURE

  - DUK_USE_SHUFFLE_TORTURE

  - DUK_USE_REFZERO_FINALIZER_TORTURE

  - DUK_USE_MARKANDSWEEP_FINALIZER_TORTURE + DUK_USE_GC_TORTURE

* Memory usage testing

  - Leaks are mostly detected by Valgrind, but bugs in valstack or object
    resize algorithms (or similar) can lead to unbounded or suboptimal
    memory usage

  - Minimal manual refcount leak test:

    - test-dev-refcount-leak-basic.js

* Performance testing

  - Check for unexpected performance regressions by compiling previous release
    and candidate release with ``-O2`` and running "make perftest" for them.

* API testcases

  - On x86-64:

    - make apitest

* Regfuzz

  - On x86-64, with DUK_USE_ASSERTIONS

    - make regfuzztest

* test262

  - on x86-64

    - make test262test

  - Run with assertions enabled at least on x86-64

* emscripten (run emscripten-generated code with Duktape)

  - on x86-64

    - make emscriptentest

* emscripten (compile Duktape with emscripten, run with Node)

  - on x86-64

    - make emscriptenduktest

* emscripten (compile Duktape with emscripten, run with Duktape)

  - on x86-64

    - make emscripteninceptiontest

* JS-Interpreter

  - on x86-64

    - make jsinterpretertest

* lua.js

  - on x86-64

    - make luajstest

* Debugger test

  - Test Makefile.dukdebug + debugger/duk_debug.js to ensure all files
    are included (easy to forget e.g. YAML metadata files)

  - Test JSON proxy

* Release notes (``doc/release-notes-*.rst``)

  - Write new release notes for release; needs known issues output from at
    least API, Ecmascript, and test262 test runs

  - Ensure instructions for upgrading from last release are correct

* Git release and tag

  - Tagging should be done before creating the candidate tar files so that
    "git describe" output will have a nice tag name.

  - This will be a preliminary tag which can be moved if necessary.  Don't
    push it to the public repo until the tag is certain not to move anymore.

  - There can be commits to the repo after tagging but nothing that will
    affect "make dist" output.

  - Make sure the tag is in the master commit chain, so that git describe will
    provide a useful output for dist packages built after the release

  - ``git tag -l -n1`` to list current tags

  - ``git tag -s -m "<one line release description>" vN.N.N`` to set tag

  - ``git tag -f -s -m "<one line release description>" vN.N.N`` to forcibly
    reset tag if it needs to be moved

* If release is a stable major/minor release (e.g. 1.1.0), create a maintenance
  branch ``vN.N-maintenance`` off the release tag.

* Build candidate tar.xz files

  - These should remain the same after this point so that their hash
    values are known.

  - Check git describe output from dist ``README.rst``, ``src/duktape.h``,
    ``src/duktape.c``, and ``src/duk_config.h``.  It should show the release
    tag.

  - This should be done in a fresh checkout to minimize chance of any
    uncommitted files, directories, etc affecting the build

* Check source dist contents

  - Check file list

  - Grep for FIXME and XXX

  - Trivial compile test for combined source

  - Trivial compile test for separate sources (important because
    it's easy to forget to add files in util/dist.py)

* Store binaries to duktape-releases repo

  - Add the tar.xz to the master branch

  - Create an independent branched named ``unpacked-vN.N.N`` with unpacked
    tar.xz contents

    + http://stackoverflow.com/questions/15034390/how-to-create-a-new-and-empty-root-branch

    + http://stackoverflow.com/questions/9034540/how-to-create-a-git-branch-that-is-independent-of-the-master-branch

  - Tag the final branch with ``vN.N.N``, push the tag, and delete the branch.
    The branch is not pushed to the server.

  - The concrete commands are packaged into ``add-unpacked.sh`` in
    duktape-releases repo.

* Update website downloads page

  - Release date

  - Link

  - Date

  - "latest" class

  - Release notes (layout and contents) for release

* Build website

  - Readthrough

  - Test that the Duktape REPL (Dukweb) works

  - Check duk command line version number in Guide "Getting started"

  - Diff website HTML against current website

* Upload website and test

* Final Git stuff

  - Ensure ``master`` is pushed and unnecessary branches are cleaned up

  - Push the release tag

  - Push the maintenance branch if created

* Make GitHub release

  - Release description should match tag description but be capitalized

  - Attach the end user distributable to the GitHub release

* Bump Duktape version for next release and testing

  - Set patch level to 99, e.g. after 0.10.0 stable release, set DUK_VERSION
    from 1000 to 1099.  This ensures that any forks off the trunk will have a
    version number easy to distinguish as an unofficial release.

  - ``src/duk_api_public.h.in``

Checklist for maintenance releases
==================================

* Make fixes to master and cherry pick fixes to maintenance branch (either
  directly or through a fix branch).  Test fixes in maintenance branch too.

* Update release notes and website in master.  **Don't** update these in
  the maintenance branch.

* Bump DUK_VERSION in maintenance branch.

* Check dist-files/Makefile.sharedlibrary; currently duplicates version
  number and needs to be fixed manually.

* Review diff between previous release and new patch release.

* Tag release, description "maintenance release" should be good enough for
  most patch releases.

* Build release.  Compare release to previous release package by diffing the
  unpacked directories.  The SPDX license can be diffed by sorting the files
  first and then using diff -u.

* Build website from master.  Deploy only ``download.html``.

  This is rather hacky: we need the release notes so the build must be made
  from master, but master may also contain website changes for the next
  release.
