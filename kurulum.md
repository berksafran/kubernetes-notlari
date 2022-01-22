# 📀 Kurulum

Tüm komutlar, kube-apiserver üzerinden verilir. Bu komutlar 3 şekilde verilebilir:

1. **REST Api** aracılığıyla (terminalden curl olarak vb.),
2. **Kubernetes GUI** (Dashboard, Lens, Octant) üzerinden,
3. **Kubectl** aracılığıyla (CLI).

## Kubectl Kurulumu

* Homebrew ile aşağıdaki komutu yazıyoruz. (kubernetes.io’da farklı kurulumlar mevcut)

```
brew install kubectl
```

* Test için `kubectl version` yazabilirsiniz. (Server bağlantısı yapılmadığı için o kısımda hata alınabilir, normaldir.)

## Kubernetes Kurulumu

### Hangi version’u kuracağız?

* En light-weight version için -> **minikube**, **Docker Desktop (Single Node K8s Cluster)**
* Diğer seçenekler -> **kubeadm, kubespray**
* Cloud çözümler -> **Azure Kubernetes Service (AKS), Google Kubernetes Engine, Amazon EKS**

### Docker Desktop

* Docker Desktop, Single Node Kubernetes Cluster ayağa kaldırmaya imkan tanıyor. Bu durum, **başka bir araca duymadan Kubernetes üzerinde işlem yapabilme yeteneği kazandırıyor.** Ama tavsiye olarak **minikube kullanılmasıdır!**
* Docker Desktop içerisinde K8s kurulumu için, Settings > Kubernetes’e gidip install etmeniz gerekiyor.

### :large\_blue\_diamond: Minikube

* Bir çok addon ile gelebiliyor. Tek bir komut ile cluster’ı durdurup, çalıştırabiliyoruz.

```
brew install minikube
```

* **minikube kullanabilmek için sistemde Docker yüklü olması gerekiyor.** Çünkü, Minikube background’da Docker’ı kullanacaktır. VirtualBox gibi bir çok tool’u da background olarak kullanabiliriz.
* Test için `minikube status`

#### **Minikube üzerinde K8s Cluster Kurulumu**

Varsayılan olarak Docker’ı background’da kullanır.

```shell
minikube start

minikube start --driver=virtualbox # VirtualBox background'ında çalıştırmak için.
```

Test için

```shell
minikube status
kubectl get nodes
```

**Kubernetes cluster’ı ve içeriğini (tüm podları) silmek için**

```shell
minikube delete
```

**Kubernetes cluster’ını durdurmak için**

```shell
minikube stop
```

## :warning::warning:\[WIP] kubeadm Kurulumu

\-> Kubernetes cluster’ı oluşturmamızı sağlayan başka bir platformdur. minikube’e göre daha gelişmiştir. Rassbery Pi üzerinde de çalışabilir :)

> **Buraya yazılacak diğer tutorial’lar:**
>
> * Google Cloud Platform’unda Kurulum,
> * AWS'de Kurulum,
> * Azure'da Kurulum.

## Play-with-kubernetes Kurulumu

* Eğer cloud için kredi kartınızı vermek istemiyorsanız ya da hızlıca bazı denemeler yapmak istiyorsanız, **play-with-kubernetes** tam size göre.
* 4 saatlik kullanım sınırı var. 4 saat sonra sistem sıfırlanıyor, ayarlar gidiyor.
* Browser based çalışır.
* Toplam max 5 tane node oluşturabiliyorsunuz.

## Tools

* **Lens -->** Kubernetes için çok iyi hazırlanmış bir yönetim tool'u.
* **kubectx -->** Hızlı config/context geçişi için.
* **Krew -->** kubectl için plugin-set'leri
