# Share a directory with SFTP.
#
# docker build -t sftp .
# docker run -p 2222:22 -v /tmp/shared:/jail/user/user --rm -it sftp
# sftp -i ./id_ecdsa -P 2222 user@127.0.0.1

FROM alpine:3.14

ARG UID=1000
ARG PUBKEY="ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJYsfeKPM/vw3e2IgLFjO9XRAIXZcUqGJaloee6+UATCU6PY4yRsti+G9vW6JjLmUk/cJ7R9BOg2KfgPiGnvB7c="

RUN apk --update add --no-cache openssh bash && rm -rf /var/cache/apk/*

# add user and unlock the account
RUN adduser -D -u $UID -s /bin/false -H -h /user user && passwd -u user

# https://unix.stackexchange.com/a/542507
RUN mkdir -m 755 -p /jail/user/user && \
    chown root:root /jail /jail/user && \
    chown user: /jail/user/user && \
    chmod 750 /jail/user/user && \
    ln -s /jail/user/user /user

# create sshd keys and update config
RUN /usr/bin/ssh-keygen -A && \
    ssh-keygen -t rsa -b 4096 -f /etc/ssh/ssh_host_key && \
    echo -e 'Match User user\n ChrootDirectory /jail/user\n ForceCommand internal-sftp\n AllowTcpForwarding no\n AuthorizedKeysFile /jail/user/authorized_keys' >>/etc/ssh/sshd_config && \
    echo $PUBKEY >>/jail/user/authorized_keys

# uncomment to add the same SSH key for the root user
#RUN mkdir /root/.ssh && echo $PUBKEY >/root/.ssh/authorized_keys

CMD ["/usr/sbin/sshd", "-e", "-D"]
