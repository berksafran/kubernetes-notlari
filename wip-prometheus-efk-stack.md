---
description: Prometheus Stack & ElasticSearch Fluentd Kibana Stack
---

# \[WIP] Prometheus, EFK Stack

![](<.gitbook/assets/image (24) (1).png>)

## Prometheus Stack

Kubernetes’te izlememiz gereken 4 farklı yer var:

1. K8s Cluster’ı nasıl çalışıyor? Hangi object’ler var?
2. Object’lerin mevcut durumu nedir? Ör: Deployment’ta yazdığımız replica gerçekten oluşturulmuş mu?
3. Node’ları izlememiz gerekiyor. Biz container’ları worker node’lar üzerinde çalıştırıyoruz. Bu node’ların CPU ve memory kullanımı, trafikleri nasıl?
4. Containerlar içerisinde loglar oluşuyor. Bu logları nasıl okuyacağız?

### Herhangi bir monitoring tool kullanmadan;

–> Mevcut Pod’ların durumunu, `kubectl get pods` komutu ile öğrenebilirim.

–> **`kubectl get all -A`** ile sistemdeki tüm objectlerin durumunu öğrenebilirim.

–> Spesifik bir object için (Ör: bir pod) `kubectl describe podName` ile öğrenebilirim.

–> Bir cluster içerisinde başlangıçtan itibaren hangi event’lar olduğunu **`kubectl get events -A`** ile öğrenebilirim. (`-A`bize tüm namespace’leri getirir.)

–> `kubectl top node` ile CPU ve Memory kullanımlarını görebilirim. (Pod dersem pod’unkileri görebiliriz.)

–> `kubectl logs <podName>` ile loglara ulaşabilirim.

Her ne kadar bu tür komutlarla manuel olarak istediğimizi elde etsekte, bunu merkezi bir yerden yönetmek daha kolay ve mantıklıdır. Bir hata çıktığında, Alert mekanizması devreye girmeli ve bana email göndermeli. Tüm bunları Prometheus ile yönetebiliriz. (İlk 3’ünü)

### Prometheus Nedir?

–> Bir metrics sunucusudur. Neredeyse tüm Dünya’da kullanılır. CNCF projesi (k8s gibi).

–> Pull based çalışması: Siz kurulumu yapıyorsunuz, Prometheus gerekli metricleri kendi toplamaktadır.

–> Sadece k8s ile çalışmaz, örneğin Frontend’de de kullanabilirsiniz.

!\[image-20211225140914404]\(/Users/bsafranbolulu/Library/Application Support/typora-user-images/image-20211225140914404.png)

* **Kubernetes Metrics**, k8s API ile konuşarak object’lerin current status’lerini çeker ve Prometheus’a iletir.
* **Node Exporter**, Node’ların durumunu çeker ve Prometheus’a iletir.
* **Kubernetes API** ile Prometheus direkt konuşabilir. Cluster’ın durumunu öğrenebilir.
* Prometheus içerisinde Query çalıştırabiliyoruz. Ama bu bilgileri görselleştirmem gerekiyor, dashboardlar oluşturmam gerekiyor. Bunu **Grafana** ile sağlayabiliyoruz.
* Prometheus ve diğer tool’ların kurulumu ve entegrasyonu normalde çok zahmetli ve kompleks. Bu sebeple, prometheus-community **helm-charts** adı altında bu durumu kolaylaştıracak bir stack oluşturdu. Stack kurulumuna geçelim..

### Kurulum

* **monitoring** adında bir namespace oluşturalım:

```shell
kubectl create namespace monitoring
```

* **kubectl-prometheus-stack** kuralım:

```shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kubeprostack --namespace monitoring prometheus-community/kube-prometheus-stack
```

* Kurulduğundan emin olalım:

```shell
kubectl get pods -n monitoring

# Output:
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-kubeprostack-kube-promethe-alertmanager-0   2/2     Running   0          15m
kubeprostack-grafana-5c5d98864b-dn2jz                    3/3     Running   0          15m
kubeprostack-kube-promethe-operator-5fcb5784fc-57pvs     1/1     Running   0          15m
kubeprostack-kube-state-metrics-5765b49669-6qqgl         1/1     Running   0          15m
kubeprostack-prometheus-node-exporter-82mcn              1/1     Running   0          15m
prometheus-kubeprostack-kube-promethe-prometheus-0       2/2     Running   0          15m
```

* Kurulumda grafana vb. bir çok pod expose edilmiş. (Dış dünyaya açılmış) Fakat, prometeus edilmemiş. Bunun için **port-forwarding** yapmamız gerekiyor. Böylece, web arayüzünden prometheus’a bağlanabileceğiz.

```shell
kubectl --namespace monitoring port-forward svc/kubeprostack-kube-promethe-prometheus 9090

# sonrasında http://localhost:9090 'a gidebiliriz.
```

* `http://localhost:9090` Prometheus UI ekranına gidelim ve bir kaç sorgu çalıştıralım:

```shell
kube_pod_created # Şuana kadar oluşturulmuş tüm podları gösterir.

count by (namespace) (kube_pod_created) # Namespace'e göre dağılım 

sum by (namespace) (kube_pod_info) # Mevcut çalışan podlar

sum by (namespace) (kube_pod_status_ready{condition="false"}) # Ready durumunda olmayan podlar "namespace'e göre dağılım"
```

* **Grafana**‘yı kontrol edelim:

```shell
kubectl --namespace monitoring port-forward svc/kubeprostack-grafana 8080:80

# user: admin
# pw: prom-operator

# Grafana password'u alabilmek için:
kubectl get secret kubeprostack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

* **Alert Manager**‘ı kontrol edelim:

```shell
kubectl --namespace monitoring port-forward svc/kubeprostack-kube-promethe-alertmanager 9093
```

## EFK Stack (ElasticSearch Fluentd Kibana)

**Logging** için kullanılan stacktir.

* ElasticSearch, logların toplandığı ve kaydedildiği yerdir.
* **Kibana**, ElasticSearch’ten alınan verileri görselleştirmeye yarar. (_Bir nevi Grafana gibi._)
* **LogStash** (_ElasticSearch ürünüdür_) ve **Fluentd** (_CNCF projesidir_), logları toplamakla görevli tooldur.
* _FluentD daha performanslıdır LogStash’e göre._

### Kurulum (minikube üzerinde)

Buradaki kurulum **minikube** baz alınarak yapılmış olup, bazı ayarları değiştirmek için yaml dosyaları hazırlanmıştır. Cloud üzerine kurulum **helm** ile kolayca yapılsa da helm’i minikube kurulumunda kullandığımızda bazı hatalarla karşılaşmaktayız.

* minikube’u ayağa kaldıralım ve ilgili storage addonları aktif hale getirelim:

```shell
minikube start --cpus 4 --memory 6144

minikube addons enable default-storageclass
minikube addons enable storage-provisioner
```

* Sürekli **log oluşturacak** bir pod yaratalım: (`testpod.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: loggenerator
spec:
  containers:
  - name: loggenerator
    image: busybox
    args: [/bin/sh, -c,'i=0; while true; do echo "Kubernetes Fundamentals $i"; i=$((i+1)); sleep 1; done']
```

* **efk** isimli yeni bir namespace oluşturalım:

```shell
$ kubectl create namespace efk
```

* ElasticSearch cluster’ı oluşturalım:
  * elastic.yaml için [click –>](https://github.com/berksafran/k8sfundamentals/blob/main/monitoring/elastic.yaml)

```shell
kubectl apply -f elastic.yaml
```

***

***

> _\[WIP] Tutorial en kısa zamanda tamamlanacaktır._
