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
title: "The Monitoring Plugins Interface"
abbrev: "MPI"
category: info

docname: draft-lorenzKaestle-MonitoringPluginsInterface-latest
submissiontype: independent
number:
date:
v: 3
area: AREA
workgroup: The Monitoring Plugins Project
keyword:
 - monitoring
venue:
  group: The Monitoring Plugins Project
  type: Working Group
  mail: devel@monitoring-plugins.org
  arch: https://www.monitoring-plugins.org
  github: monitoring-plugins/monitoring-plugin-guidelines

author:
 -
    fullname: Lorenz Kästle
    organization: The Monitoring Plugins Project
    email: lorenz@vulgrim.de

normative:

informative:


--- abstract

This document aims to document the Monitoring Plugin Interface, a standard more or less
strictly implemented by different network monitoring solutions.
Implementers and Users of network monitoring solutions, monitoring plugins and libraries can use
this as a reference point as to how these programs interface with each other.

--- middle

# Introduction

With the emergence of NetSaint/Nagios at the latest, this system and their successors/clones
have relied on a loose group of programs called "Monitoring Plugins" to do the lower level
task of actually determining the state of particular entity or conduct measurements of certain
values.

This document shall help users and especially developers of those programs as a basis
on how they should be implemented, how they should work and how they should behave.
It encourages the standardization of libraries, Monitoring Plugins and Monitoring Systems,
to reduce the cognitive load on users, administrators and developers, if they work with
different implementations.

These guidelines aim to be mostly as general as possible and not to assume anticipate a special
implementation detail, e.g. the programming language, the install mechanism or the monitoring
system which executes the Monitoring Plugin.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

Special security considerations are hard to define regarding this topic. Regarding the implementation
of this interface, the usual programming security considerations should apply (e.g. sanitize inputs),
but the risks and problems regarding security are dependent on the specific implementation and usage.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
