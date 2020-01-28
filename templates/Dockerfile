FROM selenium/node-chrome-debug:{{ docker_selenium_grid_node_chrome_version }}

{% if docker_selenium_grid_node_chrome_nssdb is defined %}
# I was never able to successfully import the cert from the shell so Chrome would play nice with it.
# My solution was to start the container and use Chrome directly to import the cert, then copy off the cert database files.
USER root
RUN mkdir /home/seluser/.pki && chmod 700 /home/seluser/.pki && chown seluser:seluser /home/seluser/.pki
ADD nssdb /home/seluser/.pki/nssdb
RUN chmod 700 /home/seluser/.pki/nssdb && chown seluser:seluser /home/seluser/.pki/nssdb
RUN chmod 600 /home/seluser/.pki/nssdb/* && chown seluser:seluser /home/seluser/.pki/nssdb/*
USER seluser
{% endif %}