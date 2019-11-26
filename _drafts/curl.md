Sometimes is necessary to troubleshoot a website from the command line and for a system administrator, what are the cases that are useful?

First, let's talk a little bit about curl. 

Curl is a command line tool that allows you to transfer data from and to a server.


curl is a tool to transfer data from or to a server, using one of the supported protocols (DICT, FILE, FTP, FTPS, GOPHER, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, POP3, POP3S, RTMP, RTSP, SCP, SFTP, SMB, SMBS,
       SMTP, SMTPS, TELNET and TFTP). The command is designed to work without user interaction.

curl -s -L -D - http://accept-myshimano.aws.shimano-eu.com/ -o /dev/null -w '%{url_effective}'




-H Header
-d data
-X request (request method)