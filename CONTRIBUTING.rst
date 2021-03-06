How To Contribute
=================


Reporting Bugs
--------------

- Try to include a reproducible example.
- Include which versions of devpi packages you use and whether you use any plugins.


Pull Requests
-------------

- Submit Pull Requests against the ``master`` branch.
- Provide a good description of what you're doing and why.
- Provide tests that cover your changes and try to run the tests locally first.
- If you are unsure on how to write a test, please ask and we will point you to
  existing tests which should help you writing new ones.
- Embrace ``git rebase`` to create "nice" commits and history.
- We strife for PEP8 compliance. New code should comply, old code must only
  comply when it's changed anyway. No unnecessary cleanups which would only
  make the PR harder to read.
- Add a changelog entry in the ``news`` folder of touched packages as a new file.
  Try to follow the style of existing entries.
  The leading dash will be generated by towncrier.
  You can test the output with ``towncrier --draft``.


Running tests
-------------

Create and activate a virtualenv for devpi and use ``dev-bootstrap.sh`` to
install Python packages needed for development and testing.

Run ``pytest <packagename>`` to run the tests of the package (client, common,
server, web etc) you are working on.

If you can, run ``tox`` in the directory of the changed packages to cover
different Python versions before making a PR.


Automated Testing
-----------------

Every PR is automatically tested and the status will be visible on the PR once
the tests ran.

The Windows Tests are known to fail sometimes.
