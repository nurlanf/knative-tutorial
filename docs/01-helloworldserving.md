# Hello World Serving

[Knative Serving](https://github.com/knative/docs/tree/master/serving) has a [samples](https://github.com/knative/docs/tree/master/serving/samples) page with helloworld demos for various languages. You can choose your language of choice. For the purposes of this tutorial, we will go with `helloworld-csharp` sample and assume .NET Core 2.2 runtime.

## Hello World - .NET Core sample

Let's start with deploying an ASP.NET Core service on Knative as described on [Hello World .NET - Core sample](https://github.com/knative/docs/tree/master/serving/samples/helloworld-csharp) page with a few minor changes to the instructions:

1. We use the (currently) latest version of .NET Core runtime: 2.2. In [Dockerfile](../serving/helloworld-csharp/Dockerfile), we changed the first line to reflect this: `FROM microsoft/dotnet:2.2-sdk`

2. When building the Docker image, we tag our image with v1, v2, etc., so we can keep track of different versions of the service: 

3. instead of `service.yaml`, we use [service-v1.yaml](../serving/helloworld-csharp/service-v1.yaml) for the first version. Again, to keep track of different service versions better. 

## Build and push Docker image

With these changes in mind, in [helloworld-csharp](../serving/helloworld-csharp/) folder, build and push the container image. Replace `{username}` with your actual Docker Hub username:

```docker
docker build -t {username}/helloworld-csharp:v1 .

docker push {username}/helloworld-csharp:v1
```
## Deploy the Knative service

Create a [service-v1.yaml](../serving/helloworld-csharp/service-v1.yaml) file.

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: helloworld-csharp
  namespace: default
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            # replace {username} with your DockerHub 
            image: docker.io/{username}/helloworld-csharp:v1
            env:
              - name: TARGET
                value: "C# Sample v1"
```

After the container is pushed, deploy the app. 

```bash
kubectl apply -f service-v1.yaml
```

Check that pods are created and all Knative constructs (service, configuration, revision, route) have been deployed:

```bash
kubectl get pod,ksvc,configuration,revision,route
NAME                                                      READY     STATUS    RESTARTS   
pod/helloworld-csharp-00001-deployment-7fdb5c5dc9-wf2bp   3/3       Running   0          

NAME                                            
service.serving.knative.dev/helloworld-csharp   

NAME                                                  
configuration.serving.knative.dev/helloworld-csharp   

NAME                                                   
revision.serving.knative.dev/helloworld-csharp-00001   

NAME                                          
route.serving.knative.dev/helloworld-csharp   
```

## Test the service

To test the service, we need to find the IP address of the Knative ingress and the URL of the service.

The IP address of Knative ingress is listed under `EXTERNAL_IP`:

```bash
kubectl get svc knative-ingressgateway -n istio-system
```

The URL of the service follows this format: `{service}.{namespace}.example.com`. In this case, we have `helloworld-csharp.default.example.com`. 

Make a request to your service (replace `IP_ADDRESS` with `EXTERNAL_IP`):

```bash
curl -H "Host: helloworld-csharp.default.example.com" http://{IP_ADDRESS}
Hello World C# v1
```

## What's Next?
[Change Configuration](02-changeconfig.md)