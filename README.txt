

The aim of this script is to rapidly deploy multiple copies
of Mailman on a single host for virtual hosting.

Mailman's design has very limited support for virtual hosting.
Various techniques exist using a shared installation of Mailman.
They have some particular limitations:
 * all virtual lists appear on a single web page
 * each list name must be globally unique across all domains
 * a single shared domain for the "site list"

The only effective way to purely achieve virtual hosting appears to
be installing multiple copies of Mailman, built from source, each
having its own directory tree.  Each tree serves a single domain.

This does not require multiple mail server instances.  A single
mail server instance can be used, however, it is necessary to use
a hack to map virtual aliases to unique names in the aliases file.
The enclosed script "gen-mapped-aliases" automates this.

== Setup procedure ==

1. Install the basic Mailman 2.1.15 Debian package (to provide images and
   other shared artifacts under /usr/share)

2. Set up a directory for the aliases files:

      mkdir /etc/mailmen

   and add them to /etc/postfix/main.cf (do not use line breaks):

      alias_maps = hash:/etc/aliases,
                            hash:/etc/mailmen/mapped-aliases
      virtual_alias_maps = hash:/etc/postfix/virtual,
                            hash:/etc/mailmen/mapped-virtual

3. Get the sources

      apt-get source mailman

   or just download from the Mailman web site. If you are using
   apt-get you may need to execute 'apt-get install dpkg-dev' before
   the source download will work.

4. Build a custom instance for each domain, e.g.:

      fakeroot ./make-mailman mailman_2.1.15.orig.tar.gz lists.example.org

   If you are not on Debian, you may need to tweak "make-mailman", particularly
   the environment variables at the beginning. If you get an error
   message about Python Distutils, execute 'apt-get install python-dev
   python-setuptools'

   You will find tarballs under /tmp for each of your domains, e.g.
   /tmp/mailman-lists.example.org.tar.gz

5. As root, unpack the compiled tarball

      su -
      cd /
      tar xzf /tmp/mailman-lists.example.org.tar.gz

6. Enable the service for each domain:

      update-rc.d mailmen-lists-example-org defaults 20 2 3 4 5 .

7. Create the site list for each domain:

      /var/lib/mailmen/lists.example.org/bin/newlist mailman

   Ignore the instructions about modifying your aliases file, it is done
   later.

8. Fix permissions (or archives won't work) - must be done after creating
   any list!

      chown -R list /var/lib/mailmen/lists.simpleid.org/archives/private/*

   of to do all lists at once:

      chown -R list /var/lib/mailmen/*/archives/*

9. Update the aliases file for the mailer

      Run the enclosed gen-mapped-aliases script

   Manually check the results in /etc/mailmen

   NOTE: this script must be run every time a new list is created
         with newlist or through the web.  Consider running
         it from cron.

10. Reload the mailer after adding any new virtual domain:

      service postfix reload

11. Add Mailman config to the Apache virtual host, note that you must
    use the cgi-bin path corresponding to the virtual host.

    See the files apache2.conf and apache2-vhost.conf for
    specific examples that are ready to use.

Copyright (C) 2013 Daniel Pocock http://danielpocock.com

Licensed under the GNU General Public License (GPL) v3.0 or later.

Parts of these scripts adapted from the Mailman package in Debian

