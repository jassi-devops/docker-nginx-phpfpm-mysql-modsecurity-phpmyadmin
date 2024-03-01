# Use the phpMyAdmin image as the base
FROM phpmyadmin/phpmyadmin:latest

# Install OpenSSH server and supervisord
USER root
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y openssh-server supervisor net-tools \
    && rm -rf /var/lib/apt/lists/*

# Configure SSH server
RUN mkdir /var/run/sshd \
    && echo 'root:your_root_password' | chpasswd \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

RUN  a2enmod ssl
ADD conf /etc/apache2/sites-available
COPY config.secret.inc.php /etc/phpmyadmin
COPY cert /etc/apache2/ssl/
# Copy supervisord configuration file
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Expose ports for phpMyAdmin, SSH, and supervisord
EXPOSE 80 22 443

# Start supervisord
CMD ["/usr/bin/supervisord"]
