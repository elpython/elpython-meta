# What is EL Python?

Right now, EL Python is just an idea in a README in a GitHub repo.
Whether it will ever become more than that remains to be seen :)

# What is the idea?

EL Python would be a CPython 3.x variant tailored for Enterprise Lifecycles. It would exist as a venue for
paid developers working for commercial redistributors to collaborate on:

- maintenance branches with even stricter backwards compatibility requirements than regular CPython
  maintenance branches
- long-lived maintenance branches that permit feature and performance improvement backports from
  later CPython feature releases 
- features of interest to enterprise Python users where it isn't clear yet if the associated maintenance
  burdens will be low enough for the CPython community project to reasonably accept
- provisional availability of new features that have been accepted for a future CPython feature release,
  but not yet published by the upstream project
 
By defining EL Python as a separate project from CPython, we'd be able to make it clear that we
*don't* consider it reasonable to ask volunteer community contributors to invest their personal time
in meeting the needs of large enterprise organisations, but we *do* consider it reasonable for
commercial Python redistributors and other organisations using Python to invest in providing a
CPython variant that promises:

- near zero churn-related compatibility breaks in maintenance releases
- the availability of selected new features from later language versions without requiring
  changes to file installation layouts, mandatory rebuilds of extension modules or
  regeneration of `pyc` files
- support for complex interpreter level features that are clearly beneficial for production
  infrastructure in large enterprise organisations, but may be of more questionable value in the
  context of smaller organisations, educational use cases, or research & development activities

# How would this idea be pursued?

If this idea ends up being pursued further, then this README would be redrafted in the form of
a Python Enhancement Proposal and submitted to python-dev for consideration.

Actually taking that step would be conditional on credible commitments of full-time developer
support from interested commercial Python redistributors and other organisations :)

# How often would new EL Python releases happen?

EL Python releases would be made on a six-monthly cadence, with the releases deliberately
timed to align with the development schedules of the Fedora and Ubuntu Linux distributions.

* Typical Ubuntu release dates: April, October
* Typical Ubuntu feature freeze dates: February, August
* Typical Fedora release dates: May, October
* Typical Fedora feature freeze dates: March, September

Given those dates, the target dates for EL Python releases would be the final week of January
and the final week of July each year.

# Which CPython branches would EL Python track?

By default, there would be an EL Python maintenance branch for each CPython maintenance
branch. These branches would all include a core set of patches that:

* added the EL Python version and variant information
* added any EL-Python-only features that had not yet been approved for inclusion in
  the CPython reference implementation (the ideal size of this patch set is for it to
  be empty, but it may be non-empty due to features that offer clear value to enterprise
  users, where python-dev is nevertheless collectively skeptical as to the value for
  broader audiences)

Each branch would also include a set of approved backports from later CPython maintenance and feature releases.

In addition, at the EL Python maintainers' discretion, some EL Python maintenance branches
may be defined as LTS branches, which would live on beyond the mainstream EOL of the
corresponding CPython branches.

# Who would have merge permissions for EL Python branches?

At least initially, EL Python merge permissions would be restricted to staff working
for commercial redistributors and other organisations formally participating in the
EL Python effort (where the exact nature of "formal participation" would be defined
in collaboration with the Python Software Foundation).

This is due to the fact that the proposed differences in maintenance policies between
EL Python and CPython are driven almost entirely by commercial and enterprise support
requirements, which gives EL Python a strongly business-driven character that isn't the
case for the more egalitarian, community-driven, CPython project.

It's expected that this restriction would eventually be dropped - it's just a
simplfying community management assumption to start out with.

# How would EL Python versions be identified?

It is proposed that EL Python versions be identified through a combination of semantic versioning
and calendar versioning:

* SemVer (X.Y.Z) for the base CPython version (e.g. 3.6.2)
* CalVer plus a patch release serial number (YY.MM.N) for EL Python releases (e.g. 18.01.0)

This information will be reported as:

* `sys.version_info` -> `sys.version_info(major=3, minor=6, micro=2, releaselevel='final', serial=0)`
* `sys.hex_version` -> `0x30602f0`
* `sys.implementation.version` -> `sys.version_info(major=18, minor=1, micro=0, releaselevel='final', serial=0)`
* `sys.implementation.hexversion` -> `0x120100f0`
* `sys.version` -> `3.6.2.18.01.0 (... other build details ...)`

To maximise backwards compatibility (and to ensure that otherwise CPython specific test cases
are run), `sys.implementation.name` will be reported as `"cpython"` in EL Python
branches, and the default CPython `sys.implementation.cache_tag` will also be re-used.

Even if no other markers were added, this would be sufficient to allow CPython and EL Python
builds to be fairly reliably distinguished based on:

    impl = sys.implementation
    if impl.name == 'cpython':
        if impl.version == sys.version_info:
            print("CPython")
        else:
            print("EL Python")

However, as a more robust marker that also allows for identification of other CPython
variants, an additional implementation-specific field, `variant` would be added:

* `sys.implementation.variant` -> `elpython`

This would be proposed as a new standard `sys.implementation` field for Python
3.9+ (defaulting to `reference` for reference implementations, such as CPython
itself for the ``sys.implementation.name == 'cpython'`` case).

Redistributors would be encouraged to always set it appropriately to help
identify their particular builds (using the `elpython-` prefix for EL Python
rebuilds).

# What does "no mandatory file regeneration" mean?

Migrating to a new CPython feature release necessarily means doing the following:

* reinstalling all components installed into `site-packages` to account for the new directory name
* regenerating all pyc files to use the new compatibility tag and (typically) magic number
* rebuilding any extension modules that targeted the regular CPython C API, rather than the limited
  API that offers greater assurances of ABI stability across feature releases
* regenerating all version-specific wheel files using the new version compatibility tag
* refreshing all virtual environments to account for the change in the path of `site-packages`,
  any changes to the standard library layout, and for the changes to `pyc` file and extension module
  naming schemes

These updates are required even if the affected software is otherwise entirely compatible with both
runtime versions at a source code level.

The "no mandatory rebuilds or file regeneration" requirement means that:

1. Any EL Python branch will *always* support the filesystem layout of the corresponding CPython
   X.Y.Z release
2. Any EL Python branch will *always* support loading `pyc` files and extension modules named with
   the compatibility tags for the corresponding CPython X.Y.Z release
3. Any EL Python branch will *always* support installing wheel files using the compatibility tags
   for the corresponding CPython X.Y.Z release

Features and fixes will thus only be considered as potential backport candidates from CPython to
EL Python if either:

* the original change didn't require an update to the filesystem layout or any of the file
  compatibility tags; or
* an alternate (potentially harder to maintain) implementation is available that doesn't require
  an update to the filesystem layout or any of the file compatibility tags; or
* the feature is deemed valuable enough to justify the complexity of enhancing EL Python itself
  to support new, EL Python specific, file compatibility tags, without losing support for the
  corresponding CPython X.Y.Z compatibility tags

# How would EL Python be expected to affect the testing matrix for community Python projects?

Any software tested against a given CPython X.Y.Z release would be expected to work unmodified on the
corresponding EL Python branch. Software tested against a later CPython X.Y.Z+ release would normally
be expected to work against the corresponding EL Python release, as long as it was not impacted by
any bugs that had been fixed in CPython, but the fix hadn't been backported due to backwards
compatibility concerns that were deemed acceptable for CPython, but not for EL Python.

This means that for projects which already promptly drop support for older CPython versions when
new feature releases become available, or when a given maintenance branch is no longer receiving
security updates, EL Python should have no impact on the typical testing matrix (and this is
considered a desirable outcome).

Where EL Python may potentially become an interesting testing target for Python community projects
is in the following cases:

- when a given CPython maintenance branch is no longer receiving security updates, some projects
  may choose to start testing against the corresponding EL Python branch instead of completely
  dropping support for that version (assuming that an EL Python branch exists for that version
  with a longer lifecycle than the regular CPython one - as noted above, that will only be the
  case for selected branches)
- when a project wants or needs to adopt a feature from a later CPython release, and that feature
  has been backported to EL Python, some projects may choose to switch to only supporting the
  EL Python variant of that older release, and not the regular CPython release
 
The goal of this approach is that the existence of EL Python should never *increase* the testing
burden for any community project - if EL Python is added to a testing matrix, it should typically
be accompanied by the *removal* of the corresponding CPython branch (either because that branch is
no longer receiving security updates, or because it lacks a feature the project is now relying on).

By contrast, both previous proposals for making new CPython features available to end users without
waiting for the next CPython feature release had the side effect of potentially significantly
expanding build & test matrices for community projects (see the deferred
[PEP 407](https://www.python.org/dev/peps/pep-0407/) and the withdrawn
[PEP 413](https://www.python.org/dev/peps/pep-0413/) for details)

# What is the proposed relationship with CPython?

*All* functional changes to EL Python should either appear in CPython first, or else be covered by a
draft PEP or feature request that has been submitted for consideration for a future version of CPython,
and *python-dev* has made the determination that it is suitable for inclusion as an EL Python feature,
pending further consideration as a potential mainline CPython feature.

To ensure the absence of churn-related compatibility breaks in EL Python maintenance releases,
EL Python may also end up including opt-in behaviour toggles that don't appear in regular CPython
feature releases (see PEP 493 for some examples of this kind of toggle related to backporting
the PEP 476 change to always verify HTTPS certificates by default).

To allow for transparent upgrades from a regular CPython build to EL Python, the directory paths
used in EL Python would necessarily be fully compatible with those used by CPython (e.g. the `/usr/lib`
and `/usr/lib` subdirectories would still be called `pythonX.Y`). However, EL Python may also
add additional EL Python specific directories that a regular CPython build will ignore.

# How else would EL Python differ from CPython?

One big difference would be that EL Python would be a source-only project: it would *not* provide
any prebuilt binary installers for any platform. Instead, the expectation would be that
redistributors and end users would take the project and produce their own binaries in the formats
of interest to them.

In addition, while EL Python would transparently consume pyc files, extension modules and wheel
files built for CPython, and would default to emitting CPython compatible versions of such
files, a mechanism would need to be provided for publishers to indicate when their
artifacts were EL Python specific (e.g. when they're depending on a feature backported from
a later CPython release).

# What is the proposed relationship with the PSF?

We'd like EL Python to be a PSF backed project, just like CPython, and contributors to EL Python
would be required to sign the CPython CLA before their contributions can be accepted.

We'd also expect any formal participation agreements to be worked out in collaboration with
the PSF, rather than directly between participating organisations.

# What Code of Conduct would the project use?

EL Python would operate under the Contributor Covenant: https://www.contributor-covenant.org/

# How would the CoC be enforced?

Initially, by the folks with merge privileges, with their respective employers and the PSF
Board available as potential paths of escalation for any CoC concerns relating to the project
maintainers themselves. EL Python would also respect any existing org-wide bans from PSF
provided communication channels.

Eventually, as the PSF defines clearer processes for reporting of CoC concerns for the
projects it oversees, EL Python would explicitly opt in to being covered by that process.

# What are the alternatives to pursuing the EL Python idea?

Historically, redistributors (most notably Red Hat, as part of Red Hat Enterprise Linux) have
offered long term commercial support for CPython simply by choosing a particular CPython X.Y.Z
release, and then offering that specific version for an extended period of time, focusing
mainly on security fixes, backports of requested bug fixes, and occasionally backports of
smaller features (e.g. backporting `collections.OrderedDict` from Python 2.7 to Python 2.6
in RHEL & CentOS 6).

An example of this approach can be seen by looking at the patch set for Python 2.7 in CentOS 7,
which relies on selective backports atop 2.7.5 rather than rebasing outright to upstream
maintenance releases: https://git.centos.org/rpms/python/blob/c7/f/SOURCES

Unfortunately, while this approach does address the key concern of providing a stable,
long-lived foundation for enterprise developers to build on, many other aspects of the
related developer experience are considered highly undesirable:

* enterprise developers targeting these long term support releases miss out on even fully
  backwards compatible enhancements added in later CPython feature releases, reducing
  the incentive for their employers to contribute to CPython maintenance & feature
  development (whether directly or indirectly via a commercial redistributor)
* enhancements to the language, reference implementation and standard library often take
  years to see truly broad adoption, as libraries and frameworks maintaining compatibility
  with these older CPython releases are also often unable to adopt them, even if the project
  maintainers themselves are regularly upgrading to newer CPython releases. (See
  [this 2015 post](https://alexgaynor.net/2015/mar/30/red-hat-open-source-community/) from
  Alex Gaynor, and
  [this reply](http://www.curiousefficiency.org/posts/2015/04/stop-supporting-python26.html)
  from Nick Coghlan for further discussion of that problem)
* the developers *maintaining* these long term support branches are currently working in
  organisation specific silos, making it difficult to ensure cross-platform consistency in
  the resulting Python developer experience
* even when selected features *are* backported to these existing LTS releases, this isn't
  communicated in a clear and consistent way, so Python developers may not be aware they
  can rely on those features being present on the platforms they care about

The EL Python concept arose from asking the question of how we might effectively address
the established commercial demand for a long term support variant of CPython in a way that:

* doesn't result in an inherently poor developer experience when targeting long-term support
  versions (especially later in their lifecycle)
* makes long term support versions (almost) as easy to develop and test against as regular
  CPython releases
* reduces the risk of fragmenting the Python developer experience across different competing
  long term support vendors
* makes it clear that offering long term support is considered the domain of professional
  software developers that are getting paid to meet that obligation, rather than something
  that can reasonably be expected of community volunteers

It is based in large part on the three tiered concept of "development", "stable", and
"long term" kernels in the Linux kernel development process:

* development kernels -> CPython development branch
* stable kernels -> CPython maintenance branches, short term EL Python maintenance branches
* long term kernels -> long term EL Python maintenance branches

# What about Python 2?

While EL Python itself is being proposed as a Python 3 only project, the `tauthon` project is
already pursuing a similar concept for Python 2.7: https://github.com/naftaliharris/tauthon

*If* EL Python were to ever gain a Python 2.7 branch, it would only be in the form of a post-2020
security-fix branch (including network security stack upgrades), rather than the kind of feature
backport branch being proposed for 3.x releases.
