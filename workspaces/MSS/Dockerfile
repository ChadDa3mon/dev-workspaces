FROM mss-docker.artifactory.swg-devops.com/mss-ansible/docker-dev-env:latest

COPY files/unified-supervisord.conf files/supervisord.conf /etc/supervisord/
COPY files/mc.ini /home/mss/.config/mc/ini
COPY files/zsh-in-docker.sh /tmp/zsh-in-docker.sh
COPY files/sshd_config /etc/ssh/
COPY files/Cronicle-0.8.61 /opt/cronicle
COPY files/cronicle-config.json /opt/cronicle/conf/config.json
COPY files/supervisord-workspace-base.conf /etc/supervisord/
COPY files/filebrowser.json /opt/filebrowser/.filebrowser.json
COPY files/mkdocs /home/docs
COPY files/mkdocs-requirements.txt /home/mss/installed-python-packages/mkdocs-requirements.txt
COPY files/infra-requirements.txt /home/mss/installed-python-packages

# Install some packages
RUN rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm \
    && yum update \
    && yum install -y nmap-ncat \
    && yum install -y jq \
    && yum install -y zip \
    && yum install -y microdnf \
    && yum install -y vim \
    && yum install -y ncdu \
    && yum install -y htop \
    && yum install -y openssh-server \
    && yum install -y nano 

# Install Docker systemctl
RUN wget https://raw.githubusercontent.com/gdraheim/docker-systemctl-replacement/master/files/docker/systemctl3.py -O /usr/local/bin/systemctl  \
    && chmod 0777 /usr/local/bin/systemctl 

# Create MSS user and setup permissions
RUN useradd -u 8877 mss \
    && chmod -R 777 /home \
    && mkdir -p /home/mss \
    && chown -R mss /home/mss 

# Modify sudoers to allow mss user to run some basic commands
RUN  echo "------------------------------------------------------ Allow users some priviledged actions" \
    && echo "# Allow non-admin users to install packages" >> /etc/sudoers \
    && echo "mss ALL = NOPASSWD : /usr/bin/apt, /usr/bin/apt-get, /usr/bin/aptitude, /usr/bin/add-apt-repository, /usr/local/bin/pip, /usr/local/bin/systemctl, /usr/bin/dpkg, /usr/sbin/dpkg-reconfigure" >> /etc/sudoers 

# Installing some other stuff
RUN pip3 install nodeenv \
    && microdnf install nodejs \
    && npm install yarn -g \
    && pip3 install supervisor==4.2.2 \
    && pip3 install glances==3.2.5 \
    && pip3 install vizex==2.1.0  \
    && pip3 install -r /home/mss/installed-python-packages/infra-requirements.txt

# Setup Supervisord
RUN mkdir -p /var/log/supervisord/ \
    && chmod -R 777 /var/log \
    && chmod gu+rw /var/run \
    && chmod -R 777 /etc/supervisord \
    && chmod -R 777 /var/log/supervisord

# Setup ZSH
RUN HOME=/root \
    && wget https://vault.centos.org/centos/8/BaseOS/x86_64/os/Packages/zsh-5.5.1-6.el8_1.2.x86_64.rpm \
    && rpm -i zsh-5.5.1-6.el8_1.2.x86_64.rpm \
    && rm zsh-5.5.1-6.el8_1.2.x86_64.rpm \
    && chmod +x /tmp/zsh-in-docker.sh \
    && /tmp/zsh-in-docker.sh \
    -t https://github.com/pascaldevink/spaceship-zsh-theme \
    -a 'SPACESHIP_PROMPT_ADD_NEWLINE="false"' \
    -a 'SPACESHIP_PROMPT_SEPARATE_LINE="false"' \
    -a 'export LS_COLORS="$LS_COLORS:ow=1;34:tw=1;34:"' \
    -a 'SPACESHIP_USER_SHOW="false"' \
    -a 'SPACESHIP_TIME_SHOW="true"' \
    -a 'SPACESHIP_TIME_COLOR="grey"' \
    -a 'SPACESHIP_DIR_COLOR="cyan"' \
    -a 'SPACESHIP_GIT_SYMBOL="⇡"' \
    -a 'SPACESHIP_BATTERY_SHOW="false"' \
    -a 'if [[ $(pwd) != /root  ]]; then cd /root; fi  # Set starting dir to /root ' \
    -a 'hash -d r=/root' \
    -p git \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions \
    -p https://github.com/zsh-users/zsh-history-substring-search \
    -p https://github.com/zsh-users/zsh-syntax-highlighting \
    -p 'history-substring-search' \
    -p https://github.com/bobthecow/git-flow-completion \
    -a 'bindkey "\$terminfo[kcuu1]" history-substring-search-up' \
    -a 'bindkey "\$terminfo[kcud1]" history-substring-search-down' \
    && printf '%s\n%s\n' "export ZSH_DISABLE_COMPFIX=true" "$(cat /root/.zshrc)" > /root/.zshrc \
    && echo "------------------------------------------------------ ZSH mss" \
    && mkdir -p /home/project \
    && HOME=/home/mss \
    && /tmp/zsh-in-docker.sh \
    -t https://github.com/pascaldevink/spaceship-zsh-theme \
    -a 'DISABLE_UPDATE_PROMPT="true"' \
    -a 'SPACESHIP_PROMPT_ADD_NEWLINE="false"' \
    -a 'SPACESHIP_PROMPT_SEPARATE_LINE="false"' \
    -a 'export LS_COLORS="$LS_COLORS:ow=1;34:tw=1;34:"' \
    -a 'SPACESHIP_USER_SHOW="true"' \
    -a 'SPACESHIP_TIME_SHOW="true"' \
    -a 'SPACESHIP_TIME_COLOR="grey"' \
    -a 'SPACESHIP_DIR_COLOR="cyan"' \
    -a 'SPACESHIP_GIT_SYMBOL="⇡"' \
    -a 'SPACESHIP_BATTERY_SHOW="false"' \
    -a 'if [[ $(pwd) != /home/project  ]]; then cd /home/project; fi  # Set starting dir to /home/project ' \
    -a 'hash -d p=/home/project' \
    -p git \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions \
    -p https://github.com/zsh-users/zsh-history-substring-search \
    -p https://github.com/zsh-users/zsh-syntax-highlighting \
    -p 'history-substring-search' \
    -p https://github.com/bobthecow/git-flow-completion \
    -a 'bindkey "\$terminfo[kcuu1]" history-substring-search-up' \
    -a 'bindkey "\$terminfo[kcud1]" history-substring-search-down' \
    && rm /tmp/zsh-in-docker.sh \
    && printf '%s\n%s\n' "export ZSH_DISABLE_COMPFIX=true" "$(cat /home/mss/.zshrc)" > /home/mss/.zshrc 

# Setup Web-Based Terminal
RUN cd /tmp \
    && wget https://github.com/tsl0922/ttyd/releases/download/1.6.3/ttyd.x86_64 \
    && mv ttyd.x86_64 /usr/bin/ttyd \
    && chmod +x /usr/bin/ttyd 

# Setup Cronicle
RUN mkdir -p /opt/cronicle \
	&& cd /opt/cronicle && nodeenv --node=12.18.3 --npm=6.0.0 env \
	&& cd /opt/cronicle && . env/bin/activate && npm install; node bin/build.js dist

# Setup ungit
RUN mkdir -p /opt/ungit \
	&& cd /opt/ungit && nodeenv --node=12.18.3 --npm=6.0.0 env \
	&& cd /opt/ungit && . env/bin/activate && npm install -g ungit@1.5.9 

# Setup static file server
RUN mkdir -p /opt/serve \
	&& cd /opt/serve && nodeenv --node=12.18.3 --npm=6.0.0 env \
	&& cd /opt/serve && . env/bin/activate &&  npm install -g serve 

# Setup mkdocs
RUN pip3 install -r /home/mss/installed-python-packages/mkdocs-requirements.txt 
COPY files/mkdocs/mkdocs.yml /home/docs/mkdocs.yml
COPY files/mkdocs/README.md /home/docs/docs/README.md
COPY files/mkdocs/Ara.png /home/docs/docs/assets/home/
COPY files/mkdocs/Blast-radius.png /home/docs/docs/assets/home/
COPY files/mkdocs/Terraform-Rover.png /home/docs/docs/assets/home/
COPY files/mkdocs/helpers.py /home/docs/macros

# Ara
COPY files/ara-settings.yaml /home/mss/.ara/server/settings.yaml
ENV ANSIBLE_CALLBACK_PLUGINS="/usr/local/lib/python3.8/site-packages/ara/plugins/callback" ARA_API_CLIENT="http" ARA_API_SERVER="http://0.0.0.0:8029" ARA_TIME_ZONE="UTC"

RUN echo "UTC" > /etc/timezone 
#    && mkdir -p /home/mss/.ara/server \
#    && ara-manage makemigrations  \
#    && ara-manage migrate 

# Setup User Stuff
RUN chown mss /home/project \
    && chmod 777 /etc/supervisord/ \
    && mkdir -p /home/mss/.local/bin \ 
    && chmod 755 /home/mss/.local && chmod 755 /home/mss/.local/bin \
    && chown mss /home/mss/.local && chown mss /home/mss/.local/bin  \
    && find /home -type d | xargs -I{} chown -R mss {} \
    && find /home -type f | xargs -I{} chown mss {} \
    && rm -rf /home/mss/.oh-my-zsh/.git \
    && rm -rf /home/mss/.oh-my-zsh/.github \
    && rm -rf /home/mss/.oh-my-zsh/custom/plugins/git-flow-completion/.git \
    && rm -rf /home/mss/.oh-my-zsh/custom/plugins/zsh-autosuggestions/.git \
    && rm -rf /home/mss/.oh-my-zsh/custom/plugins/zsh-completions/.git \
    && rm -rf /home/mss/.oh-my-zsh/custom/plugins/zsh-history-substring-search/.git \
    && rm -rf /home/mss/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/.git \
    && rm -rf /home/mss/.oh-my-zsh/custom/themes/spaceship-zsh-theme/.git \
    && mkdir -p /home/static-server \
	&& chown -R mss /opt/cronicle \
	&& chown -R mss /opt/filebrowser \
	&& chown -R mss /home/static-server \
	&& chown -R mss /home/docs \
	&& chown -R mss /opt/ungit \
	&& chown -R mss /opt/serve \
	&& mkdir -p /var/log/cronicle && chown -R mss /var/log/cronicle \
	&& mkdir -p /var/log/filebrowser && chown -R mss /var/log/filebrowser \
	&& mkdir -p /var/log/ungit && chown -R mss /var/log/ungit \
	&& mkdir -p /var/log/static-file-server && chown -R mss /var/log/static-file-server \
	&& mkdir -p /var/log/mkdocs && chown -R mss /var/log/mkdocs \
    && chown -R mss /home/mss/installed-python-packages 

# More User Stuff
RUN chown -R mss /home/mss/.ara/server \
    #&& chown -R mss /opt/theia \
    #&& mkdir -p /var/log/theia && chown -R mss /var/log/theia \
    #&& mkdir -p /var/log/ara/ && chown -R mss /var/log/ara/ \
    && chown -R mss /home/docs \
    #&& chown -R mss /home/mss/utils \
    && chown -R mss /home/mss/installed-python-packages \
    && find /home -type d | xargs -I{} chown -R mss {} \
    && find /home -type f | xargs -I{} chown mss {}

# Setup SSH Server
RUN mkdir -p /var/log/sshd  \
    && chmod -R 777 /var/log/
RUN /usr/bin/ssh-keygen -A -v

USER mss
EXPOSE 3222
ENV PATH="/home/mss/.local/bin:${PATH}"
ENTRYPOINT /etc/init.d/cron start; supervisord -c "/etc/supervisord/unified-supervisord.conf"