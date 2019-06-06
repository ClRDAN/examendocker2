# version: 1.0
# Aitor Galilea@edt M11-SAD
# EXAMEN UF4. ALTA DISPONIBILIDAD (DOCKER)
#--------------------------------------
FROM fedora:27
LABEL author="Aitor Galilea@edt ASIX M11 2018-19"
LABEL description="servidor de correo uw-imap"
RUN dnf -y install uw-imap telnet
RUN mkdir /opt/docker
COPY * /opt/docker/
RUN chmod +x /opt/docker/install.sh /opt/docker/startup.sh
CMD ["/opt/docker/startup.sh"]
WORKDIR /opt/docker 
