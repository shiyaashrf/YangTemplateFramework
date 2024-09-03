---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Principle for populating list of YANG data nodes using a template technique."
abbrev: "TODO - Abbreviation"
category: info

docname: draft-yang-config-template-framework
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ops
workgroup: netmod
keyword:
 - templates
 - profiles
 - yang scalability
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Robert Peschi
    organization: Nokia
    email: robert.peschi@nokia.com

    fullname: Shiya Ashraf
    organization: Nokia
    email: shiya.ashraf@nokia.com

    fullname: Deepak Rajaram
    organization: Nokia
    email: deepak.rajaram@nokia.com
normative:

informative:


--- abstract

Some devices can host a specific functional entity (e.g. a sub-system) which is  replicated a large number of times, where each such functional instance is to be controlled by a set of data nodes. This can lead to long device configuration times, large memory footprint in the device and inefficient YANG validation procedures due to the large running data store size. 

This RFC  considers the case of a device where each such functional instance is to be controlled by a set of data nodes with almost identical values from instance to instance. In this case this RFC proposes a generic method that improves  the inefficiency issues mentioned above. 

Rather than defining a YANG model where all data nodes of all instances need to be configured one by one in the device running data store by the management system (e.g. a NC client), this RFC presents the idea of a generic YANG model  in which the running data store is configured with a list of instances without explicit data node being configured and the instruction to locally populate each instance's data nodes with a copy of a "model" data node set (called "template") that is configured in the running data store. 

Although the data nodes controlling instances have largely similar values across instances, a small number of data nodes may need however to have a value that differs from the ones in the common template, from instance to instance. The generic method presented in this RFC also gives the possibility to overule or complement values originating from the template, on a per instance basis.

This RFC presents a generic model - a generic method rather - that relies on features available in YANG 1.1, supported by off-the-shelf toolset. It has no ambition to become a standard  track itself, instead it should be considered as   a method that can be used when designing a module for a specific purpose, possibly a specific standard.

--- middle

# Introduction

1) Introduction 

The issue with large systems
   (...)

Improving model efficiency of large system
   Goals and non goals: applicable for some type of devices only

Principle (cf execf summary)

2) Mechanism
     A replicable set of data node: [A, B, C, ..Z]
     Configuring a list of templates in running DS
     Configuring a list of instances in runnin DS
     Device instantiating the data needed to control each instance 
     (NC merge template and instance data nodes
     Viewing instantiated data nodes
     Special consideration about madatories and defaults
     same data nodes in templates and instance: 
        when groupings are handy
        how to handle leaf-ref in grouping, though ?




# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
