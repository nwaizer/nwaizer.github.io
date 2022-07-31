---
layout: post
title: How to deploy in golang
tags: [go, golang, kubernetes]
---

kubernetes offers a go wrapper to its API to allow you to do create update delete deployment, In fact look at [this](https://github.com/kubernetes/client-go/blob/ee1a5aaf793a9ace9c433f5fb26a19058ed5f37c/examples/create-update-delete-deployment/main.go)

If you want to create a single pod in your testing, you can use Niranjan example:
## Create a Pod using Pod API.

```
* This is an example program to create pods using corev1.Pod */

package main

import (
        "fmt"
        "context"
        testclient "github.com/openshift-kni/performance-addon-operators/functests/utils/client"
        v1 "k8s.io/api/core/v1"
        metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

//main function
func main() {

        // create a varible of type v1.Pod 
        var testpod *v1.Pod
        var err error 
        //Define Pod
        testpod = &v1.Pod{
                //Define Metadata
                ObjectMeta : metav1.ObjectMeta{
                        Name: "test-pod", //Pod name: Explicity specified. 
                        Namespace: "default", // The name space where pod should be created
                        Lables: map[string]string{
                                "name": "test-pod",
                        },
                },
                //Pod specification 
                Spec: v1.PodSpec{
                        // Containers to run in Pod
                        Containers: []v1.Container{
                                {
                                        Name: "test-fedora-container", // container name
                                        Image: "fedora:latest",  //container image fedora:latest
                                        Command: []string{"sleep", "inf"}, // Command that runs which is sleep
                                },
                        },
                },
        }
        err = testclient.Client.Create(context.TODO(), testpod)
        if err != nil {
                fmt.Println("Pod did not get created")
        }
}
```

But in my case. I need to define not only the pod, but also the service, and role.
So I used [daemonset](https://pkg.go.dev/k8s.io/kubernetes/pkg/registry/apps/daemonset)

The idea is to create a directory with many yaml files, each describing a single kubernetes kind to create.
For example: 01_sidecar-sa.yaml , 02_sidecar-role.yaml ...

Then use this code to apply them:

