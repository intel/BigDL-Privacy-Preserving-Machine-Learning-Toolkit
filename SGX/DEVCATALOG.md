# **BigDL PPML on SGX**

## Introduction

Learn to use BigDL PPML (BigDL Privacy Preserving Machine Learning) to run end-to-end big data analytics applications with distributed clusters on Intel Software Guard Extensions (SGX).

For more workflow examples and reference implementations, please check [Developer Catalog](TODO).


## Solution Technical Overview

[PPML](https://bigdl.readthedocs.io/en/latest/doc/PPML/Overview/ppml.html) (Privacy Preserving Machine Learning) in [BigDL 2.0](https://github.com/intel-analytics/BigDL) provides a Trusted Cluster Environment for secure Big Data & AI applications, even in an untrusted cloud environment. By combining SGX with several other security technologies (e.g., attestation, key management service, private set intersection, federated learning, and homomorphic encryption), BigDL PPML ensures end-to-end security enabled for the entire distributed workflows (Apache Spark, Apache Flink, XGBoost, TensorFlow, PyTorch, etc.).

For more details, please visit the [BigDL 2.0](https://github.com/intel-analytics/BigDL) GitHub repository.

## Solution Technical Details

PPML ensures security for all dimensions of the data lifecycle: data at rest, data in transit, and data in use. Data being transferred on a network is `in transit`, data in storage is `at rest`, and data being processed is `in use`.

![Data Lifecycle](https://user-images.githubusercontent.com/61072813/177720405-60297d62-d186-4633-8b5f-ff4876cc96d6.png)

PPML allows organizations to explore powerful AI techniques while working to minimize the security risks associated with handling large amounts of sensitive data. PPML protects data at rest, in transit and in use: compute and memory protected by SGX Enclaves, storage (e.g., data and model) protected by encryption, network communication protected by remote attestation and Transport Layer Security (TLS), and optional Federated Learning support.

![BigDL PPML](https://user-images.githubusercontent.com/61072813/177922914-f670111c-e174-40d2-b95a-aafe92485024.png)

With BigDL PPML, you can run trusted Big Data & AI applications. Different bigdl-ppml-gramine images correspond to different functions:
- **Trusted Big Data**: with trusted Big Data analytics, users can run end-to-end data analysis (Spark SQL, Dataframe, MLlib, etc.) and Flink in a secure and trusted environment.
- **Trusted Deep Learning Toolkit**: with Trusted Deep Learning Toolkits, users can run secured end-to-end PyTorch training using either a single machine or cloud-native clusters in a trusted execution environment.
- **Trusted Python Toolkit**: with trusted Python Toolkit, users can run Numpy, Pandas, Flask, and Torchserve in a secure and trusted environment.
- **Trusted DL Serving**: with trusted DL Serving, users can run Torchserve, Tritonserver, and TF-Serving in a secure and trusted environment.
- **Trusted Machin Learning**: with end-to-end trusted training and inference, users can run LightGBM (data parallel, feature parallel, voting parallel, etc.) and Spark MLlib (supervised, unsupervised, recommendation, etc.) ML applications in a distributed and secure way.

## Validated Hardware Details

| Supported Hardware           |
| ---------------------------- |
| Intel® 3th Gen Xeon® Scalable Performance processors or later |

Recommended regular memory size: 512G

Recommended EPC(Enclave Page Cache) memory size: 512G

Recommended Cluster Node Number: 3 or more

## How it Works
![image](https://user-images.githubusercontent.com/61072813/178393982-929548b9-1c4e-4809-a628-10fafad69628.png)

As the above picture shows, there are several steps in BigDL PPML, including the deployment(set up K8S, SGX, etc.), the preparation(the image and the data), the APP building(the code), the job submission and reading of results. In addition, AS(Attestation Servive) is optional and will be introduced below.

## Get Started

### BigDL PPML End-to-End Workflow

In this section, we take the image `bigdl-ppml-trusted-bigdata-gramine` and `MultiPartySparkQueryExample` as an example to go through the entire BigDL PPML end-to-end workflow. MultiPartySparkQueryExample is to decrypt people data encrypted with different encryption methods and filter out people whose age is between 20 and 40.

### Prepare your environment

Prepare your environment first, including K8s cluster setup, K8s-SGX plugin setup, key/password preparation, key management service (KMS) and attestation service (AS) setup, and BigDL PPML client container preparation. **Please follow the detailed steps in** [Prepare Environment](https://github.com/intel-analytics/BigDL/blob/main/ppml/docs/prepare_environment.md).

Next, you are going to build a base image, and a custom image on top of it to avoid leaving secrets (e.g., enclave key) in images/containers. After that, you need to register the `MRENCLAVE` in your customer image to Attestation Service Before running your application, and PPML will verify the runtime MREnclave automatically at the backend. The below chart illustrated the whole workflow:
![PPML Workflow with MREnclave](https://user-images.githubusercontent.com/60865256/197942436-7e40d40a-3759-49b4-aab1-826f09760ab1.png)

Start your application with the following guide step by step:

### Prepare your PPML image for the production environment

To build a secure PPML image for a production environment, BigDL prepared a public base image that does not contain any secrets. You can customize your image on top of this base image.

1. Prepare BigDL Base Image

    Build base image using `base.Dockerfile`.

2. Build a Custom Image

    When the base image is ready, you need to generate your enclave key which will be used when building a custom image. Keep the enclave key safe for future remote attestations.

    Running the following command to generate the enclave key `enclave-key.pem`, which is used to launch and sign SGX Enclave. Then you are ready to build your custom image.

    ```bash
    git clone https://github.com/intel-analytics/BigDL.git
    cd ppml/trusted-bigdata/custom-image
    openssl genrsa -3 -out enclave-key.pem 3072
    ./build-custom-image.sh
    ```

    **Warning:** If you want to skip DCAP (Data Center Attestation Primitives) attestation in runtime containers, you can set `ENABLE_DCAP_ATTESTATION` to *false* in `build-custom-image.sh`, and this will generate a none-attestation image. **But never do this unsafe operation in production!**

    The sensitive enclave key will not be saved in the built image. Two values `mr_enclave` and `mr_signer` are recorded while the Enclave is building. You can find `mr_enclave` and `mr_signer` values in the console log, which are hash values used to register your MREnclave in the following attestation step.

    ````bash
    [INFO] Use the below hash values of mr_enclave and mr_signer to register enclave:
    mr_enclave       : c7a8a42af......
    mr_signer        : 6f0627955......
    ````

    Note: you can also customize the image according to your own needs (e.g. third-parity python libraries or jars).
    
    Then, start a client container:

    ```
    export K8S_MASTER=k8s://$(sudo kubectl cluster-info | grep 'https.*6443' -o -m 1)
    echo The k8s master is $K8S_MASTER .
    export DATA_PATH=/YOUR_DIR/data
    export KEYS_PATH=/YOUR_DIR/keys
    export SECURE_PASSWORD_PATH=/YOUR_DIR/password
    export KUBECONFIG_PATH=/YOUR_DIR/kubeconfig
    export LOCAL_IP=$LOCAL_IP
    export DOCKER_IMAGE=intelanalytics/bigdl-ppml-trusted-bigdata-gramine-reference-16g:2.3.0-SNAPSHOT # or the custom image built by yourself

    sudo docker run -itd \
        --privileged \
        --net=host \
        --name=bigdl-ppml-client-k8s \
        --cpus=10 \
        --oom-kill-disable \
        --device=/dev/sgx/enclave \
        --device=/dev/sgx/provision \
        -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket \
        -v $DATA_PATH:/ppml/trusted-big-data-ml/work/data \
        -v $KEYS_PATH:/ppml/trusted-big-data-ml/work/keys \
        -v $SECURE_PASSWORD_PATH:/ppml/trusted-big-data-ml/work/password \
        -v $KUBECONFIG_PATH:/root/.kube/config \
        -e RUNTIME_SPARK_MASTER=$K8S_MASTER \
        -e RUNTIME_K8S_SPARK_IMAGE=$DOCKER_IMAGE \
        -e RUNTIME_DRIVER_PORT=54321 \
        -e RUNTIME_DRIVER_MEMORY=1g \
        -e LOCAL_IP=$LOCAL_IP \
        $DOCKER_IMAGE bash
    ```


## Deploy Attestation Service

Enter the client container:
```
sudo docker exec -it bigdl-ppml-client-k8s bash
```

If you do not need the attestation, you can disable the attestation service. You should configure `spark-driver-template.yaml` and `spark-executor-template` in the client container.yaml to set `ATTESTATION` value to `false` and skip the rest of the step. By default, the attestation service is disabled.
``` yaml
apiVersion: v1
kind: Pod
spec:
  ...
    env:
      - name: ATTESTATION
        value: false
  ...
```

The bi-attestation guarantees that the MREnclave in runtime containers is a secure one made by you. Its workflow is as below:
![image](https://user-images.githubusercontent.com/60865256/198168194-d62322f8-60a3-43d3-84b3-a76b57a58470.png)


To enable attestation, you should have a running Attestation Service in your environment.

**1. Deploy EHSM KMS & AS**

  KMS (Key Management Service) and AS (Attestation Service) make sure applications of the customer run in the SGX MREnclave signed above by customer-self, rather than a fake one fake by an attacker.

  BigDL PPML uses EHSM as a reference KMS & AS, you can follow the guide [here](https://github.com/intel-analytics/BigDL/tree/main/ppml/services/ehsm/kubernetes#deploy-bigdl-ehsm-kms-on-kubernetes-with-helm-charts) to deploy EHSM in your environment.

**2. Enroll in EHSM**

Execute the following command to enroll yourself in EHSM, The `<kms_ip>` is your configured-ip of EHSM service in the deployment section:

```bash
curl -v -k -G "https://<kms_ip>:9000/ehsm?Action=Enroll"
......
{"code":200,"message":"successful","result":{"apikey":"E8QKpBB******","appid":"8d5dd3b*******"}}
```

You will get an `appid` and `apikey` pair. Please save it for later use.

**3. Attest EHSM Server (optional)**

You can attest EHSM server and verify the service is trusted before running workloads to avoid sending your secrets to a fake service.

To attest EHSM server, start a BigDL container using the custom image built before. **Note**: this is the other container different from the client.

```bash
export KEYS_PATH=YOUR_LOCAL_SPARK_SSL_KEYS_FOLDER_PATH
export LOCAL_IP=YOUR_LOCAL_IP
export CUSTOM_IMAGE=YOUR_CUSTOM_IMAGE_BUILT_BEFORE
export PCCS_URL=YOUR_PCCS_URL # format like https://1.2.3.4:xxxx, obtained from KMS services or a self-deployed one

sudo docker run -itd \
    --privileged \
    --net=host \
    --cpus=5 \
    --oom-kill-disable \
    -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket \
    -v $KEYS_PATH:/ppml/trusted-big-data-ml/work/keys \
    --name=gramine-verify-worker \
    -e LOCAL_IP=$LOCAL_IP \
    -e PCCS_URL=$PCCS_URL \
    $CUSTOM_IMAGE bash
```

Enter the docker container:

```bash
sudo docker exec -it gramine-verify-worker bash
```

Set the variables in `verify-attestation-service.sh` before running it:

  ```
  `ATTESTATION_URL`: URL of attestation service. Should match the format `<ip_address>:<port>`.

  `APP_ID`, `API_KEY`: The appID and apiKey pair generated by your attestation service.

  `ATTESTATION_TYPE`: Type of attestation service. Currently support `EHSMAttestationService`.

  `CHALLENGE`: Challenge to get a quote for attestation service which will be verified by local SGX SDK. Should be a BASE64 string. It can be a casual BASE64 string, for example, it can be generated by the command `echo anystring|base64`.
  ```

In the container, execute `verify-attestation-service.sh` to verify the attestation service quote.

  ```bash
  bash verify-attestation-service.sh
  ```

**4. Register your MREnclave to EHSM**

Register the MREnclave with metadata of your MREnclave (appid, apikey, mr_enclave, mr_signer) obtained in the above steps to EHSM through running a python script:

```bash
# At /ppml/trusted-big-data-ml inside the container now
python register-mrenclave.py --appid <your_appid> \
                            --apikey <your_apikey> \
                            --url https://<kms_ip>:9000 \
                            --mr_enclave <your_mrenclave_hash_value> \
                            --mr_signer <your_mrensigner_hash_value>
```
You will receive a response containing a `policyID` and save it which will be used to attest runtime MREnclave when running distributed Kubernetes application. Remember: if you change the image, you should re-do this step and use the new policyID.

**5. Enable Attestation in configuration**

First, upload `appid`, `apikey`, and `policyID` obtained before to Kubernetes as secrets:

```bash
kubectl create secret generic kms-secret \
                  --from-literal=app_id=YOUR_KMS_APP_ID \
                  --from-literal=api_key=YOUR_KMS_API_KEY \
                  --from-literal=policy_id=YOUR_POLICY_ID
```

Configure `spark-driver-template.yaml` and `spark-executor-template.yaml` to enable Attestation as follows:
``` yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: spark-driver
    securityContext:
      privileged: true
    env:
      - name: ATTESTATION
        value: true
      - name: PCCS_URL
        value: https://your_pccs_ip:your_pccs_port
      - name: ATTESTATION_URL
        value: your_ehsm_ip:your_ehsm_port
      - name: APP_ID
        valueFrom:
          secretKeyRef:
            name: kms-secret
            key: app_id
      - name: API_KEY
        valueFrom:
          secretKeyRef:
            name: kms-secret
            key: app_key
      - name: ATTESTATION_POLICYID
        valueFrom:
          secretKeyRef:
            name: policy-id-secret
            key: policy_id
...
```
You should get `Attestation Success!` in logs after you submit a PPML job if the quote generated with `user_report` is verified successfully by Attestation Service. Or you will get `Attestation Fail! Application killed!` or `JASONObject["result"] is not a JASONObject`and the job will be stopped.

## Run BigDL PPML e2e Application
### Encrypt

Encrypt the input data of your Big Data & AI applications (here we use MultiPartySparkQueryExample) and then upload encrypted data to the Network File System (NFS) server (or any file system such as HDFS that can be accessed by the cluster).

1. Generate the input data `people.csv` for the SimpleQuery application
  you can use [generate_people_csv.py](https://github.com/intel-analytics/BigDL/blob/main/ppml/scripts/generate_people_csv.py). The usage command of the script is `python generate_people.py </save/path/of/people.csv> <num_lines>`. For example:
  ```bash
  python generate_people_csv.py amy.csv 30
  python generate_people_csv.py bob.csv 30
  ```

2. Generate the primary key.
```
docker exec -i bigdl-ppml-client-k8s bash
cd /ppml/bigdl-ppml/src/bigdl/ppml/kms/ehsm/
export APIKEY=your_apikey
export APPID=your_appid
python client.py -api generate_primary_key -ip ehsm_ip -port ehsm_port
```
Do this step twice to get two primary keys to encrypt amy.csv and bob.csv.

3. Encrypt `people.csv`

The encryption application is a BigDL PPML job. You need to choose the deploy mode and the way to submit the job first.

* **There are 4 modes to submit a job**:

    1. **local mode**: run jobs locally without connecting to a cluster. It is the same as using spark-submit to run your application: `$SPARK_HOME/bin/spark-submit --class "SimpleApp" --master local[4] target.jar`, driver and executors are not protected by SGX.
        <p align="left">
          <img src="https://user-images.githubusercontent.com/61072813/174703141-63209559-05e1-4c4d-b096-6b862a9bed8a.png" width='250px' />
        </p>


    2. **local SGX mode**: run jobs locally with SGX guarded. As the picture shows, the client JVM is running in a SGX Enclave so that driver and executors can be protected.
        <p align="left">
          <img src="https://user-images.githubusercontent.com/61072813/174703165-2afc280d-6a3d-431d-9856-dd5b3659214a.png" width='250px' />
        </p>


    3. **client SGX mode**: run jobs in k8s client mode with SGX guarded. As we know, in K8s client mode, the driver is deployed locally as an external client to the cluster. With **client SGX mode**, the executors running in the K8S cluster are protected by SGX, and the driver running in the client is also protected by SGX.
        <p align="left">
          <img src="https://user-images.githubusercontent.com/61072813/174703216-70588315-7479-4b6c-9133-095104efc07d.png" width='500px' />
        </p>


    4. **cluster SGX mode**: run jobs in k8s cluster mode with SGX guarded. As we know, in K8s cluster mode, the driver is deployed on the k8s worker nodes like executors. With **cluster SGX mode**, the driver and executors running in the K8S cluster are protected by SGX.
        <p align="left">
          <img src="https://user-images.githubusercontent.com/61072813/174703234-e45b8fe5-9c61-4d17-93ef-6b0c961a2f95.png" width='500px' />
        </p>


* **There are two options to submit PPML jobs**:
    * use [PPML CLI](https://github.com/intel-analytics/BigDL/blob/main/ppml/docs/submit_job.md#ppml-cli) to submit jobs manually
    * use [helm chart](https://github.com/intel-analytics/BigDL/blob/main/ppml/docs/submit_job.md#ppml-cli#helm-chart) to submit jobs automatically

Here we use **k8s client mode** and **PPML CLI** to run the PPML application.

```bash
export secure_password=`openssl rsautl -inkey /ppml/password/key.txt -decrypt </ppml/password/output.bin`
bash bigdl-ppml-submit.sh \
    --master $RUNTIME_SPARK_MASTER \
    --deploy-mode client \
    --sgx-enabled true \
    --driver-memory 5g \
    --sgx-driver-jvm-memory 10g \
    --driver-cores 4 \
    --executor-memory 5g \
    --sgx-executor-jvm-memory 10g \
    --executor-cores 4 \
    --num-executors 2 \
    --conf spark.cores.max=8 \
    --conf spark.network.timeout=10000000 \
    --conf spark.executor.heartbeatInterval=10000000 \
    --conf spark.kubernetes.container.image=$RUNTIME_K8S_SPARK_IMAGE \
    --conf spark.hadoop.io.compression.codecs="com.intel.analytics.bigdl.ppml.crypto.CryptoCodec" \
    --conf spark.bigdl.primaryKey.amy.kms.type=EHSMKeyManagementService \
    --conf spark.bigdl.primaryKey.amy.material=path_to/your_primary_key \
    --conf spark.bigdl.primaryKey.amy.kms.ip=your_kms_ip \
    --conf spark.bigdl.primaryKey.amy.kms.port=your_kms_port \
    --conf spark.bigdl.primaryKey.amy.kms.appId=your_kms_appId \
    --conf spark.bigdl.primaryKey.amy.kms.apiKey=your_kms_apiKey\
    --verbose \
    --class com.intel.analytics.bigdl.ppml.utils.Encrypt \
    --conf spark.executor.extraClassPath=$BIGDL_HOME/jars/* \
    --conf spark.driver.extraClassPath=$BIGDL_HOME/jars/* \
    --name amy-encrypt \
    local://$BIGDL_HOME/jars/bigdl-ppml-spark_$SPARK_VERSION-$BIGDL_VERSION.jar \
    --inputDataSourcePath file://</save/path/of/people.csv> \
    --outputDataSinkPath file://</output/path/to/save/encrypted/people.csv> \
    --cryptoMode aes/cbc/pkcs5padding \
    --dataSourceType csv
```
`Amy` is free to set, as long as it is consistent in the parameters. Do this step twice to encrypt amy.csv and bob.csv. If the application works successfully, you will see the encrypted files in `outputDataSinkPath`.

### Multi-party Decrypt and Query

Run MultiPartySparkQueryExample
```
export secure_password=`openssl rsautl -inkey /ppml/password/key.txt -decrypt </ppml/password/output.bin`
bash bigdl-ppml-submit.sh \
    --master $RUNTIME_SPARK_MASTER \
    --deploy-mode client \
    --sgx-enabled true \
    --driver-memory 5g \
    --sgx-driver-jvm-memory 10g \
    --driver-cores 4 \
    --executor-memory 5g \
    --sgx-executor-jvm-memory 10g \
    --executor-cores 4 \
    --num-executors 2 \
    --conf spark.cores.max=8 \
    --conf spark.network.timeout=10000000 \
    --conf spark.executor.heartbeatInterval=10000000 \
    --conf spark.kubernetes.container.image=$RUNTIME_K8S_SPARK_IMAGE \
    --conf spark.hadoop.io.compression.codecs="com.intel.analytics.bigdl.ppml.crypto.CryptoCodec" \
    --conf spark.bigdl.primaryKey.AmyPK.kms.type=EHSMKeyManagementService \
    --conf spark.bigdl.primaryKey.AmyPK.material=path_to/amy_primary_key \
    --conf spark.bigdl.primaryKey.AmyPK.kms.ip=your_kms_ip \
    --conf spark.bigdl.primaryKey.AmyPK.kms.port=your_kms_port \
    --conf spark.bigdl.primaryKey.AmyPK.kms.appId=your_kms_appId \
    --conf spark.bigdl.primaryKey.AmyPK.kms.apiKey=your_kms_apiKey\
    --conf spark.bigdl.primaryKey.BobPK.kms.type=EHSMKeyManagementService \
    --conf spark.bigdl.primaryKey.BobPK.material=path_to/bob_primary_key \
    --conf spark.bigdl.primaryKey.BobPK.kms.ip=your_kms_ip \
    --conf spark.bigdl.primaryKey.BobPK.kms.port=your_kms_port \
    --conf spark.bigdl.primaryKey.BobPK.kms.appId=your_kms_appId \
    --conf spark.bigdl.primaryKey.BobPK.kms.apiKey=your_kms_apiKey\
    --verbose \
    --class com.intel.analytics.bigdl.ppml.examples.MultiPartySparkQueryExample \
    --conf spark.executor.extraClassPath=$BIGDL_HOME/jars/* \
    --conf spark.driver.extraClassPath=$BIGDL_HOME/jars/* \
    --name multi-query \
    local://$BIGDL_HOME/jars/bigdl-ppml-spark_$SPARK_VERSION-$BIGDL_VERSION.jar \
    /ppml/data/test/amy_encrypted \
    /ppml/data/test/bob_encrypted
```

The expected result in the driver log will be like this:

```
+-------+---+
|   name|age|
+-------+---+
|   fbrk| 46|
|nsakdcv| 20|
|khmlxxs| 39|
|    klq| 38|
|osdbrsm| 59|
|haeeqmn| 24|
| ontxuj| 35|
|   dxbh| 39|
|    qyw| 58|
|    fjf| 31|
|  teuaw| 48|
|   aydw| 59|
|   wpbs| 50|
|  emmgi| 26|
|    yqp| 57|
| nhkelk| 21|
| eebyfz| 48|
|   rksc| 56|
|irohjgs| 24|
|rfxolgi| 29|
+-------+---+

```

## Learn More

To learn more about BigDL PPML, refer to [user guide](https://bigdl.readthedocs.io/en/latest/doc/PPML/Overview/ppml.html) and [tutorial](https://github.com/intel-analytics/BigDL/blob/main/ppml/README.md) for more details.

To build your own Big Data & AI applications, refer to [develop your own Big Data & AI applications with BigDL PPML](https://github.com/intel-analytics/BigDL/blob/main/ppml/README.md#4-develop-your-own-big-data--ai-applications-with-bigdl-ppml).

As for the example above, the tutorial of image `bigdl-ppml-trusted-bigdata-gramine` is [here](https://github.com/intel-analytics/BigDL/blob/main/ppml/trusted-bigdata/README.md). The code of Encrypt is [here](https://github.com/intel-analytics/BigDL/blob/main/scala/ppml/src/main/scala/com/intel/analytics/bigdl/ppml/utils/Encrypt.scala) and the code of MultiPartySparkQueryExample is [here](https://github.com/intel-analytics/BigDL/blob/main/scala/ppml/src/main/scala/com/intel/analytics/bigdl/ppml/examples/MultiPartySparkQueryExample.scala), they are already built into bigdl-ppml-spark_${SPARK_VERSION}-${BIGDL_VERSION}.jar, and the jar is put into image `bigdl-ppml-trusted-bigdata-gramine`.

## Troubleshooting

1. Is SGX supported on CentOS 6/7?
No. Please upgrade your OS if possible.

2. Do we need an Internet connection for the SGX node?
No. We can use PCCS for registration and certificate download. Only PCCS need an Internet connection.

3. Does PCCS require SGX?
No. PCCS can be installed on any server with an Internet connection.

4. Can we turn off SGX attestation?
Of course. But, turning off attestation will break the integrity provided by SGX. Attestation is turned off to simplify installation for a quick start.

5. Does we need to rewrite my applications?
No. In most cases, you don't have to rewrite your applications

## Support Forum

- [Mail List](mailto:bigdl-user-group+subscribe@googlegroups.com)
- [User Group](https://groups.google.com/forum/#!forum/bigdl-user-group)
- [Github Issues](https://github.com/intel-analytics/BigDL/issues)
---
