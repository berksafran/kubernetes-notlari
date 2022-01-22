# 🎺 Ingress

## Ingress

* Uygulamalarımızın dış dünyaya erişebilmesi/dış dünyadan erişilebilmesi için kullandığımız yapıdır.

**Örnek Senaryo**

![](<.gitbook/assets/Screen Shot 2022-01-03 at 10.58.07.png>)

Azure gibi bir cloud service kullandığımızı varsayalım. Servisin içerisine bir LoadBalancer service’ı tanımlayalım. Azure, bizim adımıza bu LoadBalancer servisine bir IP atıyor ve bu IP’ye gelen tüm istekler bu LoadBalancer tarafından karşılanıyor. Biz de bu IP adresi ile DNS sayesinde domainimizi eşleştirerek, kullanıcıların kolay bir şekilde erişmesini sağlayalım.

Aynı k8s cluster içerisinde bir tane daha app ve aynı servisleri tanımladığımızı düşünelim. Hatta abartalım 2, 3, 4 derken her bir LoadBalancer için **Azure’a ekstradan para ödemem ve ayarlarını manuel yapmam gerekiyor.**

**Örnek Senaryo - 2**

![](<.gitbook/assets/Screen Shot 2022-01-03 at 10.58.24.png>)

Bu örnekte ise; kullanıcı **example.com**‘a girdiğinde A uygulaması; **example.com/contact**’a girdiğinde ise B uygulaması çalışsın. Bu durumu, DNS’te **/contact** path’i tanımlayamadığımız için LoadBalancer ile kurgulama şansımız yoktur. Fakat, bizim bir gateway gibi çalışan; kullanıcıyı her halükarda karşılayan bir load balancer’a ihtiyacım var.

İşte bu 2 örnekte/sorunu da **Ingress Controller ve Ingress Object** ile çözüyoruz:

## Ingress Controller ve Ingress Object

![](<.gitbook/assets/Screen Shot 2022-01-03 at 11.02.20.png>)

* **Ingress Controller**, Nginx, Traefik, KrakenD gibi kullanabileceğimiz bir load balancer uygulamasına denir. Bu uygulamalardan birini seçip; k8s cluster’ımıza deploy edebilir ve LoadBalancer servisini kurarak dışarıya expose edebiliriz. Böylelikle, uygulamamız **public bir IP**’e sahip oluyor ve userlar ile tamamen bu IP üzerinden iletişim kurabiliriz.
* **Peki, gelen istekleri nasıl yönlendiriyoruz?** İşte bu esnada **Ingress Object**‘leri devreye giriyor. (_YAML dosyalarında tanımlanan yapılar_) Ingress Controller’larda yapacağımız konfigürasyonlarla Ingress Object’lerimizi ve Ingress Controller’ların gelen requestlere karşı nasıl davranması gerektiğini belirleyebiliriz.
* **Load balancing, SSL termination ve path-name based routing** gibi özelliklere sahiptir.

## Local’de Ingress Uygulama

### 1) minikube’ü Ayarlama

* Ingress’i çalıştırmak için minikube driver’ını değiştirmemiz gerekmektedir;
  * Windows için **Hyper-V**, macOS ve linux için **VirtualBox** seçebiliriz. Seçmeden önce kurulum yapmayı unutmayalım.

```shell
minikube start --driver=hyperv
```

### 2) Ingress Controller Seçimi ve Kurulumu

*   Biz nginx ile devam edeceğiz. Her ingress controller’ın kurulumu farklıdır. Kurulum detaylarını uygulamanın kendi web sitesinden öğrenebilirsiniz.

    **Kurulum detayları –>** https://kubernetes.github.io/ingress-nginx/deploy/
* minikube, yoğun olarak kullanılan nginx gibi bazı ingress controller’ı daha hızlı aktif edebilmek adına addon olarak sunmaktadır.

```shell
minikube addons enable ingress # ingress addonunu aktif eder.
minikube addons list # tüm addon'ları listeler.
```

* :point\_right: **Nginx** kurulduğu zaman kendisine **ingress-nginx** adında bir **namespace yaratır.**

```shell
# ingress-nginx namespace'ine ait tüm objectlerini listelemek için:
kubectl get all -n ingress-nginx 
```

### 3) Ingress Uygulamalarımızı Deploy Etmek

* **blueapp, greenapp, todoapp** için hem podlarımızı hem de servicelerimizi yaratan yaml dosyamızı deploy edelim.
* **Tüm service’lerin ClusterIP tipinde birer service olduğunu unutmayalım.**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blueapp
  labels:
    app: blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: blue
  template:
    metadata:
      labels:
        app: blue
    spec:
      containers:
      - name: blueapp
        image: ozgurozturknet/k8s:blue
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /healthcheck
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 3
---
apiVersion: v1
kind: Service
metadata:
  name: bluesvc
spec:
  selector:
    app: blue
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: greenapp
  labels:
    app: green
spec:
  replicas: 2
  selector:
    matchLabels:
      app: green
  template:
    metadata:
      labels:
        app: green
    spec:
      containers:
      - name: greenapp
        image: ozgurozturknet/k8s:green
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /healthcheck
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 3
---
apiVersion: v1
kind: Service
metadata:
  name: greensvc
spec:
  selector:
    app: green
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todoapp
  labels:
    app: todo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: todo
  template:
    metadata:
      labels:
        app: todo
    spec:
      containers:
      - name: todoapp
        image: ozgurozturknet/samplewebapp:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: todosvc
spec:
  selector:
    app: todo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 4) Ingress Object’lerini Deploy Etme ve Ayarlama

* Load balancer için gerekli olan Ingress Controller’ımızı Nginx olarak seçtik ve kurduk.
* Her bir app için gerekli olan ClusterIP tipinde servislerimizi de kurduktan sonra, sıra kullanıcıların **example.com/a** yazdığında A service’ine gitmesi için gerekli **Ingress object’lerimizi** de deploy etmeye geldi.

> _**Araştırma Konusu:** –> Layer 7 nedir? Ne işe yarar?_

**blue, green app’ler için Ingress Object tanımlaması:**

* `pathType` kısmı `exact`veya `Prefix` olarak 2 şekilde ayarlanabilir. Detaylı bilgi için: https://kubernetes.io/docs/concepts/services-networking/ingress/

```shell
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: appingress
  annotations:
  # Nginx üzerinde ayarlar, annotations üzerinden yapılır.
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: k8sfundamentals.com
      http:
        paths:
          - path: /blue
            pathType: Prefix 
            backend:
              service:
                name: bluesvc
                port:
                  number: 80
          - path: /green
            pathType: Prefix
            backend:
              service:
                name: greensvc
                port:
                  number: 80
```

* Farklı bir `path` kullanarak hazırlanan Ingress Objecti:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: todoingress
spec:
  rules:
    - host: todoapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: todosvc
                port:
                  number: 80
```

### 5) Tanımlanan Ingress Object’leri test etme:

```yaml
kubectl get ingress
```

* Eğer URL’ler ile simüle etmek istersek, **hosts** dosyasını editlememiz gerekir.
