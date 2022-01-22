# 🌟 Namespace, Deployment, ReplicaSet

## Namespace

* 10 farklı ekibin tek bir **file server** kullandığı bir senaryo düşünelim:
  * Bir kişinin yarattığı bir dosyayı başkası overwrite edebilir ya da isim çakışmasına sebep olabilir,
  * Sadece Team 1’in görmesi gereken dosyaları ayırmakta zorlanabilirim, sürekli dosya ayarları yapmam gerekir.
  * Bunun çözümü için, her ekibe özel bir klasör yaratabilir ve permissionlarını ekip üyelerine göre düzenleyebiliriz.
* Yukarıdaki örnekteki **fileserver**’ı **k8s clusterı**, **namespace’leri** ise burada her ekibe açılmış **klasörler** olarak düşünebiliriz.
* **Namespace’lerde birer k8s object’idir. Tanımlarken (özellikle YAML) dosyasında buna göre tanımlama yapılmalıdır.**
* Namespace’lerin birbirinden bağımsız ve benzersiz olması gerekir. Namespace’ler birbiri içerisine yerleştirilemez.
* Her k8s oluşturulduğunda ise 4 default namespace oluşturulur. (_default, kube-node-lease, kube-public, kube-system_)

### Namespace Listeleme

* Varsayılan olarak tüm işlemler ve objectler **default namespace** altında işlenir. `kubectl get pods`yazdığımızda herhangi bir namespace belirtmediğimiz için, `default namespace` altındaki podları getirir.

```shell
kubectl get pods --namespace <namespaceName>
kubectl get pods -n <namespaceName>

# Tüm namespace'lerdeki podları listelemek için:
kubectl get pods --all-namespaces
```

### Namespace Oluşturma

```shell
kubectl create namespace <namespaceName>

kubectl get namespaces
```

#### YAML dosyası kullanarak Namespace oluşturma

```yaml
apiVersion: v1
kind: Namespace # development isminde bir namespace oluşturulur.
metadata:
  name: development # namespace'e isim veriyoruz.
---
apiVersion: v1
kind: Pod
metadata:
  namespace: development # oluşturduğumuz namespace altında podu tanımlıyoruz.
  name: namespacepod
spec:
  containers:
  - name: namespacecontainer
    image: nginx:latest
    ports:
    - containerPort: 80
```

Bir namespace içinde çalışan Pod’u yaratırken ve bu poda bağlanırken; kısacası bu podlar üzerinde herhangi bir işlem yaparken **namespace** belirtilmek zorundadır. Belirtilmezse, k8s ilgili podu **default namespace** altında aramaya başlayacaktır.

### Varsayılan Namespace’i Değiştirmek

```
kubectl config set-context --current --namespace=<namespaceName>
```

### Namespace’i Silmek

:warning: **DİKKAT!** **Namespace silerken confirmation istenmeyecektir. Namespace altındaki tüm objectlerde silinecektir!**

```
kubectl delete namespaces <namespaceName>
```

## Deployment

K8s kültüründe “Singleton (Tekil) Pod”lar genellikle yaratılmaz. Bunları yöneten üst seviye object’ler yaratırız ve bu podlar bu objectler tarafından yönetilir. (ÖR: Deployment)

**Peki, neden yaratmıyoruz?**

Örneğin, bir frontend object’ini bir pod içerisindeki container’la deploy ettiğimizi düşünelim. Eğer bu container’da hata meydana gelirse ve bizim RestartPolicy’miz “Always veya On-failure” ise kube-sched containerı yeniden başlatarak kurtarır ve çalışmasına devam ettirir. Fakat, **sorun node üzerinde çıkarsa, kube-sched, “Ben bunu gidip başka bir worker-node’da çalıştırayım” demez!**

Peki buna çözüm olarak 3 node tanımladık, önlerine de bir load balancer koyduk. Eğer birine bir şey olursa diğerleri online olmaya devam edeceği için sorunu çözmüş olduk. **AMA..** Uygulamayı geliştirdiğimizi düşünelim. Tek tek tüm nodelardaki container image’larını yenilemek zorunda kalacağız. Label eklemek istesek, hepsine eklememiz gerekir. **Yani, işler karmaşıklaştı.**

**ÇÖZÜM: “Deployment” Object**

* Deployment, bir veya birden fazla pod’u için bizim belirlediğimiz **desired state**’i sürekli **current state**‘e getirmeye çalışan bir object tipidir. Deployment’lar içerisindeki **deployment-controller** ile current state’i desired state’e getirmek için gerekli aksiyonları alır.
* Deployment object’i ile örneğin yukarıdaki image update etme işlemini tüm nodelarda kolaylıkla yapabiliriz.
* Deployment’a işlemler sırasında nasıl davranması gerektiğini de (**Rollout**) parametre ile belirtebiliriz.
* **Deployment’ta yapılan yeni işlemlerde hata alırsak, bunu eski haline tek bir komutla döndürebiliriz.**
* :warning: :warning: :warning: Örneğin, deployment oluştururken **replica** tanımı yaparsak, k8s cluster’ı her zaman o kadar replika’yı canlı tutmaya çalışacaktır. Siz manuel olarak deployment’ın oluşturduğu pod’lardan birini silseniz de, arka tarafta yeni bir pod ayağa kaldırılacaktır. İşte bu sebeple biz **Singleton Pod** yaratmıyoruz. Yani, manuel ya da yaml ile direkt pod yaratmıyoruz ki bu optimizasyonu k8s’e bırakıyoruz.
* Tek bir pod yaratacak bile olsanız, bunu deployment ile yaratmalısınız! (k8s resmi önerisi)

### Komut ile Deployment Oluşturma

```shell
kubectl create deployment <deploymentName> --image=<imageName> --replicas=<replicasNumber>

kubectl create deployment <deploymentName> --image=nginx:latest --replicas=2

kubectl get deployment
# Tüm deployment ready kolonuna dikkat!
```

### Deployment’taki image’ı Update etme

```shell
kubectl set image deployment/<deploymentName> <containerName>=<yeniImage>

kubectl set image deployment/firstdeployment nginx=httpd
```

* Default strateji olarak, önce bir pod’u yeniler, sonra diğerini, sonra diğerini. Bunu değiştirebiliriz.

### Deployment Replicas’ını Değiştirme

```shell
kubectl scale deployment <deploymentName> --replicas=<yeniReplicaSayısı>
```

### Deployment Silme

```shell
kubectl delete deployments <deploymentName>
```

### **YAML ile Deployment Oluşturma**

1. Herhangi bir pod oluşturacak yaml dosyasındaki **`metadata`** altında kalan komutları kopyala:

```yaml
# podexample.yaml

apiVersion: v1
kind: Pod
metadata:
  name: examplepod
  labels:
    app: frontend
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

1. Deployment oluşturacak yaml dosyasında **`template`** kısmının altına yapıştır. _(Indent’lere dikkat!)_
2. **pod template içerisinden `name` alanını sil.**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: firstdeployment
  labels:
    team: development
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend # template içerisindeki pod'la eşleşmesi için kullanılacak label.
  template:	    # Oluşturulacak podların özelliklerini belirttiğimiz alan.
    metadata:
      labels:
        app: frontend # deployment ile eşleşen pod'un label'i.
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80 # dışarı açılacak port.
```

* Her deployment’ta **en az bir tane** `selector` tanımı olmalıdır.
* **Birden fazla deployment yaratacaksanız, farklı label’lar kullanmak zorundasınız.** Yoksa deploymentlar hangi podların kendine ait olduğunu karıştırabilir. **Ayrıca, aynı labelları kullanan singleton bir pod’da yaratmak sakıncalıdır!**

## ReplicaSet

K8s’de x sayıda pod oluşturan ve bunları yöneten object türü aslında **deployment değildir.** **ReplicaSet**, tüm bu işleri üstlenir. Biz deployment’a istediğimiz derived state’i söylediğimizde, deployments object’i bir ReplicaSet object’i oluşturur ve tüm bu görevleri ReplicaSet gerçekleştirir.

K8s ilk çıktığında **Replication-controller** adında bir object’imiz vardı. Halen var ama kullanılmıyor.

```shell
kubectl get replicaset # Aktif ReplicaSet'leri listeler.
```

Bir deployment tanımlıyken, üzerinde bir değişiklik yaptığımızda; deployment **yeni bir ReplicaSet** oluşturur ve bu ReplicaSet yeni podları oluşturmaya başlar. Bir yandan da eski podlar silinir.

### Deployment üzerinde yapılan değişiklikleri geri alma

```shell
kubectl rollout undo deployment <deploymentName>
```

Bu durumda ise eski deployment yeniden oluşturulur ve eski ReplicaSet önceki podları oluşturmaya başlar. İşte bu sebeple, tüm bu işlemleri **manuel yönetmemek adına** bizler direkt ReplicaSet oluşturmaz, işlemlerimize deployment oluşturarak devam ederiz.

—> **Deployment > ReplicaSet > Pods**

* ReplicaSet, YAML olarak oluşturulmak istendiğinde **tamamen deployment ile aynı şekilde oluşturulur.**
