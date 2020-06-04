# docker-selenium-grid-node-chrome

Setup a Chrome Selenium Grid Node in a docker container and connect it to a Selenium Grid Hub.
VNC will be available into the the node (password is `secret`) on the specified port.

You can access the Node's sessions at an address similar to this: http://selennode.COMPANY.com:5555/wd/hub/static/resource/hub.html

For more information: https://github.com/SeleniumHQ/docker-selenium

## Requirements

You have a working Selenium Grid Hub and the host is setup as a docker host.

## Role Variables

### REQUIRED

* docker_selenium_grid_node_chrome_dns: The DNS name to assign to the node.
* docker_selenium_grid_node_chrome_port: The port to assign to the node.
* docker_selenium_grid_node_chrome_hub_dns: The DNS name of the Selenium Grid Hub to connect with.
* docker_selenium_grid_node_chrome_hub_port: The port to use for connections with the Selenium Grid Hub.

### OPTIONAL

* docker_selenium_grid_node_chrome_version: The version of the selenium grid to use. Defaults to latest.

* docker_selenium_grid_node_chrome_instances: This is the number of executors the node provides to the hub.
  It is recommended as a best practice to only allow one at a time as all container resources 
  will be used for that browser instance, and this helps to have more stable tests. Therefore,
  this defaults to 1.

* docker_selenium_grid_node_chrome_vnc_port: The port for VNC connections. Defaults to 5900.

* docker_selenium_grid_node_chrome_screen_width: The width of the display. This is useful if you wish
  to maximize the webpage without setting the browser width directly. Defaults to 1920.
  
* docker_selenium_grid_node_chrome_screen_height: The height of the display. This is useful if you wish
  to maximize the webpage without setting the browser width directly. Defaults to 1080.

* docker_selenium_grid_node_chrome_docker_build_dir: The directory to build to chrome node docker container.
  Use to specify a different path per node if you want to run multiple nodes on a single docker host.

* docker_selenium_grid_node_chrome_container_name: The name to give the node container. 
  Use to specify a different name per node if you want to run multiple nodes on a single docker host.

* docker_selenium_grid_node_chrome_project_name: The name to give the docker compose project.
  Again, use to specify a different compose project if you want to run multiple nodes on a single docker host.

* docker_selenium_grid_node_chrome_restart_always: If yes, restart the OS when applying this role. Defaults to no.
  This is useful if you fear that the grid is having longevity problems, but it will bounce any VNC connection
  to the node running at the time. If the VNC client can't cope with that, you'll have to manually reconnect.

* docker_selenium_grid_node_chrome_nssdb: If this variable is defined, it is a path to a folder containing files to places in `/home/seluser/.pki/nssdb`.
  This path is assumed to be on the ansible originator/master, not on the ansible target machine.
  
  Despite my best efforts, I could not figure out how to script a public CA certificate into the docker
  container so Chrome could then properly validate the certificate of the page Selenium was telling it
  to access. To deal with this shortcoming, I started up the container, opened Chrome, 
  and went through Chrome's settings to add the pub for our internal CA. 
  Once it was added, I SCPed off the `/home/seluser/.pki/nssdb`
  folder for use with this role.
  
  This is useful if for some reason you do not wish to use Selenium's ability to ignore invalid certs
  and you happen to be automating against a web service with a certificate chain (i.e. not self-signed) and part
  of the chain is untrusted by Chrome because the pub is not in its cert database.

* docker_selenium_grid_node_chrome_test_files: If this variable is defined, it is a path to a folder 
  containing files which should be mounted into the container at `/home/seluser/test_files/`. 
  The path `/home/seluser/test_files/` is the directory your Selenium tests will need to reference.
  This process implies that you've already placed your files on the docker host prior to running this role.
  This is useful if your web application can do file uploads and you wish to test that with Selenium.
  
  Note that if the placement of the folder on the docker host is destructive (i.e. the folder is deleted
  and recreated every time you configure with Ansible, while the node's docker container is running), 
  this is problematic for the docker mount to this directory.
  Your docker container will not cope well with the deletion of the mount point while it's running.
  You can either use the `docker_selenium_grid_node_chrome_restart_always` option, which should solve this problem by forcing a restart of the container
  upon OS reboot, or you can place your test files in a subfolder of the path specified by this variable, 
  and delete/recreate that subfolder. If the mount was on a folder that was recreated every build, 
  docker wouldn't honor the mount unless the container was restarted. 
  That restart is something to avoid because it will bounce every VNC connection made to the container.
  
  I would suggest using Artifactory to host the test files, and perform a couple tasks prior to running this role
  that deletes and recreates the folder and re-downloads all the files from Artifactory using jfrog cli, 
  ensuring you have nothing stale on the Selenium Grid nodes. 
  If you set this playbook to run, say, nightly, you would ensure that your test
  files are relatively up-to-date, your Selenium Grid has access to them, and your Grid is always running.

## Dependencies

None

## Example Playbook

    - hosts: servers
      vars:
        docker_selenium_grid_node_chrome_dns: selennode.COMPANY.com
        docker_selenium_grid_node_chrome_port: 5555
        docker_selenium_grid_node_chrome_hub_dns: selenhub.COMPANY.com
        docker_selenium_grid_node_chrome_hub_port: 4444
      roles:
         - role: docker-selenium-grid-hub

## License

BSD-3-Clause

## Author Information

Jeremy Cornett <jeremy.cornett@forcepoint.com>
