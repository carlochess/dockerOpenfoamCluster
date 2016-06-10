# ------------------------------------------------------------
# Construya esta imagen usando:  docker build -t carlochess/of_v30plus_rhel66 .
# ------------------------------------------------------------
FROM openfoamplus/of_v30plus_rhel66
MAINTAINER Carlos Roman <carlochess@gmail.com>

# ------------------------------------------------------------
# Instale SSH Server y Client
# ------------------------------------------------------------
RUN touch /var/lib/rpm/*
RUN yum -y install openssh-server openssh-clients sudo || true
RUN mkdir /var/run/sshd
RUN echo 'root:mpirun' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

# ------------------------------------------------------------
# Agregue el usuario 'mpirun' con privilegios de superusuario
# Con la contraseña 'mpirun' y el /home/ 'mpirun'
# ------------------------------------------------------------

RUN adduser mpirun && \
    echo "mpirun ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
RUN echo 'mpirun:mpirun' | chpasswd
RUN gpasswd -a mpirun wheel
ENV HOME /home/mpirun

RUN echo "source /opt/OpenFOAM/OpenFOAM-v3.0+/etc/bashrc" >> /etc/profile
RUN echo "source /opt/OpenFOAM/OpenFOAM-v3.0+/etc/bashrc" >> ~/.bashrc
RUN echo "sudo service sshd start" >> /opt/OpenFOAM/OpenFOAM-v3.0+/etc/bashrc
RUN echo ". ~/.bashrc" >> ~/.bash_profile
RUN cp ~/.bash_profile ~/.profile

# ------------------------------------------------------------
# Agregamos las llaves
# ------------------------------------------------------------

RUN mkdir /home/mpirun/.ssh/
ADD ssh/config /home/mpirun/.ssh/config
ADD ssh/id_rsa.mpi /home/mpirun/.ssh/id_rsa
ADD ssh/id_rsa.mpi.pub /home/mpirun/.ssh/id_rsa.pub
ADD ssh/id_rsa.mpi.pub /home/mpirun/.ssh/authorized_keys

RUN chmod -R 600 /home/mpirun/.ssh/* && \
    chown -R mpirun:mpirun /home/mpirun/.ssh

EXPOSE 22
COPY start.sh /start.sh 
RUN chmod -v +x /start.sh 
#CMD ["/start.sh"]
CMD ["/usr/sbin/sshd", "-D"]
