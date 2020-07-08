# Dosya Ekleme
Nextcloud'umuza dosya eklediğimizde veri tabanında gerçekleşen değişimleri şu şekilde gösterebiliriz. Öncelikle sistemin var olan mysql dump dosyasını çıkarıyoruz. Daha sonra sisteme **"next.txt"** adlı bir metin belgesi yükledim(herhangi bir belge yüklenebilir). Windowsda kullanılabilir **Winmerge** programı ile iki dosya arasındaki oluşmuş değişiklikleri gözlemliyoruz. Değişikler aşağıda listelenmiştir.
```sql
INSERT INTO 'oc_activity' VALUES ..., (32,1594198142,30,'file_created','hardrez','hardrez','files','created_self','[{\"313\":\"\\/next.txt\"}]','','[]','/next.txt','http://192.168.1.7/nextcloud/index.php/apps/files/?dir=/','files',313);
```
Önceki eklenmiş olan dosyaları "..." temsil etmektedir. Yeni eklenen dosya için değerler kısmında görmüş olduğumuz parametrelerle beraber "oc_activity" tablosuna yeni bir girdi eklenmektedir. Tabloya ait parametreler sırasıyla; 
1) activity_id(id'si) 
2) timestamp(zaman damgası)
3) priority(öncelik*) 
4) type(işlem türü bknz. file_created) 
5) user(kullanıcı)
6) affecteduser(olaydan etkilenen kullanıcı)
7) app(nextclouddaki hangi uygulama olduğu)
8) subject(başlık) 
9) subjectparams(başlığa ait parametre, uzantı)
10) message 
11) messageparams 
12) file(yüklendiği konum) 
13) link(uzantısı)
14) object_type(nesnenin türü, uygulamalara göre)
15) object_id(belirlenmiş nesne id'si) 

şeklinde yer almaktadır.Burada dosya eklediğimizde hangi parametrelerin etkilendiğini rahatça görebilir ve anlamlandırabiliriz.( * koyulan ifade belirsizliği temsil etmektedir)

```sql
INSERT INTO 'oc_filecache' VALUES ...,(313,2,'files/next.txt','c28c0328caf88e9df740f43f3847fa1a',6,'next.txt',20,6,10,1594128594,1594128594,0,0,'8d382065a9c651e9a3cd46cd6bc3a1ee',27,'');
```
Önceki eklenmiş olan dosyaları "..." temsil etmektedir. Burada da yine **313** yani eklemiş olduğumuz dosya id'sini görebiliriz. "oc_filecache" tablosuna ait parametreler sırasıyla; 
1) fileid(dosya id)
2) storage(depolama)
3) path(dosya yolu)
4) path_hash(dosya yolunun hashlenmiş hali)
5) parent, name(dosya adı)
6) mimetype(dosya türü/uzantısı)
7) mimepart, size(dosya boyutu)
8) mtime(değişiklik zamanı)
9) storage_mtime(yüklenme zamanı)
10) encrypted(code)
11) unencrypted_size(decode boyutu)
12) etag(etiketi)
13) permissions(erişim izni)
14) checksum 

parametreleri yer almaktadır.

```sql
INSERT INTO 'oc_filecache_extended' VALUES ...,(313,NULL,0,1594198142);
```
Burada da kod bloğunda yer alan "..." ifadesi ile belirtilen kısım önceki dumpda halihazırda bulunan değerlerdir. Görmüş olduğunuz gibi "**313**" numarası ilk kod bloğunda da vardı, bu bizim dosyamızın  id'sidir. Burda da "313" id'li dosyanın cache extended tablosuna ait bilgileri tutmaktadır.Sırasıyla tabloya iat parametreler şu şekildedir; 
1) fileid(dosyamıza ait id)
2) metadata_etag(meta bilgisi etiketi)
3) creation_time(oluşturulma zamanı, "0" olarak gözlemlediğimiz değer)
4) upload_time(yüklenme zamanı) 

olarak yer almaktadır.


Aynı zamanda "oc_file_locks" tablosu da etkilenmektedir, fakat henüz durumu çözebilmiş değilim.


# Dosya Silme(İlk silme işlemi)
Öncelikle veritabanımızın mevcut halinin mysql dump dosyasını çıkarıyoruz, sonrasında nextcloud'a gidip daha önce yüklemiş olduğumuz **"next.txt"** adlı dosyamızı siliyoruz. Silme işleminden sonra tekrar mysql dump dosyasını çıkarıyoruz ve Winmerge programı ile iki dosya arasındaki değişiklikleri karşılaştırıyoruz.
```sql
INSERT INTO 'oc_activity' VALUES ...,(33,1594204720,30,'file_deleted','hardrez','hardrez','files','deleted_self','[{\"313\":\"\\/next.txt\"}]','','[]','/next.txt','http://192.168.1.7/nextcloud/index.php/apps/files/?dir=/','files',313);
```
Belirtilen parametreler "Dosya Ekleme - oc_activity "'de belirttiğimiz paramatreler ile aynıdır. Burada yeni bir aktivite ekleniyor ve activite'ye ait type bilgisi **"file_deleted"** olarak gözlemliyoruz. Buradan dosyamızın silme isteğinde bulunduğumuzu ve sonuç olarak silme aksiyonunun gerçekleştiğini görebiliriz. Aynı tablodaki işlemden bahsettiğimiz için diğer parametreler aynı şekildedir.
```sql
INSERT INTO 'oc_filecache' VALUES ..., (313,2,'files_trashbin/files/next.txt.d1594204720','528420dc4662e5136ff3f92ee41b355d',278,'next.txt.d1594204720',15,3,10,1594128594,1594128594,0,0,'8d382065a9c651e9a3cd46cd6bc3a1ee',27,'');
```
Belirtilen parametreler "Dosya Eklemede - oc_filecache"'de belirttiğimiz parametreler ile aynıdır. Ardık dosyamızın path(dosya yolu) **"files_trashbin/files/next.txt.d1594204720"** çöp kutusunu göstermektedir. Aynı zamanda dosyamızın tam adı "next.txt" iken şuanda "next.txt.d1594204720" şeklinde değiştiğini de görüyoruz. Dosya silme işleminde adının tekrar formatlandığını da burdan çıkarabiliriz. Fakat hala dosyamız tam olarak silinmemiştir.
```sql
INSERT INTO `oc_files_trash` VALUES ..., (2,'next.txt','hardrez','1594204720','.',NULL,NULL);
```
Artık dosyamız silinenlerin tutulmuş olduğu tabloya INSERT ediliyor. Parametreler sırasıyla;
1) auto_id(otomatik verilen id )
2) id(dosyamızın adını tutuyor)
3) user(işlemi yapan kişi)
4) timestamp(zaman damgası)
5) location
6) type
7) mime

şeklindedir.
**NOT:** Dosyamız hala tam olarak silinmemiştir. Sadece silinenlerin bulunduğu çöp kutusuna eklenmiştir.
# Dosya Silme(Kalıcı olarak silme - ikinci silme)
Kalıcı olarak silme veya ikince silme olarak adlandırdığımız işlem dosyamızı çöp kutusundan da kalıcı olarak yaptığımız silme işlemidir. Ve bu silme işlemini gerçekleştirmeden öncesinde ve sonrasında mysql dump dosyalarını çıkartıp programımız ile karşılaştırıyoruz. Bu silme işlemini gerçekleştirdiğimizde artık "313" fileid'ye sahip dosya(next.txt dosyası) filecache ve filecache_extended tablolarında yer almaz. Fakat file_trash'de kalmaya devam eder. Silme işleminin öncesinde ve sonrasında değişen farklılıklar şu şekildedir;
```sql
INSERT INTO `oc_filecache_extended` VALUES ...;
```
şeklinde olur. Artık dediğimiz gibi bu tabloda "313" id'li dosyamızın bilgileri yer almaz. Bu yüzden bu tabloya ait veriler tabloya akratırırken dosyamızı göremeyiz.
# Dosya Geri Yükleme(Çöp kutusundan geri yükleme)
Senaryomuz şu şekilde, "pire.txt" adında bir dosyayı sisteme yükledik, sonrasında bu dosyayı sildik ve çöp kutusuna taşınmış oldu. Şimdi dosyayı geri yükle dediğimizde gerçekleşen işemleri inceliyoruz.
"oc_filecache" tablomuzda çok büyük bir değişiklik yoktur fakat dosyamız çöp kutusunda iken veri tabanında uzantısı değişmektedir. Dosyamız geri yüklendiğimizde  tablomuzda "parent", "name", "mimetype" ve "mimepart" parametrelerinde değişiklikler gözlemleriz.  Uzantısının nasıl değiştiğini sorguda görmek için "Dosya Silme(İlk silme işlemi)" başlığı altından inceleyebilirsiniz. Burada değişiklikten kast ettiğimiz durum örneğin "pire.txt" dosyası çöp kutusuna taşınmış olduğu durumda veri tabanında adı ve uzantısı "pire.txt.d1594212848"  şekilde değiştiği için haliyle uzantıyı, adını vb. gösteren parametreler de bu değişime bağlı olarak etkilenmektedir.

Geri alma işleminden **önce**(dosyamız çöp kutusunda iken);
```sql
INSERT INTO `oc_files_trash` VALUES ..., (4,'pire.txt','hardrez','1594212848','.',NULL,NULL);
```
şeklinde **"oc_files_trash"** tablosunda yer almaktaydı. Fakat dosyamızı geri yükleme işleminden sonra, dosyamız artık bu tabloda yer almaz.

Geri alma işleminden **sonra**;
```sql
INSERT INTO `oc_files_trash` VALUES ...;
```
(...) diğer dosyaları kapsamaktadır. Sorgumuzun son halinin bu şekilde değiştiğini görürüz.