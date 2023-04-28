# Issues

Issues are outside the scope of this tutorial so details will be kept light.

## No Available Socket when Installing Docker

**Overview**

Installing Docker via apt will automatically start the Docker service.  Sometimes the service has an issue of obtaining a socket as seen in the journal logs below.      

This feels like a race condition as the error only occurs sometimes.  The Ansible playbook includes error handling (via the `rescue` tag) to handle the error.

```bash
# Get service logs
systemctl status docker.service

# Output:
Apr 27 17:26:52 computer systemd[1]: docker.service: Scheduled restart job, restart counter is at 3.
Apr 27 17:26:52 computer systemd[1]: Stopped Docker Application Container Engine.
Apr 27 17:26:52 computer systemd[1]: docker.service: Start request repeated too quickly.
Apr 27 17:26:52 computer systemd[1]: docker.service: Failed with result 'exit-code'.
Apr 27 17:26:52 computer systemd[1]: Failed to start Docker Application Container Engine.

###

# Get journal logs
sudo journalctl -u docker -n 100 --no-pager

# Output:
# As seen below, the docker daemon has trouble getting a socket
Apr 27 17:26:45 computer systemd[1]: Starting Docker Application Container Engine...
Apr 27 17:26:45 computer dockerd[102901]: time="2023-04-27T17:26:45.467419339-05:00" level=info msg="Starting up"
Apr 27 17:26:45 computer dockerd[102901]: failed to load listeners: no sockets found via socket activation: make sure the service was started by systemd
Apr 27 17:26:45 computer systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
Apr 27 17:26:45 computer systemd[1]: docker.service: Failed with result 'exit-code'.
Apr 27 17:26:45 computer systemd[1]: Failed to start Docker Application Container Engine.
Apr 27 17:26:47 computer systemd[1]: docker.service: Scheduled restart job, restart counter is at 1.
Apr 27 17:26:47 computer systemd[1]: Stopped Docker Application Container Engine.
Apr 27 17:26:47 computer systemd[1]: Starting Docker Application Container Engine...
Apr 27 17:26:47 computer dockerd[102954]: time="2023-04-27T17:26:47.710684609-05:00" level=info msg="Starting up"
Apr 27 17:26:47 computer dockerd[102954]: failed to load listeners: no sockets found via socket activation: make sure the service was started by systemd
Apr 27 17:26:47 computer systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
Apr 27 17:26:47 computer systemd[1]: docker.service: Failed with result 'exit-code'.
Apr 27 17:26:47 computer systemd[1]: Failed to start Docker Application Container Engine.
Apr 27 17:26:49 computer systemd[1]: docker.service: Scheduled restart job, restart counter is at 2.
Apr 27 17:26:49 computer systemd[1]: Stopped Docker Application Container Engine.
Apr 27 17:26:49 computer systemd[1]: Starting Docker Application Container Engine...
Apr 27 17:26:49 computer dockerd[102991]: time="2023-04-27T17:26:49.964814548-05:00" level=info msg="Starting up"
Apr 27 17:26:49 computer dockerd[102991]: failed to load listeners: no sockets found via socket activation: make sure the service was started by systemd
Apr 27 17:26:49 computer systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
Apr 27 17:26:49 computer systemd[1]: docker.service: Failed with result 'exit-code'.
Apr 27 17:26:49 computer systemd[1]: Failed to start Docker Application Container Engine.
Apr 27 17:26:52 computer systemd[1]: docker.service: Scheduled restart job, restart counter is at 3.
Apr 27 17:26:52 computer systemd[1]: Stopped Docker Application Container Engine.
Apr 27 17:26:52 computer systemd[1]: docker.service: Start request repeated too quickly.
Apr 27 17:26:52 computer systemd[1]: docker.service: Failed with result 'exit-code'.
Apr 27 17:26:52 computer systemd[1]: Failed to start Docker Application Container Engine.
```

**Solution**

The following docs provide guidance:
- https://github.com/docker/for-linux/issues/989
- https://forums.docker.com/t/failed-to-load-listeners-no-sockets-found-via-socket-activation-make-sure-the-service-was-started-by-systemd/62505

In `/lib/systemd/system/docker.service`, change `fd://` to `unix://`.  Then execute `sudo systemctl daemon-reload` so systemd picks up the change.

Confirm with `systemctl status docker.service`.  The status should have an "active (running)" state.
