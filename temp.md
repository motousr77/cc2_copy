### Test script
~~~bash
aws --profile default configure get aws_access_key_id
aws --profile default configure get aws_secret_access_key

# ^ check AWS credentials
cat ~/.aws/config ~/.aws/credentials

# ^ may be before all steps do aws configure 
aws configure --profile default && cat ~/.aws/*

# ^ switch mode of command ./run and k8s-aws-automation/kubeup !!!
chmod +x run
chmod +x k8s-aws-automation/kubeup

# ^ create cluster 
./run -f values.yaml --action create

# ^ destroy cluster
./run -f values.yaml --action destroy
~~~

### Remark
~~~bash
# ^ install aws-cli version 1
sudo apt install aws-cli
~~~

### Test box
~~~bash

# ^ X
echo AWS_ACCESS_KEY_ID=$(aws --profile default configure get aws_access_key_id)
echo AWS_SECRET_ACCESS_KEY=$(aws --profile default configure get aws_secret_access_key)


export AWS_ACCESS_KEY_ID=AKIA2BHEQHAJOAYGSHIN
export AWS_SECRET_ACCESS_KEY=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
export RELATIVE_SCRIPT_DIR=$(dirname $0)
mkdir -p ${RELATIVE_SCRIPT_DIR}/_clusters
export REPO_ROOT_DIR="$(cd "${RELATIVE_SCRIPT_DIR}/k8s-aws-automation" && pwd)"
export REPO_ROOT_DIR_CLUSTERS="$(cd "${RELATIVE_SCRIPT_DIR}/_clusters" && pwd)"
export CUSTOM_KOPS_TEMPLATES="$(cd "${RELATIVE_SCRIPT_DIR}/custom_kops_template" && pwd)"

export VALUES_FILE=values.yaml

cat $VALUES_FILE | docker run -i --rm -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY --dns=1.1.1.1 -v ${REPO_ROOT_DIR}:/app -v ${REPO_ROOT_DIR_CLUSTERS}:/app/_clusters -v ${CUSTOM_KOPS_TEMPLATES}:/app/custom_kops_template -u $(id -u):$(id -g) --name kubeup codefresh/kubeup /app/kubeup


cat $VALUES_FILE | docker run -i --rm -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY --dns=1.1.1.1 -v ${REPO_ROOT_DIR}:/app -v ${REPO_ROOT_DIR_CLUSTERS}:/app/_clusters -v ${CUSTOM_KOPS_TEMPLATES}:/app/custom_kops_template -u $(id -u):$(id -g) --name kubeup codefresh/kubeup /app/kubeup




yq r xmpl_values.yaml infrastructure.public_subnets
yq r xmpl_values.yaml infrastructure.private_subnets
yq r xmpl_values.yaml infrastructure.nat_gateway_id

PRIVATE_SUBNETS_LIST=$(yq r ${RUNTIME_OUTPUT_DIR}/values.yaml infrastructure.private_subnets)
export NAT_GATEWAY_ID=$(yq r ${RUNTIME_OUTPUT_DIR}/values.yaml infrastructure.nat_gateway_id)
if [[ "${NAT_GATEWAY_ID}" == "null" ]]; then
  unset NAT_GATEWAY_ID
fi

if kops create -f ${K8S_OUTPUT_DIR}/cluster.yaml 2>&1 >/dev/null | grep "error creating cluster: file already exists" ; then
    color yellow
    echo "kops cluster ${CLUSTER_DOMAIN} already exists"
    echo "replacing the needed state in remote S3 store"
    color reset
    kops replace -f ${K8S_OUTPUT_DIR}/cluster.yaml --force
    kops update cluster ${CLUSTER_DOMAIN} --yes
    kops rolling-update cluster ${CLUSTER_DOMAIN} --yes
else 
    kops create secret --name ${CLUSTER_DOMAIN} sshpublickey admin -i /tmp/id_rsa.pub
    kops update cluster ${CLUSTER_DOMAIN} --yes
fi



# ^ edit cluster name
sed 's/FIXME/TEST_AUTO_SCRIPT/g' values.yaml.example > values_step.yaml
sed 's/cf-cd.com/devcodemy.net/g' values_step.yaml > values.yaml
# check some case: sed 's/cf-cd.com/devcodemy.net/g' values.yaml
~~~

### Dockerfile
~~~Dockerfile


FROM alpine:3.9
RUN apk --no-cache add python py-pip py-setuptools ruby ca-certificates \
    curl groff less tar bash jq coreutils bind-tools openssl gettext libintl && \
    pip --no-cache-dir install awscli && \
    gem install rake erubis --no-ri --no-rdoc

# Latest stable version: https://github.com/mikefarah/yq/releases/
ENV YQ_VERSION 2.4.0
RUN curl -LO https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64 \
    && chmod +x ./yq_linux_amd64 \
    && mv ./yq_linux_amd64 /usr/bin/yq

# Latest stable version: https://github.com/mozilla/sops/releases
ENV SOPS_VERSION 3.3.0
RUN wget -O sops https://github.com/mozilla/sops/releases/download/${SOPS_VERSION}/sops-${SOPS_VERSION}.linux \
    && chmod +x ./sops \
    && mv ./sops /usr/bin/sops

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
ENV KOPS_VERSION 1.15.0
RUN wget -O kops https://github.com/kubernetes/kops/releases/download/${KOPS_VERSION}/kops-linux-amd64 \
   && chmod +x ./kops \
   && mv ./kops /usr/bin/

# Latest stable version: https://github.com/helm/helm/releases
ENV HELM_VERSION 2.14.3
RUN curl -sSL https://storage.googleapis.com/kubernetes-helm/helm-v${HELM_VERSION}-linux-amd64.tar.gz |tar xvz && \
    mv linux-amd64/helm /usr/bin/helm && \
    chmod +x /usr/bin/helm && rm -rf linux-amd64

# Latest stable version: https://github.com/codefresh-io/cli/releases
ENV CODEFRESH_CLI_VERSION v0.19.5
RUN curl -sSL https://github.com/codefresh-io/cli/releases/download/${CODEFRESH_CLI_VERSION}/codefresh-${CODEFRESH_CLI_VERSION}-alpine-x64.tar.gz \ 
    |tar xvz && mv ./codefresh /usr/bin/codefresh && \
    rm -rf codefresh-${CODEFRESH_CLI_VERSION}-alpine-x64.tar.gz

# Add the keys to access k8s cluster and set permissions
ARG SSH_PUBLIC_KEY
RUN echo "${SSH_PUBLIC_KEY}" > /tmp/id_rsa.pub

WORKDIR /app
CMD ["/bin/bash"]
~~~