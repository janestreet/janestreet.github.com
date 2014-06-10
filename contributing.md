---
layout: default
---

# Contributing to Core

## Licensing

Jane Street requires that all contributors of code or documentation to
Jane Street's open-source libraries complete, sign, and submit an
[Individual Contributor License Agreement](http://blogs.janestreet.com/core-icla-1_1.pdf).
The purpose of this agreement is to clearly define the terms under
which intellectual property has been contributed to thereby allow us
to defend the project should there be a legal dispute regarding the
software, and to be flexible should licensing requirements change. A
signed CLA must be on file before an individual's contributions will
be accepted.

Corporations that have assigned employees to work on Core should
execute a
[Corporate Contributor License Agreement](http://blogs.janestreet.com/core-ccla-1_1.pdf).
covering intellectual property that may have been assigned as part of
an employment agreement. Individuals working for corporations that
have signed a CCLA still have to sign their own Individual CLA, to
cover any of their contributions which are not owned by their
employer.

You can submit your CLA by:

emailing a scan to <mailto:core-cla@janestreet.com> or sending a fax
to 917-746-6597

## Submitting a patch

Once you've submitted a CLA, here's how you go about contributing a
patch.

- create a pull-request for the project you want to contribute to (for
  example: [async](https://github.com/janestreet/async)).  The request
  will be used for discussion only, it won't be merged in the public
  repository in the end because this is not how we work internally.
- Make sure that your changes match our
  [coding standards](coding-standards.html).  Also, if you introduce
  any significant new functionality, you should make sure to include
  documentation and unit tests.  The unit tests should typically be
  done using the **pa_ounit** syntax extension.
- Someone at Jane Street will take responsibility for migrating the
  patch through our internal process, and communicating with you.
  When review and testing is complete, the next release will include
  your change.

## Release process

Internally we have a weekly release cycle for our code.  For each
release we export the changes to the public repositories and increase
the version numbers of all modified projects.
