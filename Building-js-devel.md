The Problem

RHEL only ship js but not js-devel. The issue seems to be well-known and is even documented in CouchDB's own instructions:

http://wiki.apache.org/couchdb/Installing_SpiderMonkey

The Recipe

Many packages that are useful for development (e.g. js-devel, java-1.7.1-ibm) are not part of the default RHEL 7 Server repository, but they are in fact available from Red Hat through the Optional and Supplementary repositories. To add these package repositories to yum, please see these Red Hat Customer Portal articles:

- [How to access Optional and Supplementary channels, and -devel packages using Red Hat Subscription Manager (RHSM)?](https://access.redhat.com/solutions/392003)
- [How to subscribe a RHEL system to the Tools, Optional, or Supplementary channels?](https://access.redhat.com/solutions/70019)