Nagios check sphinxsearch
=========================

Requirements 
------------
Perl, libsphinx-search-perl

Usage
-----

check_sphinxsearch_query -w <warn> -c <crit> -H <server IP/Name> -q <query> -t <timeout>

   Checks the number of founded results for query
   -w (--warning)   = Min. number of results in queue to generate warning
   -c (--critical)  = Min. number of results in queue to generate critical alert ( w < c )
   -q (--query) = ('Prague' by default)
   -H (--host) = Server name or IP(default 127.0.0.1) 
   -t (--timeout)  = Set server connection timeout(1000 by default).
   -h (--help)

Example: ./check_sphinxsearch_query -w 200 -c 150 -H localhost -q why


INSTALL
-------

On Sphinxsearch server with nrpe:

 cat /etc/nagios/nrpe.d/sphinxsearch_query.cfg
  command[check_sphinxsearch_query]=/usr/lib/nagios/wikidi/check_sphinxsearch_query -H $ARG1$ -q $ARG2$ -w $ARG3$ -c $ARG4$

 mkdir /usr/lib/nagios/wikidi/
 upload check_sphinxsearch_query to /usr/lib/nagios/wikidi/

On nagios server monitor:

 edit /etc/nagios-plugins/config/check_nrpe.cfg
  
  define command {
   command_name    check_nrpe_4arg
   command_line    /usr/lib/nagios/plugins/check_nrpe -H $HOSTADDRESS$ -c $ARG1$ -a $ARG2$ $ARG3$ $ARG4$ $ARG5$
  }

-------------------------------------------------
 edit /etc/nagios3/conf.d/<servername>.cfg:

   define service {
        host_name                       <hostname>
        service_description             Sphinxsearch query
        check_command                   check_nrpe_4arg!check_sphinxsearch_query!<hostname/IP>!<query>!<warning>!<critical>
        # Example: 			check_nrpe_4arg!check_sphinxsearch_query!127.0.0.1!wtf!350!100
        use                             generic-service
        notification_interval           0
   }
-------------------------------------------------
