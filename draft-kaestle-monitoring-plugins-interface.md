---
title: "The Monitoring Plugins Interface"
abbrev: "MPI"
category: info

docname: draft-kaestle-monitoring-plugins-interface-latest
submissiontype: independent
number:
date:
v: 3
# area: AREA
# workgroup: The Monitoring Plugins Project
keyword:
 - monitoring
venue:
#  group: The Monitoring Plugins Project
#  type: Working Group
#  mail: devel@monitoring-plugins.org
#  arch: https://www.monitoring-plugins.org
  github: "RincewindsHat/rfc-monitoring-plugins-interface"

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

Maintaining computer networks and providing services to machines, networks and humans is
a complex task.
Building infrastructures from the start commonly demands huge know-how in different technologies,
but maintaining them afterwards is in many cases often more demanding.
Complex system can fail in a multitude of different way, where the original problem might lead
to symptoms which are often not immediately obvious and likely occur in a different place relative
to the problem.

To ensure continuous and reliable operation the state and the functionality has to monitored
permanently with appropriate tools.
These monitoring tools allow an operator, often a system and/or network administrator to
read the state of the whole system (or specific subsystems) and detect problems when they
occur.

The purpose of monitoring tools is therefore to determine the state of the systems in question
and detect possible anomalies or problems, measuring performance related metrics, process that data,
produce a human tangible representation and take action according to certain rules,
notify a system administrator for example.
A system implementing most or all of those tasks is called a "monitoring system" in the following.

With the emergence of NetSaint/Nagios at the latest, a group of network
monitoring systems
have relied on a loose group of programs called "Monitoring Plugins" to do the lower level
task of actually determining the state of a particular entity or conduct measurements of certain
values.

The same interface was implemented by several different monitoring solutions,
including, without claiming completeness, Icinga, Icinga2, Shinken, Naemon, Centreon
and Opsview.

On the other side of this interface there are hundreds of individual plugins
developed by different people in different languages for a multitude of purposes.

Examples for these are:

 - The monitoring plugins [](https://www.monitoring-plugins.org/)
 - The nagios plugins [](https://nagios-plugins.org/)
 - Several monitoring plugins by Consol [](https://labs.consol.de/de/nagios/)
 - Several monitoring plugins by Linuxfabrik [](https://github.com/Linuxfabrik/monitoring-plugins)
 - Several monitoring plugins by Davide Madrisan [](https://github.com/Linuxfabrik/monitoring-plugins)

This document shall serve administrator of those monitoring systems and
especially developers of these monitoring plugins
and monitoring systems as a basis
on how this interface should implemented, how the plugins should work and how they should behave.
It encourages the standardization of libraries, monitoring plugins and monitoring systems,
to reduce the cognitive load on administrators and developers, when they work with
different implementations.

~~~
 ┌────────────────────┐
 │ Visualisation tool ├────────────┐
 └────────────────────┘     ┌──────┴──────────┐  exec  ┌───────────────────┐
                            │ Monitoring tool │┄┄┄┄┄┄┄┄│ Monitoring Plugin │
                            └──────┬──────────┘        └───────────────────┘
 ┌────────────────────┐            │
 │ Notification tool  ├────────────┘
 └────────────────────┘
~~~

This document aims to be as general as possible and not to assume a special
implementation detail, e.g. the programming language, the install mechanism or the monitoring
system which executes the monitoring plugin.

## Wording, Context and Scope

### Wording

#### Monitoring system
A *monitoring system* is a collection of software components which serve the
purpose of providing the system administrator of a particular system
with an overview of the whole system.
This ideally includes all of the devices, machines and components and their state
as well as insights on particular components.

Most of the system mentioned here (for example Icinga, Naemon and Nagios) also
provide a functionality to send notifications to the system administrator
when something goes wrong, e.g. a particular component does not respond anymore
or a certain threshold is exceeded.


#### Monitoring plugin

A monitoring plugin is a standalone executable, which is executed by the
monitoring systems to conduct one or multiple tests on behalf of the
monitoring system.

The monitoring plugin does rely on functionality provided by the operating
system and is not a builtin of the monitoring system or linked against
certain components of the monitoring system.
Therefore it can also be executed manually and independently of a particular
monitoring system.

The monitoring plugin can therefor be implemented independently of the monitoring
system, it does not share necessarily share dependencies, the programming language
or the distribution mechanism or other components with the monitoring system.

The monitoring plugin MAY
accept parameters in the form of command line arguments, environment variables
or configuration files (the location of which MAY in turn be given on the
command line or via environment variable).

The monitoring plugin then proceeds
to execute its duty and returns the result to the Monitoring System. Part of
the process of returning the result is the termination of the execution of the
Monitoring Plugin itself.

### Scope

The scope of this document is limited to the interaction of a monitoring system
and a monitoring plugin, meaning the interface between them.

It does not attempt to describe the inner workings of a specific implementation
of either monitoring system or monitoring plugin.



# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Range expressions {#def_range_expression}

In many cases thresholds for metrics mark a certain range of values where the
values is considered to be good or bad if it is inside or outside. While for
significant number of metrics a upper (e.g. load on unixoid systems) or lower
(e.g. effective throughput, free space in memory or storage) border might
suffice, for some it does not, for example a temperature value from a
temperature sensor should be within certain range (e.g. between 10℃ and 45℃).

Regarding input parameters this might be handled with options like
`--critical-upper-temperature` and `--critical-lower-temperature`, but this
creates a problem in the performance data output, if only scalar values could be
used. To resolve this situation the _Range expression_ format was introduced,
with the following definition:

`[@][start:][end]`

where:

 1. At least `start` or `end` MUST be provided.
 1. `start` <= `end`
 1. If `start` == 0, then `start` can be omitted.
 1. If `end` is omitted, it has the "value" of positive infinity.
 1. Negative infinity can be specified with the tilde character `~`.
 1. If the prefix `@` IS given, the value exceeds the threshold if it is INSIDE the range between `start` and `end` (including the endpoints).
 1. If the prefix `@` is NOT given, the value exceeds the threshold if it is OUTSIDE of the range between `start` and `end` (including the endpoints).

### Examples

| Range definition | Exceeds threshold if x...|
| --- | --- |
| 10 | < 0 or > 10, (outside the range of {0 .. 10}) |
| 10: | < 10, (outside {10 .. ∞}) |
| ~:10 | > 10, (outside the range of {-∞ .. 10}) |
| 10:20 | < 10 or > 20, (outside the range of {10 .. 20}) |
| @10:20 | ≥ 10 and ≤ 20, (inside the range of {10 .. 20}) |


# Input Parameters for a Monitoring Plugin
A Monitoring Plugin MUST expect input parameters as arguments during
execution, if any are needed/expected at all. It MAY accept these parameters
given as _environment variables_ and it MAY accept them in a configuration file
(with a default path or a path given via arguments or _environment variables_).

In general positional arguments are strongly discouraged.

Some arguments MUST have this predetermined meaning, if they are used:

| Argument (long) | Argument (short version, optional) | Argument | Meaning | optional | can be given multiple times |
| --- | --- | --- | --- | --- | --- |
| --help | -h | None | Triggers the help functionality of the Monitoring Plugin, showing the individual parameters and their meaning, examples for usage of the Monitoring Plugin and general remarks about the how and why of the Monitoring Plugin. SHOULD overwrite all other options, meaning, they are ignored if `--help` is given. The Monitoring Plugin SHOULD exit with state UNKNOWN (3). | no | -- (makes no difference) |
| --version | -V | None | Shows the version of the Monitoring Plugin to allow users to report errors better and therefore help them and the developers. The Monitoring Plugin SHOULD exit with state UNKNOWN (3). | no | -- (makes no difference) |
| --timeout | -t | Integer (meaning seconds) or a time duration string | Sets a limit for the time which a Monitoring Plugin is given to execute. This is there to enforce the abortion of the test and improve the reaction time of the Monitoring System (e.g. in bad network conditions it might be helpful to abort the test prematurely and inform the user about that, than trying forever to do something which won't succeed. Or if soft real time constraints are present, a result might be worthless altogether after some time). A sane default is probably 30 seconds, although this depends heavily on the scenario and should be given a thought during development. If the execution is terminated by this timeout, it should exit with state UNKNOWN (3) and (if possible) give some helpful output in which stage of the execution the timeout occurred. | no | no |
| --hostname | -H | String, meaning either a DNS nameor an IP address of the targeted system | If the Monitoring Plugin targets exactly one other system on the network, this option should be used to tell it which one. If the Monitoring Plugin does its test just locally or the logic does not apply to it, this option is, of course, optional. | yes | no |
| --verbose | -v | None | Increases the verbosity of the output, thereby breaking the suggested rules about a short and concise output. The intention is to provide more information to a user. | yes | yes |
| --exit-ok | | The Monitoring Plugin exits unconditionally with OK (0). Mostly useful for the purpose of packaging and testing plugins, but might be used to always ignore errors (e.g. to just collect data). | yes | no |

## Examples
For the execution with `--help`:

~~~
$ my_check_plugin --help
~~~

the output might look like this:

~~~
my_check_plugin version 3.1.4
Licensed under the AGPLv1.
Repository: git.example.com/jdoe/my_check_plugin

This plugin just says hello. It fails if you don't give it a name.

Usage:
 my_check_plugin --name NAME [--greeting GREETING]

Options:
 --help
   this help
 --version
   Shows the version of the plugin
 --name NAME
   if given, uses NAME as a name to greet.
 --greeting GREETING
   if given, uses GREETING instead of Hello.

Examples:
$ my_check_plugin --name Jane
Hello Jane

$ my_check_plugin --greeting Ciao --name Alice
Ciao Alice
~~~

This imaginary Monitoring Plugin tries to be really helpful here, displays
the version, the license and the upstream repository with the help (although
not necessary), has a short description about the purpose, lists the options in
an easily readable way and even gives some examples.

For the execution with `--version`

~~~
$ my_check_plugin --version
~~~

the output might be a bit shorter:

~~~
my_check_plugin version 3.1.4
~~~

or even:

~~~
3.1.4
~~~

where both show the necessary information.

# Output of a Monitoring Plugin

The output of a Monitoring Plugin consists of two parts on the first level, the
*Exit Code* and output in textual form on _stdout_.

## Exit Code

The Monitoring Plugin MUST make use of the *Exit Code* as a
method to communicate a result to the Monitoring System. Since the *Exit
Code* is more or less standardized over different systems as an integer number
with a width of or greater than 8bit, the following mapping is used:

| *Exit Code* (numerical) | Meaning (short) | Meaning (extended) |
| --- | --- | --- |
| 0 | OK | The execution of the Monitoring Plugin proceeded as planned and the tests appeared to function properly and the measured values are within their respective thresholds |
| 1 | WARNING | The execution of the Monitoring Plugin proceeded as planned and the tests appeared to *not* function properly or the measured values are *not* with their respective thresholds. The problem(s) do(es) *not* seem exceptionally grave though and do(es) *not* require immediate attention |
| 2 | CRITICAL | The execution of the Monitoring Plugin proceeded as planned and the tests appeared to *not* function properly or the measured values are *not* with their respective thresholds. The problem(s) *do(es)* seem exceptionally grave though and *do(es)* require immediate attention |
| 3 | UNKNOWN | The execution of the Monitoring Plugin *did not* proceed as planned. The reasons might be manifold, e.g. missing permissions, missing libraries, no available network connection to the destination, etc.. In summary: The Monitoring Plugin could *not* determine the state of whatever it should have been checking and can therefore make no reliable statement about it. |
| 4-125 | reserved for future use |

## Textual Output

The original purpose of the output on _stdout_ was to provide human readable
information for the user of the Monitoring System, a way for the Monitoring
Plugin to communicate further details on what happened. This purpose still
exists, but was expanded with the, so called, performance data to
allow the machine readable communication of measured values for further
processing in the Monitoring System, e.g. for the creation of diagrams.

Therefore the further explanation is split into *human readable output* and
*performance data*.

### Human readable output

This part of the output should give an user information about the state of the
test and, in the case of problems, ideally hint what the origin of the problem
might be or what the symptoms are. If the test relies on numeric values, this
might be displayed to give an user more information about the specific problem.
It might consist of one or more lines of printable symbols.


Although no strict guidelines for creating this part of the output can really
be given, a developer should keep a potential user in mind. It might, for
example, be OK to put the output in a single line if there are only one or two
items of a similar type (think: multiple file systems, multiple sensors, etc.)
are present, but not if there 10 or 100, although this might present a valid
use case. If there are several different items exists in the output of the
Monitoring Plugin they probably SHOULD be given their own line in the output.

#### Examples

~~~
Remaining space on filesystem "/" is OK

Sensor temperature is within thresholds

Available Memory is too low

Sensore temperature exceeds thresholds
~~~

are OK, but

~~~
Remaining space on filesystem "/" is OK ( 62GiB / 128GiB )

Sensor temperature is within thresholds ( 42°C )

Available Memory is too low ( 126MiB / 32GiB )

Sensor temperature exceeds thresholds ( 78°C > 70°C )
~~~

are better.

### Performance data

In addition to the human readable part the output can
contain machine readable measurement values. These data points are separated
from the human readable part by the "|" symbol which is in effect until the end
of the output. The performance data then MUST consist of space (ASCII 0x20)
separated single values, these MUST have the following format:

`[']label[']=value[UOM][;warn[;crit[;min[;max]]]]`

with the following definitions:

 1. `label` MUST consist of at least on non-space character, but can otherwise
	contain any printable characters except for the equals sign (`=`) or single
	quotes (`'`). If it contains spaces, it must be surrounded by single quotes
 2. `value` is a numerical value, might be either an integer or a floating
	point number. Using floating point numbers if the value is really discreet
	SHOULD be avoided. The representation of a floating point number
	SHOULD NOT use the "scientific notation" (e.g. `6.02e23` or `-3e-45`),
	since some systems might not be able to parse them correctly. Values
	with a base other then 10 SHOULD be avoided (see below for more information
	on `Byte` values).
 3. `UOM` is the _Unit of measurement_ (e.g. "B" for _Bytes_, "s" for seconds)
     which gives more context to the Monitoring System.

    - The following constraints MUST be applied:

      1. An `UOM` of `%` MUST be used for percentage values
      2. An `UOM` of `c` MUST be used for continuous counters (commonly used
      for the sum of bytes transmitted on an interface)

    - The following recommendations SHOULD be applied:

      1. The `UOM` for `Byte` values is `B` and although many systems do
      understand units like `KB`,`KiB`, `MB`, `GB`, `TB` they SHOULD be
      avoided, at the least to avoid the ugly hassle about people
      misinterpreting the *base10* values as *base2* values and the other way
      round. This is also a prime example where floating point number SHOULD
      NOT be used, since there are obviously only integer numbers included.
      2. The `UOM` for time is `s`, meaning seconds, SI-Prefixes (e.g. `ms` for
      milli seconds) are allowed if necessary or useful.
      3. In general, SI units and SI prefixes MAY be used as `UOM` if
      applicable, but the Monitoring System may not understand them correctly
      (mostly in uncommon cases), in that cases appropriate workarounds MAY be
      applied on the side of the Monitoring Plugin. Since the
      values are not intented to be human readable normalized units are
      recommended (e.g. `overall_power=14000000000W` instead of
      `overall_power=14GW`)
      4. `warn` and `crit` are the threshold values for this measurement, which may
      have been given by the user as input, may be hardcoded in the Monitoring
      Plugin or may be retrieved from a file or a device or somewhere else
      during the execution of the Monitoring Plugin. The unit used MUST be
      the same as for _value_. These values are not simple numbers, but
      range expressions ({{def_range_expression}).
      5. `min` and `max` are the minimal respectively the maximal value the
      `value` could possibly be. The unit MUST be the same as for `value`.
      These values can be omitted, if the `value` is a percentage value,
      since `min` and `max` are always `0` and `100` in this case.

# Implementation Status

The interface metioned here is implemented by several network monitoring systems. A non-exhaustive
list of these systems includes:

 * Icinga 2
 * Naemon
 * Nagios

The other side of the interface is implemented by several different projects, again in an non-exhaustive
list:

 * The Monitoring Plugins Project
 * The Nagios Plugins Project
 * The Linuxfabrik Monitoring Plugins
 * Madrisan Nagios Plugins


# Security Considerations

Special security considerations are hard to define regarding this topic. Regarding the implementation
of this interface, the usual programming security considerations should apply (e.g. sanitize inputs),
but the risks and problems regarding security are dependent on the specific implementation and usage.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

Thanks for previous have to be said to the original inventors of this interface, although
it is not easy to determine who these persons are, so they are mentioned here in general.

Thanks are going also to the many different implementors on either side of this interface for
their hard work which allows the use of different components and systems with each other in the
best spirit of free software.
