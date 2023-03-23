# Kendi SMTP posta gönderme sunucunuzu oluşturun

## başlangıç

SMTP, aşağıdakiler gibi hizmetleri doğrudan bulut satıcılarından satın alabilir:

* [Amazon SES SMTP'si](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali bulut e-posta gönderme](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Kendi posta sunucunuzu da oluşturabilirsiniz - sınırsız gönderme, düşük toplam maliyet.

Aşağıda, kendi posta sunucumuzu nasıl oluşturacağımızı adım adım gösteriyoruz.

## sunucu seçimi

Şirket içinde barındırılan SMTP sunucusu, 25, 456 ve 587 numaralı bağlantı noktalarının açık olduğu bir genel IP gerektirir.

Yaygın olarak kullanılan genel bulutlar varsayılan olarak bu portları bloke etmiştir ve iş emri vererek açmak mümkün olabilir ama sonuçta çok zahmetlidir.

Bu portların açık olduğu ve ters etki alanı adlarının ayarlanmasını destekleyen bir ana bilgisayardan satın almanızı öneririm.

Burada [Contabo'yu](https://contabo.com) öneririm.

Contabo, 2003 yılında Almanya'nın Münih kentinde çok rekabetçi fiyatlarla kurulmuş bir barındırma sağlayıcısıdır.

Satın alma para birimi olarak Euro'yu seçerseniz, fiyat daha ucuz olacaktır (8 GB belleğe ve 4 CPU'ya sahip bir sunucunun yıllık maliyeti yaklaşık 529 yuan'dır ve ilk kurulum ücreti bir yıl boyunca ücretsizdir).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Sipariş verirken, `prefer AMD` edin ve AMD CPU'lu sunucunun daha iyi performansa sahip olacağını unutmayın.

Aşağıda, kendi posta sunucunuzu nasıl oluşturacağınızı göstermek için Contabo'nun VPS'sini örnek olarak alacağım.

## Ubuntu sistem yapılandırması

Buradaki işletim sistemi Ubuntu 22.04'tür.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Ssh üzerindeki sunucuda `Welcome to TinyCore 13!` (aşağıdaki şekilde gösterildiği gibi), sistemin henüz kurulmadığı anlamına gelir. Lütfen ssh bağlantısını kesin ve tekrar oturum açmak için birkaç dakika bekleyin.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

`Welcome to Ubuntu 22.04.1 LTS` göründüğünde, başlatma işlemi tamamlanmıştır ve aşağıdaki adımlarla devam edebilirsiniz.

### [İsteğe bağlı] Geliştirme ortamını başlatın

Bu adım isteğe bağlıdır.

Kolaylık sağlamak için ubuntu yazılımının kurulumunu ve sistem yapılandırmasını [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) içine koydum.

Tek tıklama ile yüklemek için aşağıdaki komutu çalıştırın.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Çinli kullanıcılar, lütfen bunun yerine aşağıdaki komutu kullanın; dil, saat dilimi vb. otomatik olarak ayarlanacaktır.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo, IPV6'yı etkinleştirir

SMTP'nin IPV6 adresleriyle e-postalar da gönderebilmesi için IPV6'yı etkinleştirin.

`/etc/sysctl.conf` dosyasını düzenleyin

Aşağıdaki satırları değiştirin veya ekleyin

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

[Contabo eğitimini takip edin: Sunucunuza IPv6 bağlantısı ekleme](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

`/etc/netplan/01-netcfg.yaml` dosyasını düzenleyin, aşağıdaki şekilde gösterildiği gibi birkaç satır ekleyin (Contabo VPS varsayılan yapılandırma dosyasında zaten bu satırlar var, açıklamaları kaldırmanız yeterli).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Ardından, değiştirilen yapılandırmanın yürürlüğe girmesi için `netplan apply` .

Yapılandırma başarılı olduktan sonra, harici ağınızın ipv6 adresini görüntülemek için `curl 6.ipw.cn` kullanabilirsiniz.

## Yapılandırma deposu işlemlerini klonlayın

```
git clone https://github.com/wactax/ops.soft.git
```

## Alan adınız için ücretsiz bir SSL sertifikası oluşturun

Posta göndermek, şifreleme ve imzalama için bir SSL sertifikası gerektirir.

Sertifika oluşturmak için [acme.sh](https://github.com/acmesh-official/acme.sh) kullanıyoruz.

acme.sh, açık kaynaklı bir otomatik sertifika imzalama aracıdır,

Yapılandırma ambarı ops.soft'a girin, `./ssl.sh` dosyasını çalıştırın ve **üst dizinde** bir `conf` klasörü oluşturulacaktır.

[acme.sh](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) dnsapi'den DNS sağlayıcınızı bulun, `conf/conf.sh` dosyasını düzenleyin.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Ardından, alan adınız için `123.com` ve `*.123.com` sertifikaları oluşturmak üzere `./ssl.sh 123.com` çalıştırın.

İlk çalıştırma, [acme.sh'yi](https://github.com/acmesh-official/acme.sh) otomatik olarak yükleyecek ve otomatik yenileme için zamanlanmış bir görev ekleyecektir. Görüyorsunuz `crontab -l` aşağıdaki gibi bir satır var.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Oluşturulan sertifikanın yolu, `/mnt/www/.acme.sh/123.com_ecc。`

Sertifika yenileme `conf/reload/123.com.sh` betiğini arayacak, bu betiği düzenleyin, ilgili uygulamaların sertifika önbelleğini yenilemek için `nginx -s reload` gibi komutlar ekleyebilirsiniz.

## Chasquid ile SMTP sunucusu oluşturun

[chasquid](https://github.com/albertito/chasquid) , Go dilinde yazılmış açık kaynaklı bir SMTP sunucusudur.

Postfix ve Sendmail gibi eski posta sunucusu programlarının yerine geçen chasquid'in kullanımı daha basit ve kolaydır ve ikincil geliştirme için de daha kolaydır.

`./chasquid/init.sh 123.com` tek tıklamayla otomatik olarak kurulacaktır (123.com'u gönderen alan adınızla değiştirin).

## E-posta İmzası DKIM'i Yapılandırın

DKIM, mektupların spam olarak değerlendirilmesini önlemek için e-posta imzaları göndermek için kullanılır.

Komut başarılı bir şekilde çalıştıktan sonra, DKIM kaydını (aşağıda gösterildiği gibi) ayarlamanız istenecektir.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

DNS'nize bir TXT kaydı eklemeniz yeterlidir (aşağıda gösterildiği gibi).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Hizmet durumunu ve günlükleri görüntüleyin

 `systemctl status chasquid` Hizmet durumunu görüntüleyin.

Normal çalışma durumu aşağıdaki şekilde gösterildiği gibidir

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` veya `journalctl -xeu chasquid` hata günlüğünü görüntüleyebilir.

## Ters etki alanı adı yapılandırması

Ters etki alanı adı, IP adresinin karşılık gelen etki alanı adına çözümlenmesine izin vermek içindir.

Ters bir etki alanı adı ayarlamak, e-postaların spam olarak tanımlanmasını engelleyebilir.

Posta alındığında, alıcı sunucu, gönderen sunucunun geçerli bir ters alan adına sahip olup olmadığını doğrulamak için gönderen sunucunun IP adresinde ters alan adı analizi yapacaktır.

Gönderen sunucunun ters alan adı yoksa veya ters alan adı gönderen sunucunun IP adresiyle eşleşmiyorsa, alıcı sunucu e-postayı spam olarak algılayabilir veya reddedebilir.

[https://my.contabo.com/rdns adresini](https://my.contabo.com/rdns) ziyaret edin ve aşağıda gösterildiği gibi yapılandırın

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Ters etki alanı adını ayarladıktan sonra, alan adı ipv4 ve ipv6'nın sunucuya ileri çözünürlüğünü yapılandırmayı unutmayın.

## Chasquid.conf ana bilgisayar adını düzenleyin

`conf/chasquid/chasquid.conf` dosyasını ters etki alanı adının değerine değiştirin.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Ardından, hizmeti yeniden başlatmak için `systemctl restart chasquid` çalıştırın.

## Conf'u git deposuna yedekle

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Örneğin, conf klasörünü kendi github işlemime aşağıdaki gibi yedekliyorum

Önce özel bir depo oluşturun

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Conf dizinine girin ve depoya gönderin

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## gönderen ekle

koşmak

```
chasquid-util user-add i@wac.tax
```

gönderen ekleyebilir

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Parolanın doğru ayarlandığını doğrulayın

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Kullanıcıyı ekledikten sonra, `chasquid/domains/wac.tax/users` güncellenecektir, depoya göndermeyi unutmayın.

## DNS SPF kaydı ekle

SPF (Sender Policy Framework), e-posta sahtekarlığını önlemek için kullanılan bir e-posta doğrulama teknolojisidir.

Bir posta gönderenin kimliğini, gönderenin IP adresinin olduğunu iddia ettiği alan adının DNS kayıtlarıyla eşleşip eşleşmediğini kontrol ederek doğrular ve dolandırıcıların sahte e-postalar göndermesini engeller.

SPF kayıtları eklemek, e-postaların spam olarak tanımlanmasını mümkün olduğunca engelleyebilir.

Alan adı sunucunuz SPF tipini desteklemiyorsa, TXT tipi kayıt eklemeniz yeterlidir.

Örneğin, `wac.tax` SPF'si aşağıdaki gibidir

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

`_spf.wac.tax` için SPF

`v=spf1 a:smtp.wac.tax ~all`

Burada `include:_spf.google.com` verdiğimi unutmayın, çünkü `i@wac.tax` daha sonra Google posta kutusunda gönderen adres olarak yapılandıracağım.

## DNS yapılandırması DMARC

DMARC, (Domain-based Message Authentication, Reporting & Conformance) ifadesinin kısaltmasıdır.

SPF sıçramalarını yakalamak için kullanılır (belki yapılandırma hatalarından kaynaklanır veya spam göndermek için başka biri sizmiş gibi davranır).

TXT kaydı ekle `_dmarc` ,

içerik aşağıdaki gibidir

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Her parametrenin anlamı aşağıdaki gibidir

### p (Politika)

SPF (Sender Policy Framework) veya DKIM (DomainKeys Identified Mail) doğrulamasında başarısız olan e-postaların nasıl işleneceğini gösterir. p parametresi üç değerden birine ayarlanabilir:

* hiçbiri: Herhangi bir işlem yapılmaz, yalnızca doğrulama sonucu e-posta raporlama mekanizması aracılığıyla gönderene geri gönderilir.
* Karantina: Doğrulamayı geçemeyen postayı spam klasörüne koyun, ancak postayı doğrudan reddetmeyecek.
* reddet: Doğrulamada başarısız olan e-postaları doğrudan reddedin.

### fo (Hata Seçenekleri)

Raporlama mekanizması tarafından döndürülen bilgi miktarını belirtir. Aşağıdaki değerlerden birine ayarlanabilir:

* 0: Tüm mesajlar için doğrulama sonuçlarını raporla
* 1: Yalnızca doğrulamada başarısız olan iletileri bildirin
* d: Yalnızca alan adı doğrulama hatalarını bildirin
* s: yalnızca SPF doğrulama hatalarını bildirin
* l: Yalnızca DKIM doğrulama hatalarını bildirin

### rua ve ruf

* rua (Toplu raporlar için Raporlama URI'si): Toplu raporları almak için e-posta adresi
* ruf (Adli raporlar için Raporlama URI'si): ayrıntılı raporlar almak için e-posta adresi

## E-postaları Google Mail'e yönlendirmek için MX kayıtları ekleyin

Evrensel adresleri destekleyen ücretsiz bir kurumsal posta kutusu bulamadığım için (Tümünü Yakala, bu alan adına gönderilen tüm e-postaları önek kısıtlaması olmadan alabilir), tüm e-postaları Gmail posta kutuma yönlendirmek için chasquid kullandım.

**Kendi ücretli iş posta kutunuz varsa, lütfen MX'i değiştirmeyin ve bu adımı atlayın.**

`conf/chasquid/domains/wac.tax/aliases` düzenleyin, yönlendirme posta kutusunu ayarlayın

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` tüm e-postaları gösterir, `i` yukarıda oluşturulan gönderen kullanıcının e-posta adresi önekidir. Posta iletmek için her kullanıcının bir satır eklemesi gerekir.

Ardından MX kaydını ekleyin (Aşağıdaki şekilde ilk satırda gösterildiği gibi burada doğrudan ters alan adının adresini gösteriyorum).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Yapılandırma tamamlandıktan sonra, Gmail'de e-posta alıp alamadığınızı görmek için `i@wac.tax` ve `any123@wac.tax` adreslerine e-posta göndermek için diğer e-posta adreslerini kullanabilirsiniz.

Değilse, chasquid günlüğünü kontrol edin ( `grep chasquid /var/log/syslog` ).

## Google Mail ile i@wac.tax adresine bir e-posta gönderin

Google Mail e-postayı aldıktan sonra doğal olarak i.wac.tax@gmail.com yerine `i@wac.tax` ile yanıt vermeyi umdum.

[https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) adresini ziyaret edin ve "Başka bir e-posta adresi ekle"yi tıklayın.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Ardından, yönlendirilen e-posta tarafından alınan doğrulama kodunu girin.

Son olarak, varsayılan gönderen adresi olarak ayarlanabilir (aynı adresle yanıt verme seçeneğiyle birlikte).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Bu sayede SMTP mail sunucusunun kurulumunu tamamladık ve aynı zamanda e-posta gönderip almak için Google Mail'i kullanıyoruz.

## Yapılandırmanın başarılı olup olmadığını kontrol etmek için bir test e-postası gönderin

`ops/chasquid` girin

`direnv allow` (direnv, önceki tek tuşlu başlatma işleminde kuruldu ve kabuğa bir kanca eklendi)

o zaman koş

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Parametrelerin anlamı aşağıdaki gibidir

* kullanıcı: SMTP kullanıcı adı
* geçiş: SMTP şifresi
* alıcıya

Bir test e-postası gönderebilirsiniz.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Yapılandırmaların başarılı olup olmadığını kontrol etmek amacıyla test e-postaları almak için Gmail'i kullanmanız önerilir.

### TLS standart şifreleme

Aşağıdaki şekilde gösterildiği gibi, SSL sertifikasının başarıyla etkinleştirildiği anlamına gelen bu küçük kilit vardır.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Ardından "Orijinal E-postayı Göster"i tıklayın

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKİM

Aşağıdaki şekilde gösterildiği gibi, Gmail orijinal posta sayfasında DKIM görüntüleniyor, bu da DKIM yapılandırmasının başarılı olduğu anlamına geliyor.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Orijinal e-postanın başlığındaki Alındı'yı kontrol edin ve gönderen adresinin IPV6 olduğunu görebilirsiniz, bu da IPV6'nın da başarıyla yapılandırıldığı anlamına gelir.
