Reviewing Patrole Code
======================
To start read the `OpenStack Common Review Checklist
<https://docs.openstack.org/infra/manual/developers.html#peer-review>`_


Ensuring code is executed
-------------------------
Any new test or change to an existing test has to be verified in the gate. This
means that the first thing to check with any change is that a gate job actually
runs it. Tests which aren't executed either because of configuration or skips
should not be accepted.


Execution time
--------------
Along with checking that the jobs log that a new test is actually executed,
also pay attention to the execution time of that test. Patrole already runs
hundreds of tests per job in its check and gate pipelines and it is important
that the overall runtime of the jobs be constrained as much as possible.
Consider applying the ``@decorators.attr(type='slow')``
`test attribute decorator`_ to a test if its runtime is longer than 30 seconds.

.. _test attribute decorator: https://docs.openstack.org/tempest/latest/HACKING.html#test-attributes


Unit Tests
----------
For any change that adds new functionality to common functionality unit tests
are required. This is to ensure we don't introduce future regressions and to
test conditions which we may not hit in the gate runs.


API Stability
-------------
Tests should only be added for published stable APIs. If a patch contains
tests for an API which hasn't been marked as stable or for an API which
doesn't conform to the `API stability guidelines
<https://wiki.openstack.org/wiki/Governance/Approved/APIStability>`_ then it
should not be approved.

Similarly, tests should only be added for policies that are covered by
`policy in code documentation
<https://specs.openstack.org/openstack/keystone-specs/specs/keystone/pike/policy-in-code.html>`_.
Any existing tests that test policies not covered by such documentation
are either:

* part of a service that has not yet migrated to policy in code; or
* legacy in the sense that they were created prior to policy in code

For the first bullet, the tests should not be considered stable, but should be
kept around to maintain coverage. These tests are a best-effort attempt at
offering RBAC test coverage for the service that has not yet migrated to
policy in code.

For the second bullet, the tests should be updated to conform to policy in
code documentation, if applicable.


Reject Copy and Paste Test Code
-------------------------------
When creating new tests that are similar to existing tests it is tempting to
simply copy the code and make a few modifications. This increases code size and
the maintenance burden. Such changes should not be approved if it is easy to
abstract the duplicated code into a function or method.


Tests overlap
-------------
When a new test is being proposed, question whether this feature is not already
tested with Patrole. Patrole has more than 600 tests, spread amongst many
directories, so it's easy to introduce test duplication.

Test Duplication
^^^^^^^^^^^^^^^^

Test duplication means:

* testing an API endpoint in more than one test
* testing the same policy in more than one test

For the first bullet, try to avoid calling the same API inside the
``self.rbac_utils.override_role`` call.

.. note::

    If the same API is tested against different policies, consider combining
    the different tests into only 1 test, that tests the API against all
    the policies it enforces.

For the second bullet, try to avoid testing the same policies across multiple
tests.

.. note::

    This is not always possible since policy granularity doesn't exist for all
    APIs. In cases where policy granularity doesn't exist, make sure that the
    policy overlap only exists for the non-granular APIs that enforce the same
    policy.


Being explicit
--------------
When tests are being added that depend on a configurable feature or extension,
polling the API to discover that it is enabled should not be done. This will
just result in bugs being masked because the test can be skipped automatically.
Instead the config file should be used to determine whether a test should be
skipped or not. Do not approve changes that depend on an API call to determine
whether to skip or not.


Release Notes
-------------
Release notes are how we indicate to users and other consumers of Patrole what
has changed in a given release. There are certain types of changes that
require release notes and we should not approve them without including a release
note. These include but aren't limited to, any addition, deprecation or removal
from the framework code, any change to configuration options (including
deprecation), major feature additions, and anything backwards incompatible or
would require a user to take note or do something extra.


Deprecated Code
---------------
Sometimes we have some bugs in deprecated code. Basically, we leave it. Because
we don't need to maintain it. However, if the bug is critical, we might need to
fix it. When it will happen, we will deal with it on a case-by-case basis.


When to approve
---------------
* Every patch needs two +2's before being approved.
* It's OK to hold off on an approval until a subject matter expert reviews it.
* If a patch has already been approved but requires a trivial rebase to merge,
  you do not have to wait for a second +2, since the patch has already had
  two +2's.
