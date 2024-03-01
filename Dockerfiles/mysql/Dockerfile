FROM mysql:8.0.36-debian

# Install OpenSSH server and supervisord
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y openssh-server supervisor \
    && rm -rf /var/lib/apt/lists/*

# Configure SSH server
RUN mkdir /var/run/sshd \
    && echo 'root:your_root_password' | chpasswd \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Explicitly set the MySQL data directory
ENV MYSQL_DATADIR=/var/lib/mysql

# Set a secure root password during build
ENV MYSQL_ROOT_PASSWORD=your_secure_root_password

# Expose ports for MySQL and SSH
EXPOSE 3306 22

# Copy supervisord configuration file
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Start supervisord
CMD ["/usr/bin/supervisord"]
