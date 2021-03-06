..
 Copyright (c) 2015 Palo Alto Networks, Inc. <techbizdev@paloaltonetworks.com>

 Permission to use, copy, modify, and distribute this software for any
 purpose with or without fee is hereby granted, provided that the above
 copyright notice and this permission notice appear in all copies.

 THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
 WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
 MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
 ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
 WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
 ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
 OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

================
firebreak-box.py
================

-------------------------------------------------------
command line program for cloud storage malware analysis
-------------------------------------------------------

NAME
====

 firebreak-box.py - command line program for cloud storage malware analysis

SYNOPSIS
========
::

 firebreak-box.py [options]
    --access_token token  Box OAuth2 access token
    --match pattern       process files matching pattern
    --tag tagname         .panrc tagname (WildFire api_key)
    --fbtag tagname       .firebreak-box.json tagname (Box OAuth2)
    --submit              submit files to WildFire for analysis
    --debug level         debug level (0-3)
    --help                display usage

DESCRIPTION
===========

 **firebreak-box.py** is a command line program which provides
 an integration between the Box cloud storage system and Palo
 Alto Networks' cloud-based WildFire malware analysis environment.

 It uses the Box Content API to access files in a user's cloud storage
 folder tree, and the WildFire API to optionally submit files as
 samples for analysis and malware verdict identification.

 The options are:

 ``--access_token`` *token*
  Specify the OAuth 2.0 *access_token* for a Box application.

  This can be a developer token for a Box application.

 ``--match`` *pattern*
  Apply the Unix shell style *pattern* to the files in a user's folder
  tree.  This uses the Python `fnmatch
  <https://docs.python.org/3.4/library/fnmatch.html>`_ module.

 ``--tag`` *tagname*
  Specify tagname for the ``.panrc`` file *api_key* varname.  This is used
  to specify the WildFire API key to use for sample submittal and
  verdict identification.

 ``--fbtag`` *tagname*
  Specify tagname for the ``.firebreak-box.json`` file.

 ``--submit``
  Submit matched files to WildFire for analysis (if previously unseen)
  and malware verdict identification (if previously seen).

 ``--debug`` *level*
  Enable debugging.
  *level* is an integer in the range 0-3; 0 specifies no
  debugging and 3 specifies maximum debugging.

 ``--help``
  Display **firebreak-box.py** command options.

Create a Box Application
------------------------

 Before you can use the Box API to access your cloud storage tree you
 need to create an application for content API access at
 https://app.box.com/developers/services.  This allows you to create
 a developer token to use in the **--access_token** option.

 Alternatively you can use the **boxoauth2.py** program to generate a
 *refresh_token* using your application's *client_id* and
 *client_secret* and save them to a ``.firebreak-box.json`` file.

WildFire API
------------

 The
 `pan.wfapi <https://github.com/kevinsteves/pan-python/blob/master/doc/pan.wfapi.rst>`_
 module from `pan-python <https://github.com/kevinsteves/pan-python>`_
 is used to perform WildFire API requests.  When the **--submit**
 option is specified matched files are submitted to the WildFire cloud
 using the ``/publicapi/submit/url`` API request and the verdict is
 obtained using the ``/publicapi/get/verdict`` API request.

 The Box content API provides the SHA-1 hash of each file, however we
 cannot query WildFire for a sample report using SHA-1, and therefore
 we do not know if the sample is known by WildFire.  The submit URL
 response provides the MD5 and SHA-256 hash of the sample, and that is
 used to request the sample's verdict.  If it is a known sample we
 will see its verdict as *malware* or *benign*; if the sample has
 previously not been seen its verdict will be *unknown*.  The WildFire
 portal can be used to determine the verdict after analysis has been
 completed.  Note: the Box download URL does not contain the origin
 filename, so you have to perform file correlation and remediation
 using a hash from the WildFire analysis report; it is recognized that
 this is suboptimal.

Secrets File
------------

 ``.firebreak-box.json`` contains OAuth 2.0 parameters and is generated
 using **boxoauth2.py**.  Each time **firebreak-box.py** is executed
 without specifying **--access_token**, an *access_token* is generated
 using the *client_id*, *client_secret* and *refresh_token* in the
 file, and the *refresh_token* is updated.

 The *refresh_token* is valid for one use in 60 days.  Each time the
 *refresh_token* is used to generate an *access_token* a new
 *refresh_token* is returned and the 60 day timer is reset.

 The secrets file can be located in the current working directory
 or the $HOME directory.

FILES
=====

 ``.panrc``
  .panrc file.

 ``.firebreak-box.json``
  secrets file.

EXIT STATUS
===========

 **firebreak-box.py** exits with 0 on success and 1 if an error occurs.

EXAMPLES
========

 List files using the ``.firebreak-box.json`` *panw* tagname.
 ::

  $ firebreak-box.py -fbtag panw
  "All Files/tmp/ksteves-edu.zip" size 157380
  "All Files/tmp/user-id.zip" size 2234
  "All Files/tmp2/folder1/wildfire-test-pe-file-1.exe" size 55296
  "All Files/tmp2/folder1/wildfire-test-pe-file-2.exe" size 55296

 Set *api_key* varname in ``.panrc`` with *wildfire* tagname:
 ::

  $ echo 'api_key%wildfire=633531ce8899bd8368e3f549bbca307e' >> ~/.panrc

 Submit '\*.exe' files to WildFire for analysis.
 ::

  $ firebreak-box.py --fbtag panw --match '*.exe' --submit --tag wildfire
  "All Files/tmp2/folder1/wildfire-test-pe-file-1.exe" size 55296
  "wildfire-test-pe-file-1.exe" (PE32 executable) uploaded to WildFire
      sha256 a1fd5883534a47c2145697da0a56fd708d4e685bc3f2a2e95f1d462d585e954d
      verdict malware
  "All Files/tmp2/folder1/wildfire-test-pe-file-2.exe" size 55296
  "wildfire-test-pe-file-2.exe" (PE32 executable) uploaded to WildFire
      sha256 81814f48506fdab108b49970c457f5b52dc9630d39eee37d53e9800cddd76f0b
      verdict pending

SEE ALSO
========

 boxoauth2.py
  https://github.com/PaloAltoNetworks-BD/firebreak-box/blob/master/doc/boxoauth2.rst

 pan.wfapi
  https://github.com/kevinsteves/pan-python/blob/master/doc/pan.wfapi.rst

 WildFire Administrator's Guide
  https://www.paloaltonetworks.com/documentation/61/wildfire/wf_admin.pdf.html

 WildFire API
  https://www.paloaltonetworks.com/documentation/61/wildfire/wf_admin/wildfire-api.html

AUTHORS
=======

 Palo Alto Networks, Inc. <techbizdev@paloaltonetworks.com>
