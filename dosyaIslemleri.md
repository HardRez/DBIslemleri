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
şeklinde olur. Artık dediğimiz gibi bu tabloda "313" id'li dosyamızın bilgileri yer almaz. Bu yüzden bu tabloya ait veriler tabloya akratırırken dosyamıza ait verileri göremeyiz.
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


# Dosyayı Favorilere Kaydetme
Senaryomuzda yine öncesinde yüklemiş olduğumuz "pire.txt" adında bir dosyamız var. Bu dosyayı favorilere aldığımızda veri tabanında yapılan işlemleri inceleyeceğiz. İşlemi yapmadan ve yaptıktan sonra mysql'in dump dosyalarını karşılaştırıyoruz.
Bu yaptığımız işlemde sonuçta bir **activity** olduğu "oc_activity" tablosuna yeni bir girdimiz ekleniyor.
```sql
INSERT INTO `oc_activity` VALUES ..., (68,1594270741,30,'favorite','hardrez','hardrez','files','added_favorite','{\"id\":327,\"path\":\"pire.txt\"}','','[]','pire.txt','','files',327);
```
"oc_activity" tablosunda yer alan parametrelerden yazımızın en başında bahsedilmişti. Ordan bakılabilir. Sorgumuzdan da basitçe baktığımızde bir "pire.txt" dosyamız  ve subject parametremiz 'added_favorite'  şeklinde. Burda **type** parametremizin 'favorite' olduğunu da görüyoruz. Yani biz bir dosyayı favorilere eklediğimizde "oc_activity" tablosunda bu eklemeler gerçekleşiyor.

```sql
INSERT INTO `oc_vcategory` VALUES (1,'hardrez','files','_$!<Favorite>!$_');
```
Bir dosyayı favorilere eklediğimizde aslında basitçe düşünürsek bir kategorize etme işlemi gerçekleştiriyoruz. "oc_vcategory" tablomuzun parametlerini inclediğimizde parametreler şu şekildedir;
1) id
2) uid(kullanıcı id / user id)
3) type (türü bknz. file, picture...)
4) category

Sorgumuzu incelicek olursak biz sistemdeki bir dosyayı favori olarak kaydettiğimizde "oc_vcategory" tablosunda otomatik olarak "1" id'ye sahip(auto increment), "hardrez" adlı kullanıcı tarafından oluşturulmuş, "files" dosya tiplerini içeren, "_$!<Favorite>!$_" kategorisini oluşturuyor. Aslında ilk başta böyle bir kategori yok, ne zaman biz bir dosyayı favorilere kaydettik, o zaman sistemde otomatik olarak bu sorgu ile favoriler adında bir kategori oluşuyor ve eklediğimiz dosya türüne bağlı olarak  hangi tipde içerik tuttuğunu gösteriyor.


```sql
INSERT INTO `oc_vcategory_to_object` VALUES (327,1,'files');
```
Yine baktığımızda yukarıdaki sorgumuza benzediğini söyleyebiliriz. Öncesinde tablomuza parametrelerimizi inceleyliyoruz.
1) objid(nesnemizin/dosyamızın id'si)
2) categoryid(kategori id)
3) type(tipi)

Değişkenlerden de gördüğümüz üzere "327" bizim dosyamızın id'sini belirtiyor. Yani hangi bu "objid"'ye sahip nesne bizim dosyamız. "categoryid" değerimiz "1" idi, "oc_vcategory" tablosunu incelediğimizde bu id'ye karşılık gelen kategorinin **"_$!<Favorite>!$_"** olduğunu gözlemliyoruz. "type" değişkeninde parametre olarak "files" girilmiş. Yani sistemdeki bu nesnenin "files" tipinden olduğunu gösteriyor. Sonuç olarak sorguyu Türkçeleştiricek olursak, "327" objidye sahip,  "1" kategori id'li, "files" tipindeki nesne tabloya ekleniyor demek istenmektedir.


# Dosyayı Favorilerden Kaldırma
Favorilere kaydetmede "pire.txt" adlı bir dosyamızı favori olarak kaydetmiştik. Şimdi ise favorilerden kaldırıp yine WinMerge programımız ile gerçekleşen sorguları inceliyicez. Sitede yaptığımız hemen hemen her işlem bir aktivite olarak geçtiği için "oc_activity" ve onun dışında "oc_vcategory_to_object" tablolarımızda değişiklikler olmaktadır. Favori eklemede "oc_vcategory" tablomuz da etkilenmişti, çünkü hiç favori eklenmediği, başlangıç durumunda(herhangi bir kategori yok iken) "oc_vcategory" tablosunun boş olduğunu gözlemledik. Ne zaman ki biz sistemde bir kategorize etme işlemi uyguladık, yani bir dosyayı favorilere ekledik o zaman "oc_vcategory" tablosuna **"_$!<Favorite>!$_"** kategori adında bir instert işlemi gerçekleşti. Yani aslında bizim en başta sistemde **"_$!<Favorite>!$_"** adında bir kategorimiz yoktu. İşlemin detayları için "Dosyayı Favorilere Ekleme"'de yer almaktadır.
Şimdi bir dosyayı favorilerden kaldırdığımızdaki ve kaldırmadan önceki farkı inceleyebiliriz.
```sql
INSERT INTO `oc_activity` VALUES ..., (71,1594293477,30,'favorite','hardrez','hardrez','files','removed_favorite','{\"id\":327,\"path\":\"pire.txt\"}','','[]','pire.txt','','files',327);
```
Dediğimiz üzere favorilerden kaldırma işlemi de bir aksiyon olduğu için burada veritabanına "hardrez" adlı kullanıcı tarafından gerçekleşen "files" tipindeki, "327" objid'ye sahip, "pire.txt" dosyasının, "removed_favorite" başlığı altında yapılması gereken işlem insert edilmiştir. Ve arkaplanda sistem tarafından favorilerden kaldırılması için gerekli sorgu basılmaktadır. Bu tabloya ait parametreleri incelemek için ilk konuya çıkabilirsiniz.
```sql
INSERT INTO `oc_vcategory_to_object` VALUES (14,1,'files');
```
Göründüğü üzere kategorilere hala 1 adet dosya insert ediliyor. Fakat "objid"'sine bakıcak olursak bu dosyanın bizim "pire.txt" dosyası olmadığını görebiliriz. Çünkü "pire.txt" dosyasını sisteme eklediğimizde "objid" parametresinin "327" olduğunu gözlemlemiştik. Yani bir dosyayı favorilerden kaldırdığımızda  artık kategorilere ait dosyaların yer aldığı "oc_vcategory_to_object" tablosunda "pire.txt" dosyamızın bilgilerinin yer almadığını görebiliriz. Farkı daha iyi gözlemlemek adına aşağıya tekrar, dosyamızın favorilere eklendiği andaki sorguyu yazıyorum.
```sql
INSERT INTO `oc_vcategory_to_object` VALUES (14,1,'files'),(327,1,'files');
```
**Önemli NOT:** "14" objid'ye sahip dosyamız şuan için anlatımla alakası yoktur, o id'ye sahip dosya çalışmalar dışında farklı testler için favorilere eklenmiştir ve önceki "Dosyayı Favorilere Ekleme" konusunda yer almamaktadır.


# Dosyayı Yeniden Adlandırma
Önceki senaryolarda sisteme yüklemiş olduğumuz "327" objid'li "pire.txt" dosyası üzerinden gitmeye devam ediyoruz. "pire.txt" adlı dosyamızın adını "degistirilmisPire.txt" şekilnde yeniden adlandırarak veri tabanında gerçekleşen sorguları inceleyecez. Burda bir işlem söz konusu olduğundan yine "oc_activity" tablomuz enbaşta olmak üzere, beraberinde "oc_filecache" tablosu da dolaylı olarak etkilenmektedir. İlk sorgumuzu inceleyelim.
```sql
INSERT INTO `oc_activity` VALUES ..., (79,1594298461,30,'file_changed','hardrez','hardrez','files','renamed_self','[{\"327\":\"\\/\\/degistirilmisPire.txt\"},{\"327\":\"\\/\\/pire.txt\"}]','','[]','//degistirilmisPire.txt','http://192.168.1.2/nextcloud/index.php/apps/files/?dir=/','files',327);
```
Bir dosyanın ismini değiştirme isteğini yolladığımızda sistem tarafında, "file_changed" parametresinde yapılması gereken düzenleme tipi tabloya uygun bilgilerle doldurulmuş ve sorgu basılmıştır. Burda basılan sorguda "327" objid'ye sahip olan dosyanın isminin yeniden adlandırılma aksiyonunun sisteme eklendiğini gözlemliyoruz. Parametrelerin detayları için "Dosya Ekleme" konusuna gidebilirsiniz. Sorgumuzda dosyanın eski ve yeni adını da gözlemliyebiliyoruz.
```sql
INSERT INTO `oc_filecache` VALUES ..., (327,2,'files/degistirilmisPire.txt','f14efdbe76ec33b8514b08e93338252c',6,'degistirilmisPire.txt',20,6,105,1594212819,1594212819,0,0,'966aac7b9f57ad17177dd0624a33e5d8',27,''), ...;
```
Mysql dump dosyalarımızı karşılaştırdığımızda buradaki tek değişen farkın dosya adı olduğunu gözlemliyoruz. Burada dikkat çeken bir husus bulunmaktadır, biz dosyamızın adını değiştirsek de "mtime" ve "storage_mtime" parametrelerinin önceki ve sonraki durumlarda **değişmediğini** gözlemliyoruz. Sistem eğer dosyanın içeriğinde herhangi bir değişiklik olursa bu zaman damgalarını güncellemektedir. Dosyanın adındaki değişiklikler bu parametreleri **etkilememektedir**. İsim değiştirmeden **önceki** sorguya da aşağıdan bakarak inceleyebilirsiniz.
```sql
INSERT INTO `oc_filecache` VALUES ..., (327,2,'files/pire.txt','40d5b374e7892f0775f505d91845a0e4',6,'pire.txt',20,6,105,1594212819,1594212819,0,0,'966aac7b9f57ad17177dd0624a33e5d8',27,''), ...;
```
Söylediklerimize ek olarak "path_hash" parametresinin de değiştiğini görüyoruz, değişmesinin sebebi dosya yolunda dosyanın da adı yer aldığı için dosya adındaki veya dosya yolundaki değişiklikler haliyle "path_hash" parametresini de etkilemektedir.