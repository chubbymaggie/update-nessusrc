<HTML>
<HEAD>
<TITLE>update-nessusrc</TITLE>
<LINK REV="made" HREF="mailto:theall@tifaware.com">
<LINK REL="shortcut icon" HREF="http://www.tifaware.com/favicon.ico">
</HEAD>

<!--#include virtual="/header.html"-->
<!--#if expr="$weblint" -->
<BODY BGCOLOR="#FFFFFF">
<!--#endif -->


<H4 ALIGN="center">
   <A HREF="/">TifaWARE</A> |
   <A HREF="../">Perl Programs</A> |
   update-nessusrc
</H4>
                                                                                
                                                                                
<HR>
<H2>update-nessusrc</H2>

<H3>Introduction</H3>

<P><A HREF="http://www.nessus.org/">Nessus</A> is a high-quality and
free remote vulnerability scanner.  While its initial cost is certainly
attractive, arguably its main advantage is its extensive and continually
evolving plugin database of vulnerability checks.  Additionally, one can
configure Nessus to constantly scan a network with a continuously
updated collection of plugins with minimal user interaction -- an
extremely appealing notion!</P>

<P>In practice, though, this doesn't quite work, at least not directly. 
The reason is that plugins not explicitly listed in the client
configuration file -- eg, new plugins -- are enabled by default
(unless you've enabled safe_checks, in which case dangerous plugins are
disabled).  Further, the only way to update the configuration file
out-of-the-box is via the GUI manually.</P>

<P>To get around this shortcoming, I've written <B>update-nessusrc</B>,
which queries a Nessus server for its list of available plugins and
updates a Nessus client configuration file named on the commandline. 
Specifically, it completely updates the sections
<CODE>SCANNER_SET</CODE> and <CODE>PLUGIN_SET</CODE> whenever it is run. 
When used periodically along with <CODE>nessus-update-plugins</CODE>, it
ensures your client configuration files are as current as possible.</P>

<P><B>update-nessusrc</B> is written in Perl and calls the Nessus client
to obtain a list of current plugins (using the option <CODE>-qp</CODE>). 
It should work on any unix-like system with Perl 5.003 or better and
Nessus 1.1.13 or better.  It also requires the following Perl
modules:</P>

<UL>
   <LI><CODE>Carp</CODE></LI>
   <LI><CODE>Getopt::Long</CODE></LI>
   <LI><CODE>IPC::Open2</CODE></LI>
   <LI><CODE>LWP::Debug</CODE></LI>
   <LI><CODE>LWP::UserAgent</CODE></LI>
   <LI><CODE>Safe</CODE></LI>
</UL>

<P>If your system does not have these modules installed already, visit
<A HREF="http://search.cpan.org/">CPAN</A>.  Note that <CODE>Safe</CODE>
must be at least version 2.0, which does not work with versions of Perl
older than 5.003.  Note also that <CODE>LWP::Debug</CODE> and
<CODE>LWP::UserAgent</CODE> are not included with the default Perl
distribution so you may need to install them yourself; they're included
as part of the <A HREF="http://search.cpan.org/dist/libwww-perl/">LWP
library</A>.</P>

<P>Note: Jay Jacobson of <A HREF="http://www.edgeos.com/">Edgeos</A>
wrote a Python script, also named update-nessusrc, that offers
functionality similar to this script. I'm not sure it's still
available, though.</P>


<H3>Installation</H3>

<OL>
   <LI>Retrieve the script <A HREF="/code/update-nessusrc/update-nessusrc">update-nessusrc</A>
      and save it locally.  You may also wish to verify its
      <A HREF="/code/update-nessusrc/update-nessusrc.sha256sum">SHA256 checksum</A> or, better, its
      <A HREF="/code/update-nessusrc/update-nessusrc.asc">GPG signature</A> against
      <A HREF="/~theall/gpg.html">my current GPG key</A>.</LI>
   <LI>Verify ownership and permissions on the script - it can (and
      probably should) be invoked as an ordinary user rather than root 
      and will hold a userid and password used to connect with a Nessus 
      server.</LI>
   <LI>Edit the script and set <CODE>$nessusd_host</CODE>, 
      <CODE>$nessusd_port</CODE>, <CODE>$nessusd_user</CODE>,
      <CODE>$nessusd_user_pass</CODE>, and <CODE>$proxy</CODE> according 
      to your environment. Also, you may wish to adjust the location of 
      the perl interpreter in the first line, <CODE>$ENV{PATH}</CODE>,
      <CODE>@plugin_cats</CODE>, <CODE>@plugin_fams</CODE>, 
      <CODE>@plugin_excludes</CODE>, <CODE>@plugin_includes</CODE>, 
      <CODE>@plugin_risks</CODE>, and/or <CODE>$script_config</CODE>
      to suit your tastes.</LI>
   <LI>Have each user create a script configuration file with personalized
      settings, if desired.</LI>
</OL>


<H3>Use</H3>

<P><B>update-nessusrc</B> offers considerable control in the selection
of plugins - you can include entire categories and families of plugins,
include specific plugins, exclude specific plugins, or even include
based on the risk factor of the vulnerabilities scanned for by
plugins.</P>

<P>Much of the script's behaviour is controlled by variables set either
in the script itself or in a separate script configuration file,
specified by <CODE>$script_config</CODE>.  If it exists, the script
configuration file is treated as Perl code and evaluated in a
<I>sandbox</I>, which supports only variable definitions.  Use of a
separate script configuration file makes it possible for several people
to share <B>update-nessusrc</B> on the same system and promises to make
upgrading the script much easier.</P>

<P>There are several commandline arguments you can use to override
variables defined in the script or the script configuration file:</P>

<BLOCKQUOTE>
<DL>
   <DT>-c, --categories &lt;plugin categories&gt;</DT>
   <DD>Enable plugins in the specified categories, overriding
      <CODE>@plugin_cats</CODE>. <CODE>_all_</CODE> can be used to
      represent all plugin categories and the prefix <CODE>!</CODE>
      to skip specific ones.</DD>

   <DT>-d, --debug</DT>
   <DD>Display debugging messages while running.  Creates a scratch
      configuration file but doesn't actually replace the original
      one.</DD>

   <DT>-f, --families &lt;plugin families&gt;</DT>
   <DD>Enable plugins in the specified families, overriding
      <CODE>@plugin_fams</CODE>. <CODE>_all_</CODE> can be used to
      represent all plugin families and <CODE>!</CODE> to skip
      specific ones.</DD>

   <DT>-i, --includes &lt;plugin ids&gt;</DT>
   <DD>Include the specified plugin ids, overriding 
      <CODE>@plugin_includes</CODE>. <CODE>_all_</CODE> can be used
      to represent all plugin ids, <CODE>!</CODE> to skip specific ones,
      and <CODE>x-y</CODE> to cover the range of plugins between ids 
      <I>x</I> and <I>y</I> inclusive.</DD>

   <DT>-r, --risks &lt;risk factors&gt;</DT>
   <DD>Enable plugins that scan for vulnerabilities with the specified risk 
      factors, overriding <CODE>@plugin_risks</CODE>. <B>Note:</B> unlike
      other options, risk factors specified are regarded as regular 
      expressions and matched case-insensitively against the risk factors
      appearing in plugin descriptions.</DD>

   <DT>-s, --summary</DT>
   <DD>Display a summary of the changes, detailing plugins added, removed,
      enabled, or disabled.</DD>

   <DT>-t, --top20</DT>
   <DD>Include plugins to scan for the 
      <A HREF="http://www.sans.org/top20/">SANS Top 20 
      Vulnerabilities</A>.</DD>

   <DT>-x, --excludes &lt;plugin ids&gt;</DT>
   <DD>Exclude the specified plugin ids, overriding 
      <CODE>@plugin_excludes</CODE>.  <CODE>_all_</CODE> can be used
      to represent all plugin ids, <CODE>!</CODE> to skip specific ones, 
      and <CODE>x-y</CODE> to cover the range of plugins between ids 
      <I>x</I> and <I>y</I> inclusive.</DD>
</DL>
</BLOCKQUOTE>

<P>Notes:</P>
<UL>
   <LI>For a list of plugin families, see <A
      HREF="http://www.nessus.org/plugins/index.php?view=all">http://www.nessus.org/plugins/index.php?view=all</A>;
      and for plugin categories, see the file <CODE>doc/WARNING.En</CODE> in 
      the nessus-core source for Nessus 2.x. Unfortunately, risk factors are 
      not standardized; while <CODE>Low</CODE>, <CODE>Medium</CODE>, and
      <CODE>High</CODE> are common ones, they are by far from the only
      ones used. Further, some risk factors depend on the outcome of
      the plugin. Possible category and family names will both change over 
      time, especially the latter: category names may vary with the 
      version of Nessus you use while family names are specified 
      by plugin writers as an arbitrary string using the 
      <CODE>script_family</CODE> function. Thus, you may wish to 
      periodically review the specific categories and families you use with
      this script.

   <LI>Commandline arguments take precedence over variables defined in a
      script configuration file, which in turn take precedence over variables
      defined in the script itself. For example, you can disable all plugin 
      categories by using the commandline argument <CODE>-c ""</CODE>.</LI>

   <LI><B>Plugin categories, families, and risk factors are considered 
      together when deciding whether to include plugins.</B>  For example, 
      choosing <CODE>-c denial -f "SMTP problems" -r "High"</CODE> will 
      run only denial of service attacks for high-risk vulnerabilities 
      specifically associated with SMTP servers.</LI>

   <LI>To negate a range of plugin ids, prefix the first id only with '!';
      eg, <CODE>!10000-100010</CODE>.</LI>

   <LI>Nessus will run plugins in the category <CODE>settings</CODE> as
      needed regardless of what's in the configuration file.  These plugins
      only update the knowledge bases; they do not actually send any 
      packets.</LI>

   <LI>The <CODE>'--top20'</CODE> option on one hand and the 
      <CODE>'--categories'</CODE>, <CODE>'--families'</CODE>, and
      <CODE>'--risks'</CODE> options on the other are mutually exclusive.</LI>

   <LI>Plugins explicitly excluded will never be used regardless of the other
      variables or commandline options.</LI>

   <LI>Multiple categories / families / risks / plugin ids can be specified 
      either by a comma-delimited string or by multiple argument pairs.  
      For example, <CODE>-c "settings,infos"</CODE> is equivalent to 
      <CODE>-c settings -c infos</CODE>.</LI>
</UL>

<P>Examples:</P>
<BLOCKQUOTE>
<DL>
   <DT><CODE>update-nessusrc ~/.nessusrc</CODE></DT>
   <DD>updates nessusrc using default settings (eg, non-dangerous plugins
      and ping / tcp_connect scanners in the script as distributed).</DD>

   <DT><CODE>update-nessusrc -s ~/.nessusrc</CODE></DT>
   <DD>same as above but also print a summary of the changes.</DD>

   <DT><CODE>update-nessusrc -d ~/.nessusrc</CODE></DT>
   <DD>produces an alternate nessusrc without replacing original and
      also prints lots of debugging info.</DD>

   <DT><CODE>update-nessusrc -c "" -f "" -r "" -i "10335,11835" .nessusrc-ms03-039</CODE></DT>
   <DD>updates a special nessusrc to use tcp_connect scanner (plugin #10335) 
      and test just for MS RPC interface buffer overruns (plugin #11835).</DD>

   <DT><CODE>update-nessusrc -c "denial,destructive_attack,flood,kill_host" -i 10335 ~/.nessusrc-destructive</CODE></DT>
   <DD>updates a special nessusrc w/ destructive plugins and tcp connect scanner.</DD>

   <DT><CODE>update-nessusrc -c "_all_" -f "SMTP problems" ~/.nessusrc-smtp</CODE></DT>
   <DD>updates a special nessusrc w/ plugins associated with SMTP servers in
      all categories.</DD>

   <DT><CODE>update-nessusrc -c "_all_" -c !destructive_attack -f "SMTP problems" ~/.nessusrc-smtp</CODE></DT>
   <DD>updates a special nessusrc w/ plugins associated with SMTP servers in
      all categories except destructive_attack.</DD>

   <DT><CODE>update-nessusrc -r "(Critical|High)" ~/.nessusrc-risky</CODE></DT>
   <DD>updates a special nessusrc w/ plugins that scan only for critical-
      and high-risk vulnerabilities.</DD>

   <DT><CODE>update-nessusrc -t ~/.nessusrc-top20</CODE></DT>
   <DD>Update a special nessusrc to scan for the SANS Top 20 
      Vulnerabilities.</DD>
</DL>
</BLOCKQUOTE>


<H3>Known Bugs and Caveats</H3>

<P>This script may hang indefinitely if <CODE>paranoia_level</CODE> is
not set in your config file (for example, if you've exported a policy
from NessusClient 3.2+) or it is set but the SSL certificate presented
by the server has changed for some reason.  If this happens, either
update the script so it calls the client with <CODE>-x</CODE>, which
disables certificate verification, or run <CODE>nessus</CODE> manually
and resolve SSL_paranoia / certificate issue outside of this script, at
least until I can figure out a satisfactory way to handle such
cases.</P>

<P>As of August 2005, SANS appears to be adjusting their content based
on the <I>browser's</I> user agent and, in the case of my script, it
redirects to a non-existent page.  A work-around is to change the
user-agent string, <CODE>$useragent</CODE>, to something more common;
eg,

<BLOCKQUOTE>
<PRE>
  Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.10) Gecko/20050716 Firefox/1.0.6
</PRE>
</BLOCKQUOTE>

<P>You can make the change either in the script itself or in the file
<CODE>~/.update-nessusrc</CODE>.</P>

<P>This script is not a substitute for the Nessus client in terms of
managing a configuration file.  On one hand, it requires that a
configuration file already exists.  On the other, several plugins
require additional configuration - simply adding them to the list of
plugins used may not be optimal.</P>

<P>There is a limit to the size of the arguments passed to
<CODE>script_cve_id()</CODE>, which sets the CVE IDs of the flaws tested
by each plugin.  Additional CVE IDs, which by convention are listed in
comments, are not handled by this script since they can not be reliably
identified.  Thus, you would do well to review the report of Top 20
Vulnerabilities for which no plugins were found and update the
configuration file manually after examining plugins available on your
server.  Otherwise, you risk generating a configuration file that's not
as comprehensive as it could be.</P>

<P>To ensure an accurate scan for the SANS Top 20 Vulnerabilities, you
must make sure <CODE>auto_enable_dependencies</CODE> is set to
<CODE>yes</CODE> in the configuration file; <B>update-nessusrc</B> will
<B><I>not</I></B> do this for you.</P>

<P>Finally, realize that this script along with its script configuration
files may hold userids and passwords used to connect to a Nessus server;
protect them accordingly!</P>


<H3>Copyright and License</H3>

<P>Copyright (c) 2003-2010, George A. Theall.<BR>
All rights reserved.</P>

<P>This script is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.</P>


<H3>History</H3>

<CENTER>
<TABLE CELLPADDING="5" CELLSPACING="0" WIDTH="95%" BORDER="1">
   <TR>
      <TH ALIGN="left">Date </TH>
      <TH ALIGN="left">Version </TH>
      <TH ALIGN="left">Verification </TH>
      <TH ALIGN="left">Notes<BR></TH>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">29-Jan-2010 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.39">2.39</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.39.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.39.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Added support for Nessus 4.2 (getting the version was failing).</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">26-May-2009 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.38">2.38</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.38.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.38.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Add support for Nessus 4.x.</LI>
         <LI>Tweaked handling of whitespace in the risk factor.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">22-Mar-2006 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.37">2.37</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.37.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.37.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Added <CODE>/opt/nessus/bin</CODE> to the default PATH.</LI>
         <LI>Fixed typo in help text.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">20-Nov-2005 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.36">2.36</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.36.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.36.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Removed loop if nessus prompts to accept the server certificate.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">27-Sep-2005 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.35">2.35</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.35.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.35.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Adjusted regular expression pattern to handle the newer
            description format used for the risk factor.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">24-Aug-2005 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.34">2.34</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.34.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.34.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Changed user-agent string to something more common to avoid
            problems when requesting the SANS Top 20.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">11-Oct-2004 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.33">2.33</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.33.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.33.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Changed patterns used to identify the start and end of plugin
            sets to allow for leading and trailing whitespace on the line,
            which should avoid generating config files with multiple plugin
            sets.</LI>
         <LI>Added distribution URL to <CODE>$useragent</CODE>.</LI>
         <LI>Added warning messages if a scanner or plugin set is not found
            in the configuration file.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">20-Aug-2004 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.32">2.32</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.32.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.32.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Changed <CODE>$useragent</CODE> to reflect current version 
            number. :-(</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">20-Aug-2004 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.31">2.31</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.31.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.31.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Removed mention of not handling server certificates from the 
            POD.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">21-Jul-2004 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.30">2.30</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.30.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.30.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Changed code to direct usage message and certificate validation
            question to stderr rather than stdout.</LI>
         <LI>Added support for plugin id ranges.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">19-Mar-2004 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.21">2.21</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.21.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.21.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Changed the bang-path to use <CODE>/usr/bin/perl</CODE> rather 
            than <CODE>/usr/local/bin/perl</CODE>.</LI>
         <LI>Changed the usage message.
         <LI>Removed any trailing spaces from Risk Factor, which otherwise 
            might present problems when trying to enable plugins based on 
            risk.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">26-Jan-2004 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.20">2.20</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.20.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.20.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Added support for <CODE>-h</CODE> option.</LI>
         <LI>Added support for script configuration files. [Suggested by 
            David Klann.]</LI>
         <LI>Added code to explicitly use binary semantics when reading
            plugin output from nessus.</LI>
         <LI>Added support for validating server certificates if
            necessary.</LI>
         <LI>Modified the default set of categories to explicitly exclude 
            those in the flood category, which is expected to be introduced
            in Nessus 2.1.0.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">27-Oct-2003 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.12">2.12</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.12.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.12.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Fixed a bug in the parsing of output from <CODE>nessus 
            -qp</CODE> that would cause risk level to be incorrect if 
            using Nessus 2.0.8 or later. This required adding code to 
            determine which version of the client is being run.</LI>
         <LI>Added support for BugTraq ID and X-Reference fields when 
            summarizing changes if the nessus client supports those 
            fields.</LI>
         <LI>Modified the pattern used to identify plugin risk factors to 
            catch those introduced by not only <CODE>Risk factor</CODE> 
            but also just <CODE>Risk</CODE>.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">08-Oct-2003 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.11">2.11</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.11.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.11.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Changed the pattern used to identify CAN/CVE numbers in the SANS
            Top 20 List so it works with version 4.0 of that list, released
            today.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">15-Aug-2003 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.10">2.10</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.10.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.10.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Added support for bundling commandline options; eg, 
            <CODE>-ds</CODE> is the same as <CODE>-d -s</CODE>.</LI>
         <LI>Added ability to negate categories, families, and/or plugin 
            ids so you can say something like <CODE>-f _all_ -f 
            !CISCO</CODE> to use plugins from all families except CISCO. 
            Also, extended use of <CODE>_all_</CODE> to plugins included 
            or excluded.</LI>
         <LI>Added ability to select by risk factor of the vulnerability
            scanned for by the plugin. [Suggested by Phil Agcaoili.]</LI>
         <LI>Added warnings about any plugin categories, families, ids, or
            risks selected but not available via the Nessus server.</LI>
         <LI>Changed defaults for <CODE>@plugins_cats</CODE> and 
            <CODE>@plugins_fams</CODE> so that all non-dangerous plugins 
            are enabled by default in a manner that does not rely on 
            explicitly listing category and/or family names. Also, 
            changed default scanner from nmap to tcp connect() in 
            <CODE>@plugin_includes</CODE>.</LI>
         <LI>Changed code to provide help rather than complain about a 
            non-existent configuration file if none is specified on the
            commandline.</LI>
         <LI>Changed code to strip extraneous information from the version 
            string and also include the risk factor of the vulnerability 
            scanned for by a plugin when summarizing changes.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">30-May-2003 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.01">2.01</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.01.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.01.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Changed the documentation to reflect that the 
            <CODE>'--top20'</CODE> option on one hand and the 
            <CODE>'--categories'</CODE> and <CODE>'--families'</CODE>
            options on the other are mutually exclusive.</LI>
         <LI>Added code to check for existence of the configuration file
            and croak rather than simply providing help if not.</LI>
         <LI>Added code to debug LWP's retrieval of the SANS Top 20 List.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">21-May-2003 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.00">2.00</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-2.00.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-2.00.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Changed major portions of code.</LI>
         <LI>Am now sorting plugins by id when writing new configuration 
            file.</LI>
         <LI>Added support for configuration files lacking PLUGIN_SET
            and/or SCANNER_SET sections.</LI>
         <LI>Added support for enabling / disabling plugins based on 
            family.</LI>
         <LI>Added support for pseudo category / family <CODE>_all_</CODE>.</LI>
         <LI>Added support for summarizing changes made.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">17-Feb-2003 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-1.10">1.10</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-1.10.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-1.10.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Am now allowing nessusd port to be configured.</LI>
         <LI>Removed support for <CODE>-s</CODE> option since I discovered 
            Nessus actually treats plugins not listed in a configuration 
            file as enabled.</LI>
         <LI>Added support for plugin categories introduced in 
            Nessus 1.3.0.</LI>
         <LI>Am now reporting an error if the nessus client produces 
            an error, as opposed to can't be run.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">19-Jan-2003 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-1.01">1.01</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-1.01.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-1.01.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Am now passing along the specified rcfile when invoking 
            the client so settings such as <CODE>cert_file</CODE> and 
            <CODE>key_file</CODE> in that file will be used.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">13-Jan-2003 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-1.00">1.00</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/update-nessusrc/update-nessusrc-1.00.sha256sum">SHA256 checksum</A> / 
         <A HREF="/code/update-nessusrc/update-nessusrc-1.00.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Initial version.</LI>
      </UL></TD>
   </TR>
</TABLE>
</CENTER>


<HR>
<H4 ALIGN="center">
   <A HREF="/">TifaWARE</A> |
   <A HREF="../">Perl Programs</A> |
   update-nessusrc
</H4>
                                                                                
<!-- $Id$ -->

<!--#include virtual="/footer.html" -->
<!--#if expr="$weblint" -->
</BODY>
<!--#endif -->
</HTML>