---
description: Pod Nedir, Komutlar, YAML, Çoklu Container …
---

# 🟡 Pod

## Pod Nedir?

* K8s üzerinde koşturduğumuz, çalıştırdığımız, deploy ettiğimiz şeylere **Kubernetes Object** denir.
* **En temel object Pod‘dur.**
* K8s’den önce hep docker container ile çalıştık. **K8s’de ise biz, direkt olarak container oluşturmayız.** K8s Dünya’sında oluşturabileceğimiz, yönetebileceğimiz **en küçük birim Pod’dur.**
* Pod’lar **bir veya birden** fazla container barındırabilir. **Best Practice için her bir pod bir tek container barındırır.**
* Her pod’un **unique bir ID’si (uid)** vardır ve **unique bir IP’si** vardır. Api-server, bu uid ve IP’yi **etcd’ye kaydeder.** Scheduler ise herhangi bir podun node ile ilişkisi kurulmadığını görürse, o podu çalıştırması için **uygun bir worker node** seçer ve bu bilgiyi pod tanımına ekler. Pod içerisinde çalışan **kubelet** servisi bu pod tanımını görür ve ilgili container’ı çalıştırır.
* Aynı pod içerisindeki containerlar aynı node üzerinde çalıştırılır ve bu containerlar localhost üzerinden haberleşir.
* **`kubectl run`** şeklinde pod oluşturulur.

## Pod Oluşturma

```shell
kubectl run firstpod --image=nginx --restart=Never --port=80 --labels="app=frontend" 

# Output: pod/firstpod created
```

* `–restart` -> Eğer pod içerisindeki container bir sebepten ötürü durursa, tekrar çalıştırılmaması için `Never` yazdık.

### Tanımlanan Podları Gösterme

```shell
kubectl get pods -o wide

# -o wide --> Daha geniş table gösterimi için.
```

### Bir Object’in Detaylarını Görmek

```shell
kubectl describe <object> <objectName>

kubectl describe pods first-pod
```

* `first-pod` podu ile ilgili tüm bilgileri getirir.
* Bilgiler içerisinde **Events**‘e dikkat. Pod’un tarihçesi, neler olmuş, k8s neler yapmış görebiliriz.
  * Önce Scheduler node ataması yapar,
  * kubelet container image’ı pull’lamış,
  * kubelet pod oluşturulmuş.

### Pod Loglarını Görmek

```shell
kubectl logs <podName>

kubectl logs first-pod
```

**Logları Canlı Olarak (Realtime) Görmek**

```shell
kubectl logs -f <podName>
```

### Pod İçerisinde Komut Çalıştırma

```
kubectl exec <podName> -- <command>

kubectl exec first-pod -- ls /
```

### Pod İçerisindeki Container’a Bağlanma

```shell
kubectl exec -it <podName> -- <shellName>

kubectl exec -it first-pod -- /bin/sh
```

**Eğer 1 Pod içerisinde 1'den fazla container varsa:**

```
kubectl exec -it <podName> -c <containerName> -- <bash|/bin/sh>
```

\-> Bağlandıktan sonra kullanabileceğimiz bazı komutlar:

```shell
hostname #pod ismini verir.
printenv #Pod env variables'ları getirir.
```

\--> Eğer bir pod içerisinde birden fazla container varsa, istediğimiz container'a bağlanmak için:

```
kubectl exec -it <podName> -c <containerName> -- /bin/sh
```

### Pod’u Silme

```shell
kubectl delete pods <podName>
```

\-> Silme işlemi yaparken dikkat! Çünkü, confirm almıyor. **Direkt siliyor!** Özellikle production’da dikkat!

## YAML

* k8s, declarative yöntem olarak **YAML** veya **JSON** destekler.
* **`---` (üç tire) koyarak bir YAML dosyası içerisinde birden fazla object belirletebiliriz.**

_k8sfundamentals/pod/objectbasetemplate.yaml dosyasını açtık._

```yaml
apiVersion:
kind:
metadata:
spec:
```

* Her türlü object oluşturulurken; **apiVersion, kind ve metadata** olmak zorundadır.
* **`kind`** –> Hangi object türünü oluşturmak istiyorsak buraya yazarız. ÖR: `pod`
* **`apiVersion`** –> Oluşturmak istediğimiz object’in hangi API üzerinde ya da endpoint üzerinde sunulduğunu gösterir.
* **`metadata`** –> Object ile ilgili unique bilgileri tanımladığımız yerdir. ÖR: `namespace`, `annotation` vb.
* **`spec`** –> Oluşturmak istediğimiz object’in özelliklerini belirttiğimiz yerdir. Her object için gireceğimiz bilgiler farklıdır. Burada yazacağımız tanımları, dokümantasyondan bakabiliriz.

### apiVersion’u Nereden Bulacağız?

1. Dokümantasyona bakabiliriz.
2. kubectl aracılığıyla öğrenebiliriz:

```shell
kubectl explain pods
```

Yukarıdaki explain komutunu yazarak pod’un özelliklerini öğrenebiliriz.

–> `Versions` karşısında yazan bizim `apiVersion`umuzdur.

### metadata ve spec Yazımı

> _Aşağıdaki yaml’ı k8sfundamentals/pod/pod1.yaml dosyasında görebilirsin._

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: first-pod 	# Pod'un ismi
	labels:			# Atayacağımız etiketleri yazabiliriz.
		app: front-end  # app = front-end label'i oluşturduk.
spec:	
	containers: 			# Container tanımlamaları yapıyoruz.
	- name: nginx			# Container ismi
		image: nginx:latest
		ports:
		- containerPort: 80 # Container'e dışarıdan erişilecek port
```

### K8s’e YAML Dosyasını Declare Etme

```shell
kubectl apply -f pod1.yaml
```

* pod1.yaml dosyasını al ve burada tanımlanan object’i benim için oluştur.
* `kubectl describe pods firstpod` ile oluşturulan pod’un tüm özellikleri görüntülenir.
* YAML yöntemi kullanmak pipeline’da yaratmakta kullanılabilir.
* :thumbsup: **Declarative Yöntem Avantajı** –> Imperative yöntemle bir pod tanımladığımızda, bir özelliğini update etmek istediğimizde “Already exists.” hatası alırız. Ama declarative olarak YAML’ı değiştirip, update etmek isteseydik ve apply komutunu kullansaydık “pod configured” success mesajını alırız.

### Declare Edilen YAML Dosyasını Durdurma

```shell
kubectl delete -f pod1.yaml
```

### Kubectl ile Pod’ları Direkt Değiştirmek (Edit)

```shell
kubectl edit pods <podName>
```

* Herhangi tanımlanmış bir pod’un özelliklerini direkt değiştirmek için kullanılır.
* `i`tuşuna basarak `INSERT`moduna geçip, editleme yaparız.
* CMD + C ile çıkıp, `:wq` ile VIM’den çıkabiliriz.
* Pod’un editlendiği mesajını görürüz.
* Tercih edilen bir yöntem değildir, YAML + `kubectl apply` tercih edilmelidir.

## Pod Yaşam Döngüsü

* **Pending** –> Pod oluşturmak için bir YAML dosyası yazdığımızda, YAML dosyasında yazan configlerle varsayılanlar harmanlanır ve etcd’ye kaydolur.
* **Creating** –> kube-sched, etcd’yi sürekli izler ve herhangi bir node’a atanmamış pod görülürse devreye girer ve en uygun node’u seçer ve node bilgisini ekler. Eğer bu aşamada takılı kalıyorsa, **uygun bir node bulunamadığı anlamına gelir.**
  * etcd’yi sürekli izler ve bulunduğu node’a atanmış podları tespit eder. Buna göre containerları oluşturmak için image’leri download eder. Eğer image bulunamazsa veya repodan çekilemezse **ImagePullBackOff** durumuna geçer.
  * Eğer image doğru bir şekilde çekilir ve containerlar oluşmaya başlarsa Pod **Running** durumuna geçer.

> _Burada bir S verelim.. Container’ların çalışma mantığından bahsedelim:_

* Container imagelarında sürekli çalışması gereken bir uygulama bulunur. Bu uygulama çalıştığı sürece container da çalışır durumdadır. Uygulama 3 şekilde çalışmasını sonlandırır:
  1. Uygulama tüm görevlerini tamamlar ve hatasız kapanır.
  2. Kullanıcı veya sistem kapanma sinyali gönderir ve hatasız kapanır.
  3. Hata verir, çöker, kapanır.

> _Döngüye geri dönelim.._

* Container uygulamasının durmasına karşılık, Pod içerisinde bir **RestartPolicy** tanımlanır ve 3 değer alır:
  * **`Always`** -> Kubelet bu container’ı yeniden başlatır.
  * **`Never`** -> Kubelet bu container’ı yeniden **başlatmaz**.
  * **`On-failure`** -> Kubelet sadece container hata alınca başlatır.\\
* **Succeeded** -> Pod başarıyla oluşturulmuşsa bu duruma geçer.
* **Failed** -> Pod başarıyla oluşturulmamışsa bu duruma geçer.
* **Completed** -> Pod başarıyla oluşturulup, çalıştırılır ve hatasız kapanırsa bu duruma geçer.
* :warning: **CrashLookBackOff** -> Pod oluşturulup sık sık kapanıyorsa ve RestartPolicy’den dolayı sürekli yeniden başlatılmaya çalışılıyorsa, k8s bunu algılar ve bu podu bu state’e getirir. Bu state de olan **podlar incelenmelidir.**

## Multi Container Pods

### **Neden 2 uygulamayı aynı container’a koymuyoruz?**

–> Cevap: İzolasyon. 2 uygulama izole çalışsın. Eğer bu izolasyonu sağlamazsanız, yatay scaling yapamazsınız. Bu durumu çoklamak gerektiğinde ve 2. containeri aldığımızda 2 tane MySQL, 2 tane Wordpress olacak ki bu iyi bir şey değil.

:ok\_hand: Bu sebeple **1 Pod = 1 Container = 1 uygulama** olmalıdır! Diğer senaryolar **Anti-Pattern** olur.

### Peki, neden pod’lar neden multi-container’a izin veriyor?

–> Cevap: Bazı uygulamalar bütünleşik (bağımlı) çalışır. Yani ana uygulama çalıştığında çalışmalı, durduğunda durmalıdır. Bu tür durumlarda bir pod içerisine birden fazla container koyabiliriz.

–> Bir pod içerisindeki 2 container için network gerekmez, localhost üzerinden çalışabilir.

\-> Eğer multi-container’a sahip pod varsa ve bu containerlardan birine bağlanmak istersek:

```shell
kubectl exec -it <podName> -c <containerName> -- /bin/sh
```

> _k8sfundamentals/podmulticontainer.yaml dosyası örneğine bakabilirsiniz._

### Init Container ile bir Pod içerisinde birden fazla Container Çalıştırma

Go’daki `init()` komutu gibi ilk çalışan container’dır. Örneğin, uygulama container’ın başlayabilmesi için bazı config dosyalarını fetch etmesi gerekir. Bu işlemi init container içerisinde yapabiliriz.

1. Uygulama container’ı başlatılmadan önce **Init Container** ilk olarak çalışır.
2. Init Container yapması gerekenleri yapar ve kapanır.
3. Uygulama container’ı, Init Container kapandıktan sonra çalışmaya başlar. **Init Container kapanmadan uygulama container’ı başlamaz.**

> _k8sfundamentals/podinitcontainer.yaml dosyası örneğine bakabilirsiniz._
