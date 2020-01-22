#### About this copy 
This repository copied from other my repo named https://github.com/devcodemy/cc2

### Some kind of kubernetes cluster tool
- firts of all i need to choose installation way
- second i need to automate some of them (local[Vagrant], remote[Terraform])
- choose language: bash/pyrhon/ruby/nodejs


### Build docker image
~~~bash
docker build -t devcodemy/img_cc2:v1 .

docker images --format='{{json .}}'
docker images --format='{{json .}}' | jq '.Repository'
docker container ls --format='{{json .}}'

docker run -it --rm --name cc2 devcodemy/img_cc2:v1

# ^ print all untaged images
docker images -f "dangling=true" -q

# ^ remove untagged images
docker rmi -f $(docker images -f "dangling=true" -q)
~~~

### Variables
~~~sh
export S3_BUCKET_NAME=test-bucket
export AWS_REGION=us-east-2 # USA - Ohio
export SUBNET_ID=subnet-a05bedee
export ALLOCATION_ID=...
~~~

### AWS
~~~bash
# ^ create s3 bucket
aws s3 mb s3://$S3_BUCKET_NAME --region $AWS_REGION

~~~


### TMP
~~~Dockerfile
RUN apk add --no-cache --virtual .build-deps <dev packages>
 && apk add --no-cache --update python3
 && pip3 install --upgrade pip setuptools

RUN pip3 install -f ./python-packages --no-index -r requirements.txt ./python-packages/pkgs


b2711f1189c0 3c162ba0bc28 8e37dec0d80b ea2cb58aa4cb 68d60081f72b 51904b30ea38



~~~