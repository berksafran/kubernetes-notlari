# ⛵ Nedir, Components, Nodes

## Neden Gerekli?

Örnek üzerinden gidelim:

Birden fazla microservice’in çalıştığı bir docker environment’ı düşünelim. Her bir microservice bir docker container içerisinde çalıştığını ve uygulamamızı kullanıcılara açtığımızı düşünelim. Uygulamamız, şuan tek bir sunucu üzerinde koşuyor ve sunucu üzerinde güncelleme yaptığımızda sistemde kesintiler oluşmaya başlayacaktır.

Bu sorunumuzu çözmek adına, yeni bir sunucu kiralayıp, aynı docker environment’ını kurgulayıp (clonelayıp), bir de **load balancer** kurarak dağıtık bir mimariye geçiş yaptık. Buna rağmen, docker container’lar kendiliğinden kapanıyor ve bu duruma manuel müdahale etmemiz gerekir. Daha fazla trafik aldığımız için, 2 sunucu daha kiraladık. 2 sunucu daha, 2 sunucu daha..

Tüm bu sunucuları **manuel yönetmeye devam ediyoruz.** Zamanla DevOps süreçlerine manuel müdahaleden dolayı çok fazla zaman harcamaya başladık, geriye kalan işlere zaman kalmadı.

Peki bu durumu çözecek durum nedir? Cevap -> **Container Orchestration!**

Tüm sistem konfigürasyonlarını ayarlayıp, bu karar mekanizmasını bir orkestra şefine emanet edebiliriz. İşte bu şef **Kubernetes’tir!**

Diğer alternatifler -> Docker Swarm, Apache H2o

## K8s Tarihçesi

* Google tarafından geliştirilen Orchestration sistemi.
* Google, çok uzun yıllardır Linux containerlar kullanıyor. Tüm bu containerları yönetmek için **Borg** adında bir platform geliştirmiş. Fakat, zamanla hatalar çıkmış ve yeni bir platform ihtiyacı duyulmuş ve **Omega** platformu geliştirilmiş.
* 2013 yılında 3 Google mühendisi open-source bir şekilde GitHub üzerinde Kubernetes reposunu açtı. _Kubernetes: Deniz dümencisi_ _(k8s)_
* 2014 senesinde proje, Google tarafından CNCF’e bağışlandı. (Cloud Native Computing Foundation)

## Kubernetes nedir?

* Declarative (_beyan temelli yapılandırma_), Container orchestration platformu.
* Proje, hiç bir şirkete bağlı değil, bir vakıf tarafından yönetilmektedir.
* Ücretsizdir. Rakip firmalarda **open-source**’dur.
* Bu kadar popüler olmasının sebebi, platformun tasarımı ve çözüm yaklaşımı.
* Semantic versioning izler (_x.y.z. -> x: major, y: minor, z: patch_) ve her 4 ayda bir minor version çıkartır.
* Her ay patch version yayınlar.
* Bir kubernetes platformu en fazla 1 yıl kullanılır, 1 yıldan sonra güncellemek gerekmektedir.

![](<.gitbook/assets/Screen Shot 2021-12-12 at 18.26.50.png>)

### Kubernetes Tasarımı ve Yaklaşımı

Birden fazla geliştirilebilir modüllerden oluşmaktadır. Bu modüllerin hepsinin bir görevi vardır ve tüm modüller kendi görevlerine odaklanır. İhtiyaç halinde bu modüller veya yeni modüller geliştirilebilir. (extendable)

K8s, “_Şunu yap, daha sonra şunu yap_” gibi adım adım ne yapmamız gerektiğini söylemek yerine _**(imperative yöntem);** “Ben şöyle bir şey istiyorum” **(declarative yöntem)**_ yaklaşımı sunmaktadır. **Nasıl yapılacağını tarif etmiyor, ne istediğimizi söylüyoruz.**

* Imperative yöntem, bize zaman kaybettirir, tüm adımları tasarlamak zorunda kalırız.
* Declarative yöntem’de ise sadece ne istediğimizi söylüyoruz ve sonuca bakıyoruz.

Kubernetes bize ondan istediğimizi soruyor, biz söylüyoruz ve bizim istediklerimizin dışına çıkmıyor. Örneğin, **Desired State (Deklara Edilen Durum - \_İstekler gibi düşünebiliriz**\_**)** şu şekilde olsun:

* Berksafran/k8s:latest isimli imaj yaratıp, 10 containerla çalıştır. Dış dünyaya 80 portunu aç ve ben bu service’e bir güncelleme geçtiğimde aynı anda 2 task üzerinde yürüt ve 10sn beklensin.

Kubernetes, 9 container çalışır durumda kalırsa hemen bir container daha ayağa kaldırır ve isteklerimize göre platformu **optimize eder.** Bu bizi, çok büyük bir işten kurtarıyor. _(Docker’da manuel olarak ayağa kaldırıyorduk, hatırlayın.)_

## Kubernetes Componentleri

K8s, **microservice mimarisi dikkate alınarak** oluşturulmuştur.

![](<.gitbook/assets/Screen Shot 2021-12-12 at 19.23.15.png>)

### **Control Plane** (Master Nodes)

Aşağıdaki, 4 component k8s yönetim kısmını oluşturur ve **master-node** üzerinde çalışır.

* **Master-node** -> Yönetim modullerinin çalıştığı yerdir.
* **Worker-node** -> İş yükünün çalıştığı yerdir.

![](<.gitbook/assets/Screen Shot 2021-12-12 at 18.48.28.png>)

* **kube-apiserver** **(api) –>** K8s’in beyni, **ana haberleşme merkezi, giriş noktasıdır**. Bir nev-i **Gateway** diyebiliriz. Tüm **componentler** ve **node**’lar, **kube-apiserver** üzerinden iletişim kurar. Ayrıca, dış dünya ile platform arasındaki iletişimi de **kube-apiserver** sağlar. Bu denli herkesle iletişim kurabilen **tek componenttir**. **Authentication ve Authorization** görevini üstlenir.
* **etcd** **->** K8s’in tüm cluster verisi, metada bilgileri ve Kubernetes platformunda oluşturulan componentlerin ve objelerin bilgileri burada tutulur. Bir nevi **Arşiv odası.** etcd, **key-value** şeklinde dataları tutar. Diğer componentler, etdc ile **direkt haberleşemezler.** Bu iletişimi, **kube-apiserver** aracılığıyla yaparlar.
* **kube-scheduler (sched) ->** K8s’i çalışma planlamasının yapıldığı yer. Yeni oluşturulan ya da bir node ataması yapılmamış Pod’ları izler ve üzerinde çalışacakları bir **node** seçer. (_Pod = container_) Bu seçimi yaparken, CPU, Ram vb. çeşitli parametreleri değerlendirir ve **bir seçme algoritması sayesinde pod için en uygun node’un hangisi olduğuna karar verir.**
* **kube-controller-manager (c-m) ->** K8s’in kontrollerinin yapıldığı yapıdır. **Mevcut durum ile istenilen durum arasında fark olup olmadığını denetler.** Örneğin; siz 3 cluster istediniz ve k8s bunu gerçekleştirdi. Fakat bir sorun oldu ve 2 container kaldı. kube-controller, burada devreye girer ve hemen bir cluster daha ayağa kaldırır. Tek bir binary olarak derlense de içerisinde bir çok controller barındırır:
  * Node Controller,
  * Job Controller,
  * Service Account & Token Controller,
  * Endpoints Controller.

### **Worker Nodes**

Containerlarımızın çalıştığı yerlerdir. Container veya Docker gibi container’lar çalıştırır. Her worker node’da **3 temel component** bulunur:

1. **Container runtime ->** Varsayılan olarak Docker’dır. Ama çeşitli sebeplerden dolayı **Docker’dan Containerd‘ye** geçmiştir. Docker ve containerd arasında fark yok nedebilecek kadar azdır. Hatta Docker kendi içerisinde de containerd kullanır. Diğer desteklenen container çeşidi **CRI-O’dur.**
2. **kubelet ->** API Server aracılığıyla etcd’yi kontrol eder ve sched tarafından bulunduğu node üzerinde çalışması gereken podları yaratır. Containerd’ye haber gönderir ve belirlenen özelliklerde bir container çalışmasını sağlar.
3. **kube-proxy ->** Nodelar üstünde ağ kurallarını ve trafik akışını yönetir. Pod’larla olan iletişime izin verir, izler.

Tüm bunların dışında GUI hizmeti sağlayan vb. pluginler de kurulmaktadır.
