# 👈 Rollout ve Rollback

Rollout ve Rollback kavramları, **deploment’ın** güncellemesi esnasında devreye girer, anlam kazanır.

**YAML** ile deployment tanımlaması yaparken **`strategy`** olarak 2 tip seçilir:

### Rollout Strategy - **`Recreate`**

```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rcdeployment
  labels:
    team: development
spec:
  replicas: 3
  selector:
    matchLabels:
      app: recreate 
  strategy:
    type: Recreate # recreate === Rollout strategy
... 
```

* “_Ben bu deployment’ta bir değişiklik yaparsam, öncelikle tüm podları sil, sonrasında yenilerini oluştur._” Bu yöntem daha çok **hardcore migration** yapıldığında kullanılır.

ÖR: Uygulamamızın yeni versionuyla eski versionunun birlikte çalışması **sakıncalı** ise bu yöntem seçilir.

### Rollback Strategy - **`RollingUpdate`**

```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolldeployment
  labels:
    team: development
spec:
  replicas: 10
  selector:
    matchLabels:
      app: rolling
  strategy:
    type: RollingUpdate # Rollback Strategy
    rollingUpdate:
      maxUnavailable: 2 # Güncelleme esnasında aynı anda kaç pod silineceği
      maxSurge: 2 # Güncelleme esnasında toplam aktif max pod sayısı
  template:
  ...
```

* Eğer YAML dosyasında strategy belirtmezseniz, **default olarak RollingUpdate seçilir.** **maxUnavailable ve maxSurge** değerleri ise default **%25’dir.**
* RollingUpdate, Create’in tam tersidir. “Ben bir değişiklik yaptığım zaman, hepsini silip; yenilerini **oluşturma**.” Bu strateji’de önemli 2 parametre vardır:
  * **`maxUnavailable`** –> En fazla burada yazılan sayı kadar pod’u sil. Bir güncellemeye başlandığı anda en fazla x kadar pod silinecek sayısı. (%20 de yazabiliriz.)
  * **`maxSurge`** –> Güncelleme geçiş sırasında sistemde toplamda kaç **max aktif pod’un olması gerektiği sayıdır.**

**Örnek**

Bir deployment ayağa kaldırdığımızı düşünelim. Image = nginx olsun. Aşağıdaki komut ile varolan deployment üzerinde güncelleme yapalım. nginx image'ı yerine httpd-alphine image'ının olmasını isteyelim:

```shell
kubectl set image deployment rolldeployment nginx=httpd-alphine --record=true
```

* `--record=true` parametresi bizim için tüm güncelleme aşamalarını kaydeder. Özellikle, bir önceki duruma geri dönmek istediğimizde işe yarar.

### Yapılan değişikliklerin listelenmesi

```shell
# rolldeployment = deploymentName
# tüm değişiklik listesi getirilir.
kubectl rollout history deployment rolldeployment 

# nelerin değiştiğini spesifik olarak görmek için:
kubectl rollout history deployment rolldeployment --revision=2
```

### Yapılan değişikliklerin geri alınması

```shell
# rolldeployment = deploymentName
# Bir önceki duruma geri dönmek için:
kubectl rollout undo deployment rolldeployment

# Spesifik bir revision'a geri dönmek için:
kubectl rollout undo deployment rolldeployment --to-revision=1
```

### Canlı olarak deployment güncellemeyi izlemek

```shell
# rolldeployment = deploymentName
kubectl rollout status deployment rolldeployment -w 
```

### Deployment güncellemesi esnasında pause’lamak

Güncelleme esnasında bir problem çıktı ve geri de dönmek istemiyorsak, ayrıca sorunun nereden kaynaklandığını da tespit etmek istiyorsak kullanılır.

```shell
# rolldeployment = deploymentName
kubectl rollout pause deployment rolldeployment
```

### Pause’lanan deployment güncellemesini devam ettirmek

```shell
kubectl rollout resume deployment rolldeployment
```
