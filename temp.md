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
chmod +x k8s-aws-automation/destroy

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

export AWS_ACCESS_KEY_ID=NEWXXXXXXXXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=newyyyyyyyyyyyyyyyyyyyyyyyxxxxxxxxxx
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
