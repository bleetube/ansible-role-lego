# selinux

tl;dr nginx wants

```
sudo semanage fcontext -a -t httpd_sys_content_t "/var/acme(/.*)?"
sudo semanage fcontext -a -t httpd_var_run_t "/var/run/nginx.pid"
sudo restorecon -R /var/acme
sudo semanage port -a -t http_port_t -p tcp 4430-4439
```

## File system access
On RedHat 7.9, in order to permit nginx to read `/var/acme`:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/var/acme(/.*)?"
sudo restorecon -R /var/acme
```

This is because its in the `` context:

```bash
$ ls -Z /usr/sbin/nginx
-rwxr-xr-x. root root system_u:object_r:httpd_exec_t:s0 /usr/sbin/nginx
```

Also, it needs access to write a PID file:

```
nginx: [emerg] open() "/var/run/nginx.pid" failed (13: Permission denied)
```

That can be added as well:

```
semanage fcontext -a -t httpd_var_run_t "/var/run/nginx.pid"
restorecon -v /var/run/nginx.pid
```

## Network port utilization

```
nginx: [emerg] bind() to 10.100.102.100:4430 failed (13: Permission denied)
```

Another change that was necessary was to permit nginx to listen on an unpriveled port.

```
semanage port -l | grep http_port_t
sudo semanage port -a -t http_port_t -p tcp 4430-4439
```

And proxy_pass also gets blocked:

```
*126 connect() to 127.0.0.1:8083 failed (13: Permission denied) while connecting to upstream
```

Workaround:

```
sudo setsebool -P httpd_can_network_connect 1
```