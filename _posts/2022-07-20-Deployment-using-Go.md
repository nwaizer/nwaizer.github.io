---
layout: post
title: How to deploy in golang
tags: [go, golang, kubernetes]
---

kubernetes offers a go wrapper to its API to allow you to do create update delete deployment, In fact look at [this](https://github.com/kubernetes/client-go/blob/ee1a5aaf793a9ace9c433f5fb26a19058ed5f37c/examples/create-update-delete-deployment/main.go)

If you want to create a single pod in your testing, you can use Niranjan example:
## Create a Pod using Pod API.

```go
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

Then use this [code](../assets/posts/Deployment-using-Go/deployIt.go) to apply them:
```go
package Deployment_using_Go

import (
	"context"
	"fmt"
	"log"
	"time"

	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/client/config"

	"k8s.io/apimachinery/pkg/util/wait"

	"github.com/k8snetworkplumbingwg/sriov-network-operator/pkg/render"
	"k8s.io/apimachinery/pkg/apis/meta/v1/unstructured"
	"k8s.io/apimachinery/pkg/types"

	appsv1 "k8s.io/api/apps/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
)

const (
	createMode = "create"
	deleteMode = "delete"
	updateMode = "update"
)

// DeployProcessExporter2 deploys process exporter and returns the daemonset and error if any.
// clientset is a kubernetes clientset
// configDir a directory where all the deployment yaml exist
// image container image
// namespace to create the pod at
// podName given name

func DeployByManifests(
	clientset *kubernetes.Clientset, configsDir, image, namespace, podName string) (*appsv1.DaemonSet, error) {
	
	cfg, _ := config.GetConfig()
	k8sClient, _ := client.New(cfg, client.Options{})

	daemonset, err := clientset.AppsV1().DaemonSets(namespace).Get(
		context.Background(),
		podName, metav1.GetOptions{},
	)
	if err != nil {
		// adds manifests from given directory to cluster.
		err = modifyObjects(createMode, configsDir, k8sClient)
	} else {
		// updates existing resources based on manifests from given directory.
		err = modifyObjects(updateMode, configsDir, k8sClient)
	}
	if err != nil {
		return nil, err
	}
	wait.Poll(5*time.Second, 5*time.Minute, func() (bool, error) {
		daemonset, err = clientset.AppsV1().DaemonSets(namespace).Get(
			context.Background(),
			podName,
			metav1.GetOptions{},
		)
		if err != nil {
			log.Println("Failed to retrieve status for daemonset: ", daemonset.Name)

			return false, err
		}
		if daemonset.Spec.Template.Spec.Containers[0].Image != image {
			// Update dummy image to configured value
			daemonset.Spec.Template.Spec.Containers[0].Image = image
			_, _ = clientset.AppsV1().DaemonSets(namespace).Update(
				context.Background(),
				daemonset,
				metav1.UpdateOptions{},
			)

			return false, fmt.Errorf("image updated, retrieve updated daemonset in next round")
		}
		if daemonset.Status.NumberReady == daemonset.Status.DesiredNumberScheduled {
			log.Printf("All pods are ready in daemonset: %s\n", daemonset.Name)

			return true, nil
		}

		return false, fmt.Errorf("not all daemon pods are ready in daemonset: %s", daemonset.Name)
	})

	return daemonset, nil
}

func modifyObjects(mode string, resDir string, k8sClient client.Client) error {
	data := render.MakeRenderData()
	objs, err := render.RenderDir(resDir, &data)

	if err != nil {
		return err
	}

	var errorList []error

	for _, obj := range objs {
		switch mode {
		case createMode:
			err = k8sClient.Create(context.TODO(), obj)
		case deleteMode:
			err = k8sClient.Delete(context.TODO(), obj)
		case updateMode:
			err = updateObject(obj, k8sClient)
		}

		if err != nil {
			errorList = append(errorList, err)
		}
	}

	if len(errorList) > 0 {
		return fmt.Errorf(
			"one or more errors occurred while processing resources from dir %s \n%v errors",
			resDir, errorList)
	}

	return nil
}

func updateObject(obj *unstructured.Unstructured, k8sClient client.Client) error {
	if obj.GetName() == "" {
		return fmt.Errorf("object %s has no name", obj.GroupVersionKind().String())
	}

	gvk := obj.GroupVersionKind()
	existing := &unstructured.Unstructured{}
	existing.SetGroupVersionKind(gvk)
	err := k8sClient.Get(
		context.TODO(),
		types.NamespacedName{Name: obj.GetName(), Namespace: obj.GetNamespace()},
		existing)

	if err != nil {
		return err
	}

	obj.SetCreationTimestamp(existing.GetCreationTimestamp())
	obj.SetResourceVersion(existing.GetResourceVersion())
	obj.SetUID(existing.GetUID())
	obj.SetGeneration(existing.GetGeneration())
	obj.SetManagedFields(existing.GetManagedFields())
	obj.SetFinalizers(existing.GetFinalizers())

	return k8sClient.Update(context.TODO(), obj)
}

```