FROM alpine:edge
RUN apk --no-cache add tree jq less vim htop curl wget bash python
# RUN curl -O https://bootstrap.pypa.io/get-pip.py && python3 get-pip.py --user
RUN apk --no-cache add python py-pip
RUN pip install --upgrade pip setuptools
RUN pip --no-cache-dir install awscli

# FROM ubuntu:bionic
# RUN apt update && apt install -y tree jq less watch vim htop nodejs ruby curl wget bash coreutils openssl python3 python3-pip awscli

# Latest stable version: https://github.com/mikefarah/yq/releases/
ENV YQ_VERSION 2.4.0
RUN curl -LO https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64 \
    && chmod +x ./yq_linux_amd64 \
    && mv ./yq_linux_amd64 /usr/bin/yq

# Latest stable version: https://github.com/mozilla/sops/releases
# ENV SOPS_VERSION 3.3.0
# RUN wget -O sops https://github.com/mozilla/sops/releases/download/${SOPS_VERSION}/sops-${SOPS_VERSION}.linux \
#     && chmod +x ./sops \
#     && mv ./sops /usr/bin/sops

# Latest stable version: curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest | grep tag_name | cut -d '"' -f 4 | tr --delete v
ENV TERRAFORM_VERSION=0.11.13
RUN curl -sSL https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip \
        -o /tmp/terraform.zip && \
    unzip /tmp/terraform.zip -d /usr/bin && \
    rm /tmp/terraform.zip

# Latest stable version: curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt
ENV KUBECTL_VERSION v1.16.3
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl \
    && chmod +x ./kubectl \
    && mv ./kubectl /usr/bin/

# Latest stable version: curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4
# ENV KOPS_VERSION 1.15.0
# RUN wget -O kops https://github.com/kubernetes/kops/releases/download/${KOPS_VERSION}/kops-linux-amd64 \
#    && chmod +x ./kops \
#    && mv ./kops /usr/bin/

# Latest stable version: https://github.com/helm/helm/releases
# ENV HELM_VERSION 2.14.3
# RUN curl -sSL https://storage.googleapis.com/kubernetes-helm/helm-v${HELM_VERSION}-linux-amd64.tar.gz |tar xvz && \
#     mv linux-amd64/helm /usr/bin/helm && \
#     chmod +x /usr/bin/helm && rm -rf linux-amd64

WORKDIR /app
CMD ["/bin/bash"]
# ENTRYPOINT ["/bin/sh"]