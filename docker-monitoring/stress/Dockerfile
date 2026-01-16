FROM almalinux:9.7
RUN dnf update -y
RUN dnf install -y epel-release
RUN dnf install -y stress
ENTRYPOINT ["stress"]
CMD []