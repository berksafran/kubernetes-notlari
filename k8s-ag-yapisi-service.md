# ☀ K8s Ağ Yapısı, Service

## K8s Temel Ağ Altyapısı

Aşağıdaki üç kural dahilinde K8s network yapısı ele alınmıştır (_olmazsa olmazdır_):

1. K8s kurulumda pod’lara ip dağıtılması için bir **IP adres aralığı** (`--pod-network-cidr`) belirlenir.
2. K8s’te her pod bu cidr bloğundan atanacak **unique bir IP adresine sahip olur.**
3. **Aynı cluster içerisindeki Pod’lar** default olarak birbirleriyle herhangi **bir kısıtlama olmadan ve NAT (Network Address Translation) olmadan haberleşebilir.**

> _**TODO: Buraya K8s’in Nasıl Network konusunu işlediğini yazılacak.**_

* K8s içerisindeki containerlar 3 tür haberleşmeye maruz bırakılır:
  1. Bir container k8s dışındaki bir IP ile haberleşir,
  2. Bir container kendi node içerisindeki başka bir container’la haberleşir,
  3. Bir container farklı bir node içerisindeki başka bir container’la haberleşir.
* İlk 2 senaryoda sorun yok ama 3. senaryo’da NAT konusunda problem yaşanır. Bu sebeple k8s containerların birbiri ile haberleşme konusunda **Container Network Interface (CNI)** projesini devreye almıştır.
* **CNI, yanlızca containerların ağ bağlantısıyla ve containerlar silindiğinde containerlara ayrılan kaynakların drop edilmesiyle ilgilenir.**
* **K8s ise ağ haberleşme konusunda CNI standartlarını kabul etti ve herkesin kendi seçeceği CNI pluginlerinden birini seçmesine izin verdi.** Aşağıdaki adreslerden en uygun olan CNI pluginini seçebilirsiniz:

{% embed url="https://github.com/containernetworking/cni" %}

{% embed url="https://kubernetes.io/docs/concepts/cluster-administration/networking" %}

**Container’ların “Dış Dünya” ile haberleşmesi konusunu ele aldık. Peki, “Dış Dünya”, Container’lar ile nasıl haberleşecek?**

Cevap: **Service** object’i.

## Service

* K8s network tarafını ele alan objecttir.

### Service Objecti Çıkış Senaryosu

1 Frontend (React), 1 Backend (Go) oluşan bir sistemimiz olduğunu düşünelim:

* Her iki uygulama için **birer deployment** yazdık ve **3’er pod** tanımlanmasını sağladık.
* 3 Frontend poduna dış dünyadan nasıl erişim sağlayacağım?
* Frontend’den gelen istek backend’de işlenmeli. Burada çok bir problem yok. Çünkü, her pod’un bir IP adresi var ve K8s içerisindeki her pod birbiriyle bu IP adresleri sayesinde haberleşebilir.
* Bu haberleşmeyi sağlayabilmek için; Frontend podlarının Backend podlarının IP adreslerini **bilmeleri gerekir.** Bunu çözmek için, frontend deployment’ına tüm backend podlarının IP adreslerini yazdım. Fakat, pod’lar güncellenebilir, değişebilir ve bu güncellemeler esnasında **yeni bir IP adresi alabilir. Yeni oluşan IP adreslerini tekrar Frontend deployment’ında tanımlamam gerekir.**

İşte tüm bu durumları çözmek için **Service** objecti yaratırız. **K8s, Pod’lara kendi IP adreslerini ve bir dizi Pod için tek bir DNS adı verir ve bunlar arasında yük dengeler.**

## Service Tipleri

### **ClusterIP** (Container’lar Arası)

–> Bir ClusterIP service’i yaratıp, bunu label’lar sayesinde podlarla ilişkilendirebiliriz. Bu objecti yarattığımız zaman, Cluster içerisindeki tüm podların çözebileceği unique bir DNS adrese sahip olur. Ayrıca, her k8s kurulumunda sanal bir IP range’e sahip olur. (Ör: 10.0.100.0/16)

–> ClusterIP service objecti yaratıldığı zaman, bu object’e bu IP aralığından **bir IP atanır** ve **ismiyle bu IP adresi Cluster’ın DNS mekanizmasına kaydedilir.** Bu IP adresi **Virtual (Sanal) bir IP adresidir.**

–> Kube-proxy tarafından tüm node’lardaki IP Table’a bu IP adresi eklenir. Bu IP adresine gelen her trafik Round Troppin algoritmasıyla Pod’lara yönlendirilir. Bu durum bizim 2 sıkıntımızı çözer:

1. Ben Frontend node’larına bu IP adresini tanımlayıp (ismine de `backend`dersem), Backend’e gitmen gerektiği zaman bu IP adresini kullan diyebilirim. (**Selector = app:backend**) Tek tek her seferinde Frontend podlarına Backend podlarının IP adreslerini eklemek zorunda kalmam!

:tada: :tada: **Özetle: ClusterIP Service’i Container’ları arasındaki haberleşmeyi, Service Discovery ve Load Balancing görevi üstlenerek çözer!**\\

***

### NodePort Service (Dış Dünya -> Container)

–> Bu service tipi, **Cluster dışından gelen bağlantıları** çözmek için kullanılır.

–> `NodePort` key’i kullanılır.\\

### LoadBalancer Service (Cloud Service’leri İçin)

–> Bu service tipi, sadece Agent K8s, Google K8s Engine gibi yerlerde kullanılır.\\

### ExternalName Service (Şuan için gereksiz.)

## Service Objecti Uygulaması

### Cluster IP Örneği

```shell
apiVersion: v1
kind: Service
metadata:
  name: backend # Service ismi
spec:
  type: ClusterIP # Service tipi
  selector:
    app: backend # Hangi podla ile eşleşeceği (pod'daki labella aynı)
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
```

Herhangi bir object, cluster içerisinde oluşan `clusterIP:5000` e istek attığında karşılık bulacaktır.

**Önemli Not:** Service’lerin ismi oluşturulduğunda şu formatta olur: `serviceName.namespaceName.svc.cluster.domain`

Eğer aynı namespace’de başka bir object bu servise gitmek istese core DNS çözümlemesi sayesinde direkt `backend` yazabilir. Başka bir namespaceden herhangi bir object ise ancak yukarıdaki **uzun ismi** kullanarak bu servise ulaşmalıdır.

### NodePort Örneği

* Unutulmamalıdır ki, tüm oluşan NodePort servislerinin de **ClusterIP’si** mevcuttur. Yani, içeriden bu ClusterIP kullanılarak bu servisle konuşulabilir.
* **`minikube service –url <serviceName>`** ile minikube kullanırken tunnel açabiliriz. Çünkü, biz normalde bu worker node’un içerisine dışardan erişemiyoruz. _Bu tamamen minikube ile alakalıdır._

```shell
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### Load Balancer Örneği

Google Cloud Service, Azure üzerinde oluşturulan cluster’larda çalışacak bir servistir.

```shell
apiVersion: v1
kind: Service
metadata:
  name: frontendlb
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### Imperative Olarak Service Oluşturma

```shell
kubectl delete service <serviceName>

# ClusterIP Service yaratmak
kubectl expose deployment backend --type=ClusterIP --name=backend

kubectl expose deployment backend --type=NodePort --name=backend
```

### Endpoint Nedir?

Nasıl deployment’lar aslında ReplicaSet oluşturduysa, Service objectleride arka planda birer Endpoint oluşturur. Service’lerimize gelen isteklerin nereye yönleneceği Endpoint’ler ile belirlenir.

```shell
kubectl get endpoints
```

Bir pod silindiğinde yeni oluşacak pod için, yeni bir endpoint oluşturulur.
