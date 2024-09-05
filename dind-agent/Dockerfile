FROM jenkins/jnlp-agent-docker@sha256:eb9ab5139a598c3a210d5a491d3cc31db501cb870b011035e9cdbc0b5e661dfa
ENV PYTHONUNBUFFERED=1
USER root


COPY entrypoint.sh /entrypoint.sh
RUN chown jenkins:jenkins /entrypoint.sh
RUN chmod +x /entrypoint.sh

RUN apk add --update --no-cache python3 py3-pip && ln -sf python3 /usr/bin/python

RUN pip3 install --no-cache --upgrade pip setuptools "jenkins-job-builder==5.1.0" --break-system-packages
RUN export PATH="/home/jenkins/.local/bin:$PATH"

USER jenkins
ENTRYPOINT ["./entrypoint.sh"]
