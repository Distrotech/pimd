The original PIM-SM multicast daemon
====================================

pimd is a lightweight, stand-alone implementation of RFC 2362, available
under the 3-clause BSD license.  This is the restored original version
from University of Southern California, by Ahmed Helmy, Rusty Eddy and
Pavlin Ivanov Radoslavov.

pimd is currently maintained at GitHub.  Use its facilities for issue
reports, patches and GIT pull requests.

  http://github.com/troglobit/pimd/issues

pimd is primarily maintained on Linux and should work as-is out of the
box on all major distributions.  Other UNIX variants should also work,
but are not as thoroughly tested.  See the file `config.mk` for details.
 
Configuration
-------------

The configuration is kept in the file `/etc/pimd.conf` and the order of
the statements are in some cases important.  See the template .conf for
details.

Since pimd currently cannot retrieve distance (preference) and metric
from the kernel, or a route manager like Zebra, two settings are instead
provided that is used as default values for the

    default_source_preference <1-255>
    default_source_metric     <1-1024>

These two settings provide default values to interface specific
settings, can be o  thewill be used in PIM Assert messages:

    phyint interface

`phyint` refers to physical interface.  You fill in interface with a
reference to the ethernet card or other network interface you're telling
pimd about, either with the device's IP address or name (for example,
eth0).  If you just want to activate this interface with default values,
you don't need to put anything else on the line.  However, you do have
some additional items you can add.  The items are, in the order you
would need to use them, as follows:

   * `disable`:  Do not send PIM-SM traffic through this interface nor listen
     for PIM-SM traffic from this interface

   * `preference pref`:  This interface's value in an election. It will have
     `default_source_preference` if not assigned

   * `metric cost`:  The cost of sending data through this interface. It will
     have the `default_source_metric` if not assigned

Add one `phyint` line per interface on this router.  If you don't do
this, pimd will simply assume that you want it to utilize all interfaces
on the machine with the default values.  If you set phyint for one or
more interfaces but not for all, the missing ones will be assigned the
defaults.  After you have done this, start the next line with:

    cand_rp

cand_rp refers to Candidate Rendez-vous Point (CRP).  This statement
specifies which interface on this machine should be included in RP
elections.  Additional options to choose from are, listed in the order
used, as follows:

   * `ipadd`: The default is the largest activated IP address. If you
     don't want to utilize that interface, add the IP address of the
     interface to use as the next term

   * `time value`: The number of seconds to wait between advertising
     this CRP. The default value is 60 seconds

   * `priority num`: How important this CRP is compared to others. The
     lower the value here, the more important the CRP

The next line begins with:

    cand_bootstrap_router

Here you give the information for how this machine advertises itself as
a Candidate BootStrap Router (CBSR).  If you need to, add the ipaddr
and/or priority items as defined earlier after `cand_bootstrap_router`.
What follows is a series of statements that start with:

    group_prefix

Each group_prefix statement outlines the set of multicast addresses that
CRP, if it wins an election, will advertise to other routers. The two
items you might include here are, listed in order, as follows:

   * `groupaddr`: A specific IP or network range this router will
     handle.  Remember that a single multicast network is written as a
     single IP address

   * `masklen len`: The number of IP address segments taken up by the
     netmask.  Remember that a multicast address is a Class D and has a
     netmask of 255.255.255.255, which means its length is 4.  The
     prefix length can also be given as `groupaddr/len`

Max `group_prefix` multicast addresses supported in pimd is 255.

After this comes:

    switch_data_threshold rate rvalue interval ivalue

This statement defines the threshold at which transmission rates trigger
the changeover from the shared tree to the RP tree; starting the line
with `switch_register_threshold` does the opposite in the same format.
Regardless of which of these you choose, the `rvalue` stands for the
transmission rate in bits per second, and `ivalue` sets how often to
sample the rate in seconds -- with a recommended minimum of five
seconds.  It's recommended by the pimd developers to have `ivalue` the
same in both statements.

For example, I might end up with the following (these are real IP
addresses; don't use them for actual testing purposes):

    default_source_preference 105
    
    phyint 199.60.103.90 disable
    phyint 199.60.103.91 preference 1029
    phyint 199.60.103.92 preference 1024
    cand_rp 199.60.103.91
    cand_bootstrap_router 199.60.103.92
    group_prefix 224.0.0.0 masklen 4
    switch_data_threshold rate 60000 interval 10
    switch_register_threshold rate 60000 interval 10


Running pimd
------------

After you've set up the configuration file, you're ready to actually run
the PIM-SM daemon.  As usual, we recommend that you run this by hand for
testing purposes and then later add the daemon, with any startup flags
it needs, to your system's startup scripts.

The format for running this daemon is:

    pimd [-c file] [-d [level1,...,levelN]]

Both of the flags with their values are optional:

   * `-c file`: Utilize the specified configuration file rather than the
      default, `/etc/pimd.conf`

   * `-d [level1,...,levelN]`: Specifies the debug level(s) to utilize
      when running the daemon.  Type `pimd -h` for a full list of levels


Monitoring
----------

    pimd -r

or

    watch pimd -r


