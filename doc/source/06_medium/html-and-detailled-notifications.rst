.. _medium/html-and-detailled-notifications:

Shinken email notification plugin
___________________________________

Tired of brut text email notification, want to add more details to them and even make them readable for the common mortals. Then you can use the plugin shipped with Shinken. It can notify as usual in text and also in html in a table with all details needed from the service or host object.

.. image:: /_static/images/shinken-html-notification.png

Here it is the command you can use to have a basic html notification. You have to specify usual shinken macros in option. If not specified then it will try to get them from environment variable if you have set option ``enable_environment_macros`` in ``shinken.cfg`` if you want to used them. It isn't recommanded to use them for large environment. You better use option ``-c``, ``-o`` and ``-s`` or ``-h`` depend on which object you'll notify. Each macros separated by double comma.

Here is commands examples ready to use with it :

::


    define command {
        command_name    host-by-email
        command_line    $PLUGINSDIR$/notify_by_email.py -n host -S localhost -r $CONTACTEMAIL$ -f html -c "$NOTIFICATIONTYPE$,,$HOSTNAME$,,$HOSTADDRESS$,,$LONGDATETIME$"" -o ""$HOSTSTATE$,,$HOSTDURATION$"
    }

    define command {
        command_name    service-by-email
        command_line    $PLUGINSDIR$/notify_by_email.py -n service -S localhost -r $CONTACTEMAIL$ -f html -c "$NOTIFICATIONTYPE$,,$HOSTNAME$,,$HOSTADDRESS$,,$LONGDATETIME$" -o "$SERVICEDESC$,,$SERVICESTATE$,,$SERVICEOUTPUT$,,$SERVICEDURATION$"
    }

Add an header logo
~~~~~~~~~~~~~~~~~~~~

Let's say that you want to deliver mail to a customer and integrate its logo into, you only have to get a logo, named it ``customer_logo.png`` and paste it into ``/var/lib/shinken/share/images/``.


Detailled Notifications
--------------------------

Detailled notifications is a way to customize and add useful informations in email notification send to your contacts. To do so, use 3 objects macros, services or hosts :

- **_DETAILLEDESC** : Complete and add a more detailled description of service or host.
- **_IMPACT**       : Specify what will be the impact
- **_FIXACTIONS**   : And what is the recommended actions to do when there is an alert about it.

Example 
~~~~~~~~

For example, you can have the below configuration:
our service :

::

  
    define service {
        service_description   Cpu
        use                   20min_long,linux-service
        register              0
        host_name             linux-snmp
        check_command         check_linux_cpu

        _DETAILLEDESC         Detect abnormal CPU usage
        _IMPACT               Slow down applications hosted by the system
        _FIXACTIONS           If recurrent situation then make performance audit
    }

Define our detailled notifications commands adding the detailled informations and used with Shinken plugin so you will have an html email :

::


    ## Notify Host by Email with detailled informations
    # Service have appropriate macros.
    define command {
        command_name    detailled-host-by-email
        command_line    $PLUGINSDIR$/notify_by_email.py -n host -S localhost -r $CONTACTEMAIL$ -f html -c "$NOTIFICATIONTYPE$,,$HOSTNAME$,,$HOSTADDRESS$,,$LONGDATETIME$"" -o ""$HOSTSTATE$,,$HOSTDURATION$" -d "$_HOSTDETAILLEDDESC$" -i "$_HOSTIMPACT$"
    }

    ## Notify Service by Email with detailled informations
    # Service have appropriate macros.
    define command {
        command_name    detailled-service-by-email
        command_line    $PLUGINSDIR$/notify_by_email.py -n service -S localhost -r $CONTACTEMAIL$ -f html -c "$NOTIFICATIONTYPE$,,$HOSTNAME$,,$HOSTADDRESS$,,$LONGDATETIME$" -o "$SERVICEDESC$,,$SERVICESTATE$,,$SERVICEOUTPUT$,,$SERVICEDURATION$" -d "$_SERVICEDETAILLEDESC$" -i "$_SERVICEIMPACT$" -a "$_SERVICEFIXACTIONS$"
    }

And now define our notification ways:

::


    define notificationway{
       notificationway_name            detailled-email
       service_notification_period     24x7
       host_notification_period        24x7
       service_notification_options    c,w,r
       host_notification_options       d,u,r,f,s
       service_notification_commands   detailled-service-by-email ; send service notifications via email
       host_notification_commands      detailled-host-by-email    ; send host notifications via email
    }  

Then you'll receive a nice html mail giving all your details in a table.
