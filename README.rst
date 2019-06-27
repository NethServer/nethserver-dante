================
nethserver-dante
================

See [official documentation](https://github.com/nethesis/dante)


Build
=====

To build the RPM, just execute: ::

  ./prep-sources && make-rpms *.spec

Database
========

Properties:

- ``Anonymization``: can be ``enabled`` or ``disabled``, if enabled data will be anonymized accordingly to widget configuratio
- ``Interval``: can be ``week``, ``month`` and ``halfyear``. The report will display respectively data from last week, last month or last 6 monts.
- ``Language``: an existing language code, default is ``en``
- ``MaxEntries``: an integer number indicating the number of entries displayed inside the widget list
- ``Palette``: UI default palette, default to ``palette1``
- ``Theme``: UI default theme, can be ``light`` or ``dark``
- ``alias``: a random generated hash to access the UI
- ``PublicHost``: the hostname (or IP address) used to publicly access the report
- ``MailDestinations``: a comma separated list of tuples, each tuple is composed by 2 fields separated with a semilcon:
    - the first field is the time when the mail will be sent, it is in cron format
    - the second field is a destination email address where the report will be delivered

Example: ::

 dante=configuration
    Anonymization=
    Interval=halfyear
    Language=en
    MailDestinations=0 3 * * 1;test@local.org,0 0 * * 2;test3@local.org
    MaxEntries=
    Palette=palette9
    PublicHost=192.168.5.246
    Theme=dark
    alias=07afb386946f357f8119d3c211378fdf29699bf4


Cockpit API
===========

dashboard
---------

The ``dashboard`` API has only the ``read`` method, it takes one argument:

- ``hostname``: the name used to access Cockpit from the web browser


Input example: ::

  {
    "hostname": "myhost.nethserver.org"
  }

Output example ::

 {
   "url": "https://myhost.nethserver.org/07afb386946f357f8119d3c211378fdf29699bf4/#?theme=darkpalette=palette9last=halfyearlang=en"
 }


Bash example: ::
  
  echo '{"hostname": "myhost.nethserver.org"}' | /usr/libexec/nethserver/api/nethserver-dante/dashboard/read  | jq



settings
--------

The ``settings`` API has 3 methods:

- ``read``
- ``validate``
- ``update``

read
^^^^

It takes an ``action`` field. Valid actions are:

- ``configuration``: return current configuration
- ``languages``: return the list of supported languages

Output example for ``configuration`` action: ::

 {
  "settings": {
    "Interval": "week",
    "Palette": "palette9",
    "Language": "en",
    "Anonymization": "enabled",
    "Theme": "dark",
    "MaxEntries": "15",
    "MailDestinations": [
      {
        "time": "0 3 * * 1",
        "mail": "test@local.org"
      },
      {
        "time": "0 9 * * 2",
        "mail": "test3@local.org"
      }
    ],
    "PublicHost": "192.168.5.246",
    "alias": "07afb386946f357f8119d3c211378fdf29699bf4"
  }
}

Output example for ``languages`` action: ::

 {
  "languages": [
    "en"
  ]
 }

validate
^^^^^^^^

Validate current configuration.

Input example: ::

 {
  "Interval": "week",
  "Palette": "palette1",
  "Language": "en",
  "Anonymization": "disabled",
  "Theme": "light",
  "MaxEntries": "10",
  "MailDestinations": [
    {
      "time": "0 3 * * 1",
      "mail": "test@nethserver.org"
    },
    {
      "time": "0 4 * * 2",
      "mail": "test3@nthserver.org"
    }
  ],
  "PublicHost": "test.nethserver.org"
 }

update
^^^^^^

It takes the same input of ``validate`` method.

execute
^^^^^^^

It requires an ``action`` field. Valid actions are: 

- ``test-mail``: send the report to a list of addresses specified inside the ``addresses`` field

Input example: ::

 {
  "action": "test-mail",
  "addresses": [
    "test@nethserver.org"
  ]
 }

