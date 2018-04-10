Nagios check sphinxsearch
=========================

Requirements 
------------
Perl, libsphinx-search-perl

Usage
-----

```
check_sphinxsearch_query -w <warn> -c <crit> -H <server IP/Name> -q <query> -t <timeout>
  Checks the number of founded results for query
  -w (--warning)   = Min. number of results in queue to generate warning
  -c (--critical)  = Min. number of results in queue to generate critical alert ( w < c )
  -q (--query) = ('Prague' by default)
  -H (--host) = Server name, IP address, or path to Unix socket (default 127.0.0.1)
  -t (--timeout)  = Set server connection timeout(1000 by default).
  -h (--help)
Example: ./check_sphinxsearch_query -w 200 -c 150 -H localhost -q why
```

Nagios configuration
--------------------

On Sphinxsearch server with nrpe:

edit /etc/nagios/nrpe.d/sphinxsearch_query.cfg:
  
```
command[check_sphinxsearch_query]=/usr/lib/nagios/wikidi/check_sphinxsearch_query -H $ARG1$ -q $ARG2$ -w $ARG3$ -c $ARG4$
```

mkdir /usr/lib/nagios/wikidi/
upload check_sphinxsearch_query to /usr/lib/nagios/wikidi/

On nagios server monitor:

edit /etc/nagios-plugins/config/check_nrpe.cfg:
  
```
define command {
 command_name    check_nrpe_4arg
 command_line    /usr/lib/nagios/plugins/check_nrpe -H $HOSTADDRESS$ -c $ARG1$ -a $ARG2$ $ARG3$ $ARG4$ $ARG5$
}
```

edit /etc/nagios3/conf.d/<servername>.cfg :

```
define service {
  host_name                       <hostname>
  service_description             Sphinxsearch query
  check_command                   check_nrpe_4arg!check_sphinxsearch_query!<hostname/IP>!<query>!<warning>!<critical>
  # Example:             			check_nrpe_4arg!check_sphinxsearch_query!127.0.0.1!wtf!350!100
  use                             generic-service
  notification_interval           0
}
```

Icinga2 configuration
---------------------

Create a new `CheckCommand` configuration in your zone:

```
object CheckCommand "sphinxsearch_query" {
    import "plugin-check-command"
    command = [ PluginDir + "/check_sphinxsearch_query" ]

    arguments = {
        "--warning" = {
            value = "$sphinxsearch_query_warning$"
            description = "Min. number of results in queue to generate warning"
            required = true
        }
        "--critical" = {
            value = "$sphinxsearch_query_critical$"
            description = "Min. number of results in queue to generate critical alert ( w > c )"
            required = true
        }
        "--query" = {
            value = "$sphinxsearch_query_query$"
            description = "'Prague' by default"
        }
        "--host" = {
            value = "$sphinxsearch_query_host$"
            description = "Server name, IP address, or path to Unix socket (default 127.0.0.1)"
        }
        "--timeout" = {
            value = "$sphinxsearch_query_timeout$"
            description = "Server name, IP address, or path to Unix socket (default 127.0.0.1)"
        }
    }
    vars.sphinxsearch_query_warning = 10
    vars.sphinxsearch_query_critical = 5
    vars.sphinxsearch_query_host = "localhost"
}
```

Define a new service and assign it to all host with vars.sphinxsearch:
```
apply Service "Sphinx search query" {
  import "generic-service"
  vars.sphinxsearch_query_query = "test"
  check_command = "sphinxsearch_query"
  command_endpoint = host.vars.client_endpoint
  assign where host.vars.client_endpoint && host.vars.sphinxsearch
}
```
