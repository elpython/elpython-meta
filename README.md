# What is EL Python?

Right now, EL Python is just an idea in a README in a GitHub repo.
Whether it will ever become more than that remains to be seen :)

# What is the idea?

EL Python would be a CPython 3.x variant tailored for Enterprise Lifecycles. It would exist as a venue for
paid developers working for commercial redistributors to collaborate on:

- maintenance branches with even stricter backwards compatibility requirements than regular CPython
  maintenance branches
- long-lived maintenance branches that permit feature backports from later CPython feature releases 
- features of interest to enterprise Python users where it isn't clear yet if the associated maintenance
  burdens will be low enough for the CPython community project to reasonably accept
- provisional availability of new features that have been accepted for a future CPython feature release,
  but not yet published by the upstream project
 
By defining EL Python as a separate project from CPython, we'd be able to make it clear that we
*don't* consider it reasonable to ask volunteer community contributors to invest their personal time
in meeting the needs of large enterprise organisations, but we *do* consider it reasonable for
commercial Python redistributors to invest in providing a CPython variant that promises:

- near zero churn-related compatibility breaks in maintenance releases
- the availability of selected new features from later language versions without requiring
  mandatory rebuilds of extension modules or regeneration of `pyc` files
- support for enterprise features that are of questionable value in the context of educational
  use cases and smaller organisations

# How would EL Python be expected to affect the testing matrix for community Python projects?

Any software tested against a given CPython X.Y.0 release would be expected to work unmodified on the
corresponding EL Python branch. Software tested against a later CPython X.Y.Z release would normally
be expected to work against the corresponding EL Python release, as long as it was not impacted by
any bugs that had been fixed in CPython, but the fix hadn't been backported due to backwards
compatibility concerns that were deemed acceptable for CPython, but not for EL Python.

This means that for projects which already promptly drop support for older CPython versions when
new feature releases become available, or when a given maintenance branch is no longer receiving
security updates, EL Python would have no impact on the typical testing matrix.

Where EL Python may potentially becoming an interesting testing target for Python community projects
is in the following cases:

- when a given CPython maintenance branch is no longer receiving security updates, some projects
  may choose to start testing against the corresponding EL Python branch instead (assuming that
  branch has a longer lifecycle than the regular CPython one - that may not be the case for all
  branches)
- when a project wants or needs to adopt a feature from a later CPython release, and that feature
  has been backported to EL Python, some projects may choose to switch to only supporting the
  EL Python variant of that older release, and not the regular CPython release
 
The goal of this approach is that the existence of EL Python should never *increase* the testing
burden for any community project - if EL Python is added to a testing matrix, it should be
accompanied by the *removal* of the corresponding CPython branch (either because that branch is
no longer receiving security updates, or because it lacks a feature the project is now relying on).

By contrast, both previous proposals for making new CPython features available to end users without
waiting for the next CPython feature release had the side effect of potentially significantly
expanding build & test matrices (see the deferred [PEP 407](https://www.python.org/dev/peps/pep-0407/)
and the withdrawn [PEP 413](https://www.python.org/dev/peps/pep-0413/) for details)

# What is the proposed relationship with CPython?

*All* functional changes to EL Python should either appear in CPython first, or else be covered by a
draft PEP that has been submitted for consideration for a future version of CPython, and *python-dev*
has made the determination that it is suitable for inclusion as an EL Python feature, pending
further consideration as a potential mainline CPython feature.

To ensure the absence of churn-related compatibility breaks, EL Python may also end up including
opt-in behaviour toggles that don't appear in regular CPython releases.

# How else would EL Python differ from CPython?

One big difference would be that EL Python would be a source-only project: it would *not* provide
any prebuilt binary installers for any platform. Instead, the expectation would be that
redistributors would take the project and produce binaries in the formats of interest to them.

In addition, while EL Python would transparently consume extension modules and wheel files built
for CPython, a mechanism would need to be provided for publishers to indicate when their
artifacts were EL Python specific (i.e. they're depending on a feature backported from a later
Python release).

# What is the proposed relationship with the PSF?

We'd like EL Python to be a PSF backed project, just like CPython, and contributors to EL Python
would be required to sign the CPython CLA before their contributions can be accepted.

# What about Python 2?

While EL Python itself would be Python 3 only, the tauthon project is already pursuing a similar concept for Python 2.7: https://github.com/naftaliharris/tauthon
