# Dosya Ekleme
Nextcloud'umuza dosya eklediğimizde veri tabanında gerçekleşen değişimleri şu şekilde gösterebiliriz. Öncelikle sistemin var olan mysql dump dosyasını çıkarıyoruz. Daha sonra sisteme **"next.txt"** adlı bir metin belgesi yükledim(herhangi bir belge yüklenebilir). Windowsda kullanılabilir **Winmerge** programı ile iki dosya arasındaki oluşmuş değişiklikleri gözlemliyoruz. Değişikler aşağıda listelenmiştir.
```sql
INSERT INTO 'oc_activity' VALUES ..., (32,1594198142,30,'file_created','hardrez','hardrez','files','created_self','[{\"313\":\"\\/next.txt\"}]','','[]','/next.txt','http://192.168.1.7/nextcloud/index.php/apps/files/?dir=/','files',313);
```
Önceki eklenmiş olan dosyaları "..." temsil etmektedir. Yeni eklenen dosya için değerler kısmında görmüş olduğumuz parametrelerle beraber "oc_activity" tablosuna yeni bir girdi eklenmektedir. Bu parametrelerde dosyayı oluşturan kullanıcıyı, dosyanın oluşturulduğunu ve dosya adı gibi bilgileri görebiliriz.

```sql
INSERT INTO 'oc_filecache' VALUES ...,(313,2,'files/next.txt','c28c0328caf88e9df740f43f3847fa1a',6,'next.txt',20,6,10,1594128594,1594128594,0,0,'8d382065a9c651e9a3cd46cd6bc3a1ee',27,'');
```
Önceki eklenmiş olan dosyaları "..." temsil etmektedir. Burada da yine **313** yani eklemiş olduğumuz dosya id'sini görebiliriz. Burda veritabanına eklemiş olduğumuz dosyaya ait sırasıyla; fileid(dosya id), storage(depolama), path(dosya yolu), path_hash(dosya yolunun hashlenmiş hali), parent, name(dosya adı), mimetype(dosya türü/uzantısı), mimepart, size(dosya boyutu), mtime(değişiklik zamanı), storage_mtime(yüklenme zamanı), encrypted(code), unencrypted_size(decode boyutu), etag(etiketi), permissions(erişim izni), checksum parametreleri yer almaktadır

```sql
INSERT INTO 'oc_filecache_extended' VALUES ...,(313,NULL,0,1594198142);
```
Burada da kod bloğunda yer alan "..." ifadesi ile belirtilen kısım önceki dumpda halihazırda bulunan değerlerdir. Görmüş olduğunuz gibi "**313**" numarası ilk kod bloğunda da vardı, bu bizim dosyamızın id numarası diyebiliriz. Burda da 313 nolu dosyanın cache extended bilgilerini tutmaktadır. **NULL** ile gösterilen kısım dosyaya ait **metadata_etag**'i, "0" ile gösterilen kısım dosyanın oluşturulma zamanını, en son yazılan parametre ise dosyanın yüklenme zamanının girdisini belirtmektedir.