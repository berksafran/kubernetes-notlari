# 🏷 Label, Selector, Annotation

## Label Nedir?

* Label -> Etiket
* Selector -> Etiket Seçme

ÖR: `example.com/tier:front-end` –>`example.com/` = Prefix (optional) `tier` = **key**, `front-end` = **value**

* `kubernetes.io/`ve `k8s.io/` Kubernetes core bileşenler için ayrılmıştır, kullanılamazdır.
* Tire, alt çizgi, noktalar içerebilir.
* Türkçe karakter kullanılamaz.
* **Service, deployment, pods gibi objectler arası bağ kurmak için kullanılır.**

## Label & Selector Uygulama

* Label tanımı **metadata** tarafında yapılır. Aynı object’e birden fazla label ekleyemeyiz.
* Label, gruplandırma ve tanımlama imkanı verir. CLI tarafında listelemekte kolaylaşır.

### Selector - Label’lara göre Object Listelemek

* İçerisinde örneğin “app” key’ine sahip objectleri listelemek için:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod8
  labels:
    app: berk # app key burada. berk ise value'su.
    tier: backend # tier başka bir key, backend value'su.
...
---
```

```shell
kubectl get pods -l <keyword> --show-labels

## Equality based Syntax'i ile listeleme

kubectl get pods -l "app" --show-labels

kubectl get pods -l "app=firstapp" --show-labels

kubectl get pods -l "app=firstapp, tier=front-end" --show-labels

# app key'i firstapp olan, tier'ı front-end olmayanlar:
kubectl get pods -l "app=firstapp, tier!=front-end" --show-labels

# app anahtarı olan ve tier'ı front-end olan objectler:
kubectl get pods -l "app, tier=front-end" --show-labels

## Set based ile Listeleme

# App'i firstapp olan objectler:
kubectl get pods -l "app in (firstapp)" --show-labels

# app'i sorgula ve içerisinde "firstapp" olmayanları getir:
kubectl get pods -l "app, app notin (firstapp)" --show-labels

kubectl get pods -l "app in (firstapp, secondapp)" --show-labels

# app anahtarına sahip olmayanları listele
kubectl get pods -l "!app" --show-labels

# app olarak firstapp atanmış, tier keyine frontend değeri atanmamışları getir:
kubectl get pods -l "app in (firstapp), tier notin (frontend)" --show-labels
```

* İlk syntax’ta (equality based) bir sonuç bulunamazken, 2. syntax (set based selector) sonuç gelir:

```yaml
kubectl get pods -l "app=firstapp, app=secondapp" --show-labels # Sonuç yok!
kubectl get pods -l "app in (firstapp, secondapp)" --show-labels # Sonuç var :)
```

### Komutla label ekleme

```shell
kubectl label pods <podName> <label>

kubectl label pods pod1 app=front-end
```

### Komutla label silme

Sonuna - (tire) koymak gerekiyor. Sil anlamına gelir.

```
kubectl label pods pod1 app-
```

### Komutla label güncelleme

```shell
kubectl label --overwrite pods <podName> <label>

kubectl label --overwrite pods pod9 team=team3
```

### Komutla toplu label ekleme

Tüm objectlere bu label eklenir.

```
kubectl label pods --all foo=bar
```

## Objectler Arasında Label İlişkisi

* NŞA’da kube-sched kendi algoritmasına göre bir node seçimi yapar. Eğer bunu manuel hale getirmek istersek, aşağıdaki örnekte olduğu gibi `hddtype: ssd` label’ına sahip node’u seçmesini sağlayabiliriz. Böylece, pod ile node arasında label’lar aracılığıyla bir ilişki kurmuş oluruz.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod11
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
  nodeSelector:
    hddtype: ssd
```

> _minikube cluster’ı içerisindeki tek node’a `hddtype: ssd` label’ı ekleyebiliriz. Bunu ekledikten sonra yukarıdaki pod “Pending” durumundan, “Running” durumuna geçecektir. (Aradığı node’u buldu çünkü_ :smile: _)_

```shell
kubectl label nodes minikube hddtype=ssd
```

## Annotation

* Aynı label gibi davranır ve **metadata** altına yazılır.
* Label’lar 2 object arasında ilişki kurmak için kullanıldığından hassas bilgi sınıfına girer. Bu sebeple, label olarak kullanamayacağımız ama önemli bilgileri **Annotation** sayesinde kayıt altına alabiliriz.
* **example.com/notification-email:admin@k8s.com**
  * example.com –> Prefix (optional)
  * notification-email –> Key
  * admin@k8s.com –> Value

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotationpod
  annotations:
    owner: "Ozgur OZTURK"
    notification-email: "admin@k8sfundamentals.com"
    releasedate: "01.01.2021"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  containers:
  - name: annotationcontainer
    image: nginx
    ports:
    - containerPort: 80
```

### Komutla Annotation ekleme

```shell
kubectl annotate pods annotationpod foo=bar

kubectl annotate pods annotationpod foo- # Siler.
```
