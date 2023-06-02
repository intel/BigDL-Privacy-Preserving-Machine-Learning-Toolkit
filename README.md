# **BigDL PPML**

## Overview

Learn to use BigDL PPML (BigDL Privacy Preserving Machine Learning) to run end-to-end big data analytics applications with distributed clusters on Intel® **Software Guard Extensions (SGX)** and Intel® **Trust Domain Extensions (TDX)**.

## Intruduction

[PPML](https://bigdl.readthedocs.io/en/latest/doc/PPML/Overview/ppml.html) (Privacy Preserving Machine Learning) in [BigDL 2.0](https://github.com/intel-analytics/BigDL) provides a Trusted Cluster Environment for secure Big Data & AI applications, even in an untrusted cloud environment. By combining SGX with several other security technologies (e.g., attestation, key management service, private set intersection, federated learning, and homomorphic encryption), BigDL PPML ensures end-to-end security enabled for the entire distributed workflows (Apache Spark, Apache Flink, XGBoost, TensorFlow, PyTorch, etc.).

For more details, please visit the [BigDL 2.0](https://github.com/intel-analytics/BigDL) GitHub repository.

PPML ensures security for all dimensions of the data lifecycle: data at rest, data in transit, and data in use. Data being transferred on a network is `in transit`, data in storage is `at rest`, and data being processed is `in use`.

![Data Lifecycle](https://user-images.githubusercontent.com/61072813/177720405-60297d62-d186-4633-8b5f-ff4876cc96d6.png)

PPML allows organizations to explore powerful AI techniques while working to minimize the security risks associated with handling large amounts of sensitive data. PPML protects data at rest, in transit, and in use: compute and memory protected by SGX Enclaves, storage (e.g., data and model) protected by encryption, network communication protected by remote attestation and Transport Layer Security (TLS), and optional Federated Learning support.

![BigDL PPML](https://user-images.githubusercontent.com/61072813/177922914-f670111c-e174-40d2-b95a-aafe92485024.png)

With BigDL PPML, you can run trusted Big Data & AI applications. Different bigdl-ppml-gramine images correspond to different functions:
- **Trusted Big Data**: with trusted Big Data analytics, users can run end-to-end data analysis (Spark SQL, Dataframe, MLlib, etc.) and Flink in a secure and trusted environment.
- **Trusted Deep Learning Toolkit**: with Trusted Deep Learning Toolkits, users can run secured end-to-end PyTorch training using either a single machine or cloud-native clusters in a trusted execution environment.
- **Trusted Python Toolkit**: with trusted Python Toolkit, users can run Numpy, Pandas, Flask, and Torchserve in a secure and trusted environment.
- **Trusted DL Serving**: with trusted DL Serving, users can run Torchserve, Tritonserver, and TF-Serving in a secure and trusted environment.
- **Trusted Machin Learning**: with end-to-end trusted training and inference, users can run LightGBM (data parallel, feature parallel, voting parallel, etc.) and Spark MLlib (supervised, unsupervised, recommendation, etc.) ML applications in a distributed and secure way.