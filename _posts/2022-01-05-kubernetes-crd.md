---
layout: post
title: 操作k8s的CRD
---

CRD的全称为`CustomResourceDefinitions`，即自定义资源。k8s拥有一些内置的资源，比如说Pod，Deployment，ReplicaSet等等，而CRD则提供了一种方式，使用户可以自定义新的资源，以扩展k8s的功能。

<!--more-->

**以下基于 1.23.1 版本的kubernetes**

查看当前的CRD定义指令:
```bash
# kubectl api-resources
```

## CRD的定义
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: users.k8s.ir0.cn
spec:
  group: k8s.ir0.cn
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        description: User is the Schema for the users API
        type: object
        properties:
          spec:
            description: UserSpec
            type: object
            properties:
              name:
                description: User LoginName
                type: string
              email:
                description: User EMail
                type: string
              phone:
                description: User Phone
                type: string
          status:
            description: UserStatus
            type: object
            properties:
              lastLoginTime:
                type: integer

  names:
    kind: User
    listKind: UserList
    plural: users
    singular: user
  scope: Namespaced
```

## 加载CRD


```bash
# kubectl create -f crd-user.yaml
customresourcedefinition.apiextensions.k8s.io/users.k8s.ir0.cn created

# kubectl api-resources | grep ir0
users      k8s.ir0.cn/v1     true     User
```

## 创建对象


```yaml
apiVersion: "k8s.ir0.cn/v1"
kind: User
metadata:
  name: user-hello
spec:
  name: "admin"
  phone: "13344445555"
  email: "admin@ir0.cn"
```

```bash
# kubectl create -f user-hello.yaml
# kubectl describe user user-hello
Name:         user-hello
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  k8s.ir0.cn/v1
Kind:         User
Metadata:
  Creation Timestamp:  2022-01-05T07:48:53Z
  Generation:          1
  Managed Fields:
    API Version:  k8s.ir0.cn/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:email:
        f:name:
        f:phone:
    Manager:         kubectl-create
    Operation:       Update
    Time:            2022-01-05T07:48:53Z
  Resource Version:  789
  UID:               2c8d7306-ac6f-469f-b619-4999f80f832b
Spec:
  Email:  admin@ir0.cn
  Name:   admin
  Phone:  13344445555
Events:   <none>
```

