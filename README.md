# mwarechk
It provides the ability to check the middleware‘s status, such as mycat/mysql and so on, and use it as a web by keepalived's healthchecker.  

1 Download this script    
  
2 Grant privileges  
chmod 755 /mysql/component/mysql01/bin/mysql  
chmod 777 /mysql/component/mwarechk -R  
  
3 Modify /etc/services  
mwarechk 9077/tcp # Check MWare's status  
  
4 Create xinetd's file    
[mycat@db01 ~]$ cat /etc/xinetd.d/mwarechk   
service mwarechk  
{  
        flags           = REUSE  
        socket_type     = stream  
        port            = 9077   
        wait            = no  
        user            = nobody  
        server          = /mysql/component/mwarechk/mwarechk  
        log_on_failure  += USERID  
        disable         = no  
        per_source      = UNLIMITED  
}  
  
5 restart xinetd  
Shell>service xinetd restart  
  
6 Test  
[mycat@db01 ~]$ wget --server-response http://133.128.174.68:9077/mycat_status  
Connecting to 192.168.0.122:9077... connected.  
HTTP request sent, awaiting response...   
  HTTP/1.0 200 OK  
Length: unspecified  
Saving to: ‘mycat_status’  
  
    [ <=>                                   ] 0           --.-K/s   in 0s        
  
2015-11-29 16:43:37 (0.00 B/s) - ‘mycat_status’ saved [0]  
  
7 Modify keepalived.conf  
[root@lvs-001 keepalived]# cat keepalived.conf   
	...  
virtual_server 192.168.0.140 8066 {  
	...  
  
    real_server 192.168.0.122 8066 {  
        weight 3  
        HTTP_GET {  
	    url {  
		path /mycat_status  
		status_code 200  
	    }  
            connect_timeout 3  
            nb_get_retry 3  
            delay_before_retry 3  
            connect_port 9077  
        }  
    }  
  
    real_server 192.168.0.123 8066 {  
		...  
    }  
}  
