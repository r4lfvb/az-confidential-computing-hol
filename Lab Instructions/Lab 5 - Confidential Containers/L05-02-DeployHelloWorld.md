L05-02

# Deploy a test application

Review the *hello-world-enclave.yaml* file in the *.\Code\Lab 4 - Confidential Containers* folder. This is the manifest for a test application that you can find at [Open Enclave project](https://github.com/openenclave/openenclave/tree/master/samples/helloworld). 
This manifest pulls a public container image for the test application from Docker Hub.

Now let's create a job to apply the manifest:

```powershell
kubectl apply -f ".\Code\Lab 5 - Confidential Containers\hello-world-enclave.yaml"
```

You can confirm that the workload successfully created a Trusted Execution Environment (enclave) by running the following commands:

```powershell
kubectl get jobs -l app=sgx-test
```

This should return something like:
```
NAME       COMPLETIONS   DURATION   AGE
sgx-test   1/1           1s         23s
```

```powershell
kubectl get pods -l app=sgx-test
```

This should return something like:
```
NAME                READY   STATUS      RESTARTS   AGE
sgx-test--1-64xrl   0/1     Completed   0          30s
```

```powershell
kubectl logs -l app=sgx-test
```

This should show the following log entry if the test application executed successfully:
```
Hello world from the enclave
Enclave called into host to print: Hello World!
```