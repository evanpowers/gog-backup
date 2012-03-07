==========
gog-backup
==========
---------------------------------------------------------
Download installers and extras for your GOG.com purchases
---------------------------------------------------------

:Author: Evan Powers
:Date: 2012-01-05
:License: GPLv3+
:Manual section: 1


Synopsis
--------

``gog-backup`` [--version] [--help] <command> [<args>]


Description
-----------

``gog-backup`` performs an unattended batch download of *everything*
you've purchased from GOG.com, both game installers and available
extras. When you make an additional purchase, just run ``gog-backup``
again to download only the new files necessary to make your backup
complete.


Features
--------

* Open source: The code is short and simple enough to be understood
  and/or customized without much effort.

* Cross platform: ``gog-backup`` is a pure Python program (as is its
  only dependency); it should run on any platform Python supports.

* Unattended batch download: Run one command to update your manifest,
  another to begin the download, and you can walk away.

* Resume partial downloads: Running ``gog-backup`` again after it
  aborts because of an error (or you terminate it yourself) will
  resume partial downloads where they left off.  You can also resume
  partial downloads begun by other tools, like your web browser or the
  official GOG.com downloader.

* Verify and repair files: GOG.com publishes XML containing MD5 hashes
  for game installers and "chunks" thereof; ``gog-backup`` can compare
  files against this information.  If one or more chunks of a file are
  corrupt, ``gog-backup`` will re-download only those chunks—not the
  entire file.  For game extras, which lack XML metadata but are
  always ``.zip`` archives, ``gog-backup`` can detect corruption by
  validating the checksums internal to the archive format.

* Multithreaded downloads: ``gog-backup`` will perform as many
  downloads in parallel as you specify.  You can choose whether all
  the threads should focus on one file until it is complete or should
  each download different files.

* Configurable directory hierarchy: ``gog-backup`` doesn't force you
  to organize your downloaded files in any particular way beyond
  putting all installers and extras for a particular game in the same
  directory.

* Covers and thumbnails: While this is disabled by default (because
  it's kinda pointless), ``gog-backup`` can download the box covers
  and thumbnails used to represent your games on the GOG.com website.


Installation
------------

Obtain the ``gog-backup`` script directly from its GitHub page
[1]_. There are exactly two dependencies:

1. Python 2.6 or compatible, unless I'm wrong and it actually requires 2.7
2. html5lib [2]_, tested with 0.90

If you're a Python developer,

    ``easy_install html5lib``

may be the easiest way to obtain the second dependency; everyone else
can just download the source archive from the aforementioned URL,
extract it, and move the sub-folder ``src/html5lib`` into the
directory containing ``gog-backup``.

Note that ``gog-backup`` itself doesn't need to be placed anywhere in
particular.


Basic Usage
-----------

While easy to use by virtue of having only a few sub-commands,
``gog-backup`` nevertheless has the flavor of a tool designed by a
programmer; its response to most problems is to die and print a
traceback. If you are not a programmer yourself, you'll need to bear
with this.

For an overview of command line syntax and the supported sub-commands,
run ``gog-backup --help``.

* ``login <email> [password]``

  Logs in to GOG.com using the supplied email and password, prompting
  for the latter if it is omitted, then saves the authentication
  cookie to disk for use during subsequent invocations. Neither the
  email nor password themselves are saved.

  While convienient for scripts, be aware that supplying your password
  on the command line may reveal it to other logged in users (it will
  appear in the output of ``ps``, for example).

* ``manifest``

  Parses your account pages on GOG.com to determine the complete list
  of games you own, then probes GOG.com's content servers for XML
  metadata and details like file size. (If enabled, it also downloads
  game covers and thumbnails.)

* ``list``

  Prints a summary of the current manifest—a list of game unique
  names, the corresponding game titles, the total size and number of
  both setup files and extras for each, and the total disk space
  required for the complete backup.

* ``compare`` and ``update``

  Verifies downloaded files by comparing them against available
  metadata from the manifest or internal checksums (in the case of
  ``.zip`` archives), then prints a report on what must be
  downloaded. Both commands update a cache of previous validation
  results; the difference is that ``compare`` re-validates everything,
  while ``update`` only considers files which have not been previously
  validated.

  ``compare`` is therefore useful for detecting corruption due to
  media wear or GOG.com posting an updated version of a file, but
  takes a long time if you own a lot of games.

* ``fetch``

  Downloads any missing, incomplete, or corrupted files (after an
  implicit ``update``). Failed HTTP requests are merely reported, not
  reattempted, so you should run ``fetch`` repeatedly until it stops
  downloading files (perhaps waiting an appropriate period between
  invocations if GOG.com is experiencing high load). By default, game
  files are placed in per-game sub-directories of the current working
  directory.

Therefore, the simplest command flow would be to first ``login``, then
download a ``manifest``, then ``fetch`` one or more times.

If you have previously downloaded files using another tool, such as
your web browser or the official GOG.com downloader, you needn't
download them again; if you place them either directly in the backup
directory or the sub-directory for the corresponding game, they'll be
detected. Partial downloads can be resumed in this way as well.


Advanced Usage
--------------

While ``gog-backup`` doesn't *require* any configuration to work out
of the box, there are a few things you can change if you feel the need
to do so. Most are controlled by a group of global variables
initialized near the top of the script; they include:

* BACKUPINTO: the directory into which all files, and ``gog-backup``
  state, are placed; by default, the current working directory
* CONCURRENCY: the number of download threads to use
* BREADTHFIRST: whether the download threads should generally work on
  different files (True) or the same file (False)
* FETCHCOVERS: whether to download covers and thumbnails

By default, files for a particular game are downloaded into a
sub-directory of BACKUPINTO named according to the unique name GOG.com
uses for the game; this unique name is merely the last component of
the URL for its description page. For example, ``beneath_a_steel_sky``
is the unique name of *Beneath a Steel Sky*, which is described at
http://www.gog.com/en/gamecard/beneath_a_steel_sky.

You can override this default by creating a "path map", which is a
text file mapping unique game names to the paths into which their
files should be placed. The file must be named ``.gog.pathmap.txt``
and be located in BACKUPINTO; within the file, list on each line a
unique name and the path into which that game's files should be
placed, separated by one or more white-space characters. Paths can be
relative to BACKUPINTO or absolute, and blank lines are not
allowed. For example:

::

    beneath_a_steel_sky      steelsky
    lure_of_the_temptress    temptress
    tyrian_2000              /opt/games/tyrian2k

The "path map" needn't be constructed in advance; you can adjust it at
any time provided you move already downloaded files into the new
location manually.


Bugs & Contributions
--------------------

If you discover a bug, please submit it on GitHub, ideally including a
patch that fixes the problem. If there's a feature you think is
missing, feel free to implement it and send me a pull request. Known
bugs and limitations include:

* Connection timeouts during manifest creation aren't handled, which
  makes large game collections problematic during peak hours.

* Files which are corrupt, truncated, *and* lack XML metadata cannot
  be distinguished from partial downloads. As a consequence, they'll
  be "resumed". Since only extras lack XML metadata, and extras are
  apparently always ``.zip`` files, the corruption *will* be detected
  after the download is "complete", but the post-"resume" data will
  need to be re-downloaded.

* Files lacking XML metadata which change in content but not in length
  will not be re-downloaded. For example, if GOG.com updates an extra
  in a way that does not affect the file's length, ``gog-backup`` will
  be unable to detect this.


Alternatives
------------

``gog-backup`` is but one of several unofficial GOG.com downloaders;
if it doesn't meet your needs, perhaps one of the others will. You can
find a feature comparison at [3]_.


References
----------

.. [1] https://github.com/evanpowers/gog-backup
.. [2] http://code.google.com/p/html5lib/
.. [3] https://github.com/evanpowers/gog-backup/wiki/Comparison


.. Typeset Documentation

    To convert this file into nroff format and view it using 'man', run:
    rst2man README.rst | man -l -

    If you prefer HTML, run:
    rst2html README.rst > gog-backup.html
