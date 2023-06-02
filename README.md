# **BigDL Privacy-preserving Maching Learning Toolkit**

## Overview

Learn to use BigDL PPML (BigDL Privacy Preserving Machine Learning) to run end-to-end big data analytics applications with distributed clusters on Intel® **Software Guard Extensions (SGX)** and Intel® **Trust Domain Extensions (TDX)**.

## Intruduction

[PPML](https://bigdl.readthedocs.io/en/latest/doc/PPML/Overview/ppml.html) (Privacy Preserving Machine Learning) in [BigDL 2.0](https://github.com/intel-analytics/BigDL) provides a Trusted Cluster Environment for secure Big Data & AI applications, even in an untrusted cloud environment. By combining SGX or TDX with several other security technologies (e.g., attestation, key management service, private set intersection, federated learning, and homomorphic encryption), BigDL PPML ensures end-to-end security enabled for the entire distributed workflows (Apache Spark, Apache Flink, XGBoost, TensorFlow, PyTorch, etc.).

For more details, please visit the [BigDL 2.0](https://github.com/intel-analytics/BigDL) GitHub repository.

PPML ensures security for all dimensions of the data lifecycle: data at rest, data in transit, and data in use. Data being transferred on a network is `in transit`, data in storage is `at rest`, and data being processed is `in use`.

![Data Lifecycle](https://user-images.githubusercontent.com/61072813/177720405-60297d62-d186-4633-8b5f-ff4876cc96d6.png)

To protect data in transit, enterprises often choose to encrypt sensitive data prior to moving or use encrypted connections (HTTPS, SSL, TLS, FTPS, etc) to protect the contents of data in transit. For protecting data at rest, enterprises can simply encrypt sensitive files prior to storing them or choose to encrypt the storage drive itself. However, the third state, data in use has always been a weakly protected target. There are three emerging solutions that seek to reduce the data-in-use attack surface: homomorphic encryption, multi-party computation, and confidential computing.

![BigDL PPML](https://user-images.githubusercontent.com/61072813/177922914-f670111c-e174-40d2-b95a-aafe92485024.png)

Among these security technologies, Confidential computing protects data in use by performing computation in a hardware-based Trusted Execution Environment (TEE).

Intel® SGX is Intel's Trusted Execution Environment (TEE), offering hardware-based memory encryption that isolates specific application code and data in memory. With Intel® SGX, users can use **Trusted BigData** image based on **Base** image to run end-to-end data analysis (Spark SQL, Dataframe, MLlib, etc.) and Flink in a secure and trusted environment. For more information, please visit the [SGX DevCatalog](https://github.com/intel/BigDL-Privacy-Preserving-Machine-Learning-Toolkit/blob/main/SGX/DEVCATALOG.md).

Intel® TDX is the next generation of Intel's Trusted Execution Environment (TEE), introducing new, architectural elements to help deploy hardware-isolated, virtual machines (VMs) called trust domains (TDs). With Intel® TDX, users can use **Trusted Big Data** image to run end to end Big Data analytics solution in trusted environment and **Trusted Deep Learning** image to run trusted distributed deep learning in secured environment. For more information, please visit the [TDX DevCatalog](https://github.com/intel/BigDL-Privacy-Preserving-Machine-Learning-Toolkit/blob/main/TDX/DEVCATALOG.md).