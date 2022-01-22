# 🚃 Kubectl

## Kubectl

![](<.gitbook/assets/Screen Shot 2021-12-12 at 23.53.55.png>)

* kubectl ile mevcut cluster’ı yönetimini **config** dosyası üzerinden yapmamız gerekir. Minikube gibi tool’lar config dosyalarını otomatik olarak oluşturur.
* **Default config** dosyasına `nano ~/.kube/config` yazarak ulaşabiliriz.
* VSCode’da açmak için

```
 cd ~/.kube 
 code .
```

* **context ->** Cluster ile user bilgilerini birleştirerek context bilgisini oluşturur. “_Bu cluster’a bu user’la bağlanacağım._” anlamına geliyor.

### Kubectl Config Komutları

**`kubectl config`**

–> Config ayarlarının düzenlenmesini sağlayan komuttur.

**`kubectl config get-contexts`**

–> Kubectl baktığı config dosyasındaki mevcut contextleri listeler.

#### **Current Context**

Current sütununda minikube’un yanında bir yıldız (\*) işareti görünür. Bunun anlamı: birden fazla context tanımlansa da **o an kullanılan context** anlamına gelir. Yapacağımız tüm işlemleri bu context’in içerisinde gerçekleşecektir.

**`kubectl config current-context`**

–> Kubectl baktığı config dosyasındaki current context’i verir.

**`kubectl config use-context <contextName>`**

–> Current context’i contextName olarak belirtilen context’i ile değiştirir.

ÖR: `kubectl config use-context docker-desktop` –> docker-desktop context’ine geçer.

## Kubectl Kullanım Komutları

* kubectl’de komutlar belli bir şemayla tasarlanmıştır:

```
 kubectl <fiil> <object> ​
 
 # <fiil> = get, delete, edit, apply 
 # <object> = pod
```

* kubectl’de aksi belirtilmedikçe tüm komutlar **config’de yazılan namespace’ler** üzerinde uygulanır. Config’de namespace belirtilmediyse, **default namespace** geçerlidir.

### `kubectl cluster-info`

–> Üzerinde işlem yaptığımız **cluster** ile ilgili bilgileri öğreneceğimiz komut.

### `kubectl get pods`

–> **Default namespace**‘deki pod’ları getirir.

### `kubectl get pods testpod`

–> **testpod** isimli pod’u getirir.

### **`kubectl get pods -n kube-system`**

–> **kube-system** isimli namespace’de tanımlanmış pod’ları getirir.

### `kubectl get pods -A`

–> **Tüm namespacelerdeki** pod’ları getirir.

### `kubectl get pods -A -o <wide|yaml|json>`

–> **Tüm namespacelerdeki** pod’ları **istenilen output formatında** getirir.

```shell
# jq -> json query pluginin kur.
# brew install jq

kubectl get pods -A -o json | jq -r ".items[].spec.containers[].name"
```

### `kubectl apply --help`

–> **apply** komutunun nasıl kullanılacağı ile ilgili bilgi verir. Ama `kubectl pod –-help` yazsak, bu pod ile ilgili **bilgi vermez.** Bunun yerine aşağıdaki **explain** komutu kullanılmalıdır.

### `kubectl explain pod`

\--> **pod** objesinin ne olduğunu, hangi field’ları aldığını gösterir.

\--> Çıkan output'ta **Version** ile hangi namespace’e ait olduğunu anlayabiliriz.

### `kubectl get pods`<mark style="color:red;">`-w`</mark>

\--> kubectl'i izleme (watch) moduna alır ve değişimlerin canlı olarak izlenmesini sağlar.

### `kubectl get all`<mark style="color:red;">`-A`</mark>

\--> Sistemde çalışan **tüm object'lerin durumunu** gösterir.

### `kubectl exec -it <podName> -c <containerName> -- bash`

\--> Pod içerisinde çalışan bir container'a bash ile bağlanmak için.

## Hızlı Kubectl Config Değiştirme

Hızlıca config değiştirmek için aşağıdaki bash scriptten yararlanabiliriz:

{% code title="change.sh" %}
```bash
#! /bin/bash

CLUSTER=$1

if [ -z "$1" ]
  then
    echo -e "\n##### No argument supplied. Please select one of these configs. #####"
    ls  ~/.kube |grep config- | cut -d "-" -f 2
    echo -e "######################################################################\n"
    #array=($(ls -d * |grep config_))
    read -p 'Please set config file: ' config
    cp -r ~/.kube/config_$config ~/.kube/config
    echo -e '\n'
    kubectl cluster-info |grep -v "To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'."
    kubectl config get-contexts
    kubectl get node -o wide |head -n 4
else
  cp -r ~/.kube/config-$CLUSTER ~/.kube/config
  if [ $? -ne 0 ];
  then
  exit 1
  fi
  echo -e '\n'
#  kubectl cluster-info |grep -v "To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'."
  kubectl config get-contexts
  echo -e '\n'
  kubectl get node -o wide |head -n 4
  echo -e '\n'
fi
```
{% endcode %}

Kullanım:

\--> Config dosyası `config-minikube` şeklinde oluşturulmalıdır. Script çalıştırırken config prefix'i ekliyor.

```
./change.sh <configName>

./change.sh minikube
```
