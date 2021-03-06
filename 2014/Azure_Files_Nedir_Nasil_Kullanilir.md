---
FallbackID: 2912
Title: Azure Files Nedir? Nasıl Kullanılır?
PublishDate: 29/7/2014
EntryID: Azure_Files_Nedir_Nasil_Kullanilir
IsActive: True
Section: software
MinutesSpent: 0
Tags: Azure Storage Services, Windows Azure
---
Bundan yaklaşık iki sene önce [Azure
Drives](http://daron.yondem.com/software/post/Windows_Azure_Drive) ile
ilgili bir yazı yazmıştım. Özellikle migration veya on-prem (şirket içi)
legacy uygulamalar, sistemlerle cloud ortamındaki yapıları konuşturmak
için kullanılabilecek bir alt yapıydı Azure Drive. Neden geçmiş zamanlı
konuşuyorum? Çünkü **Azure Drive hizmeti 2015 yılı içerisinde
sonlandırılacak** ve onun yerine şu anda Preview olan **Azure Files**
gelecek. Durum bu olunca ben de erkenden Azure Files ile ilgili bir yazı
yazarak hem ne nedir sorusunu biraz cevaplayalım, hem de Azure Drive
kullandığınız yerleri nasıl Azure Files'a geçirirsiniz göz atalım
istedim.

Azure Drives servisi normal Storage Account'lar üzerinden veriliyor.
Hizmet şu anda Preview olduğu için ilk olarak [Preview
hizmetler](https://account.windowsazure.com/PreviewFeatures?fid=xsmb)
sayfasından aktif hale getirmeniz gerekecek. Hizmeti aktif hale
getirdikten sonra yeni bir Storage Account yaratmanız gerekiyor. Eski
hesaplara şu an için Azure Files otomatik olarak eklenmiyor.

### Nasıl kullanılır?

Azure Files esasen SMB 2.1 destekleyen bir Folder Share. Yani bildiğimiz
Folder Share. Azure Drive kullananlar hatırlayacaktır; Azure Drive
yazma-okuma anlamında sadece bir instance/VM'e ataçlanabiliyordu. Bu
durum Azure Files'da böyle değil :) İstediğiniz kadar makineye,
instance'a ataçlayabilirsiniz Folder Share'leri. Azure Drive ile en
büyük farklılıklarından biri zaten bu SMB 2.1 desteği. Bu protokol
desteği Azure Drive'ın ITPro'lar tarafından da rahatlıkla kullanılmasını
sağlıyor. Karşılaştıracak olursak Azure Drive aslında sadece
developerlara hitap eden bir araç olarak kalmıştı ve özünde çözmeye
çalıştığı sorunlar düşünüldüğünde ITPro'lar tarafından kullanılamıyor
olması aslında çok da akıllıca değildi. Azure Files ile bu sorun
çözülürken bu sefer de aklınıza, "Tamam da ben developer olarak REST
isterim!" haykırışı gelebilir :) Merak etmeyin, o da var. Zaten benim de
odaklanacağım taraf o olarak. ITPro tarafına bulaşma niyetim yok :)

REST API'lar üzerinden Azure Files'a ulaşmak için Storage Client'ın 4.0
ve üstü sürümlerinden birini kullanmanız gerekiyor. Ben bu yazıyı
yazarken [Storage Account
kütüphanesi](http://www.nuget.org/packages/WindowsAzure.Storage) zaten
4.2 sürümünde.  Gelin ufak bir proje yaratıp Nuget paketini ekleyelim
bakalım neler yapabiliyoruz.

**[C\#]**

```cs
CloudStorageAccount account = CloudStorageAccount.Parse(this.connString);
CloudFileClient client = account.CreateCloudFileClient();
CloudFileShare share = client.GetShareReference("birklasor");
share.CreateIfNotExistsAsync().Wait();
```

Storage Account'larla çalışanlar için yukarıdaki kod süper tanıdık
gözükecektir. Her zamanki StorageAccount objesi üzerinden normal Blob,
Table, Queue ile çalıştığımız zamanlardaki gibi bu sefer de
**CloudFileClient** alıyoruz. Bu client üzerinden var olan veya olmayan
bir ShareFolder'ın ismini verip referansını aldıktan sonra
**IfNotExist** ile Share Folder'ı oluşturuyoruz. Hepsi bu kadar aslında.
REST üzerinden Folder Share'i yaratmış olduk.

![Azure Files Storage Account Endpoint'lerinden
biri.](media/Azure_Files_Nedir_Nasil_Kullanilir/azurefiles_1.png)
*Azure Files Storage Account Endpoint'lerinden biri.*

Şimdi bu noktada birkaç uyarıda bulunmam gerek ve bunlar maalesef can
sıkıcı olacak. İlk olarak şu anda maalesef Azure Files desteği local
Storage Emülatöründe yok. O nedenle herşeyi Cloud ortamına karşı
çalıştırmak zorundasınız. Bir diğer sıkıntı ise SMB desteğinin sadece
storage accountun bulunduğu bölgede (region) verilmesi. Yani başka bir
datacenterdaki VM'i alıp da storage accounttaki Folder Share'i
bağlayamazsınız. Bu durum local bilgisayarınız için de geçerli :( Yani
yukarıdaki kod REST üzerinden gittiği için bilgisayarınızdan
çalıştırdığınızda sıkıntı olmayacaktır ama gidip de bu Folder Share'i
kendi bilgisayarınız map etmeye kalkarsanız maalesef sonuç hüsran
olacak. İşin o kısmını test etmek için benim tavsiyem Storage Account
ile aynı Region'da Visual Studio yüklü bir VM provision etmeniz. İtiraf
etmek gerekirse dev/testing açısından bu durum biraz kötü. En azından
local emülatör desteği gelirse süper olacak.

![Azure Files Folder Share VM'e
maplendi.](media/Azure_Files_Nedir_Nasil_Kullanilir/azurefiles_2.png)
*Azure Files Folder Share VM'e maplendi.*

Kod ile REST üzerinden Folder Share'i oluşturduktan sonra hemen test
için bir console açıp "net use" ile yukarıdaki ekran görüntüsünde de
görebileceğiniz üzere share'i VM'e mapledim. Böylece Azure Files
üzerinde bir Folder Share'imiz var artık. Buraya atılan tüm dosyalar,
dosya sistemi artık hem SMB hem de REST üzerinden ulaşılabilir durumda.
Test sonrası "**net use z: /delete**" ile mapi kaldırabilirsiniz.

### Web ve Worker Role'dan kullanımı?

Aslında yukarıdaki taktikten yola çıkarsak ne yapacağımı tahmin
edebilirsiniz sanırım. Net Use'ü doğrudan process olarak çalıştırarak
map işlemini Web veya Worker role içerisinde de yapabiliriz. Burada
önemli olan nokta WebRole.cs'teki RoleStart'ta yapmamak. Aslında bu
genel bir kural değil. Esas sorun Role'ünüz security context'ini
elevated hale getirirseniz ortaya çıkıyor. Malum Folder Share Map'leri
söz konusu user context içerisinde çalışıyor. RoleEnvironment ile
Application farklı security contextlerde çalışırsa doğal olarak
birbirlerinin oluşturduğu mapleri göremeyeceklerdir. Eğer Role'ün
security contexti yükseltirseniz Role System hesabı ile çalışırken
uygulamanız varsayılan kullanıcı hesap ile çalışır. Unutmayan Role
WaIISHost.Exe tarafından host edilirken uygulama w3wp.exe tarafından
host ediliyor. Bu gibi durumları engellemek için en iyisi mapleme
işlemini uygulama içerisinde yapmak. Eğer her iki tarafta da diske
ihtiyacınız varsa hem Role Start hem de App Start'da mapleme yapmanız
şart.

**[C\#]**

```cs
Process p = new Process();
int exitCode;
p.StartInfo.FileName = "net.exe";
p.StartInfo.Arguments = "use z: \\azurefilesdarontest.file.core.windows.net\birklasor 
                                        /u:azurefilesdarontest HhAItw2Q==";
p.StartInfo.CreateNoWindow = true;
p.StartInfo.UseShellExecute = false;
p.StartInfo.RedirectStandardError = true;
p.Start();
string error = p.StandardError.ReadToEnd();
p.WaitForExit(20000);
exitCode = p.ExitCode;
p.Close();

using (StreamWriter outfile = new StreamWriter(@"Z:\deneme.txt"))
{
    outfile.Write("Merhaba Dünya!");
}
```

Map işlemi bittikten sonra gördüğünüz üzere normal System.IO altındaki
her şeyi bu diskte kullanabiliyorum. Artık birden çok instance, role
tarafından erişilebilecek bir folder share'imiz var :) Süper! Örneğin bu
diski IAAS tarafından alınmış bir VM'e mapleyip, PAAS tarafındaki role
instancelarınıza da mapleyip IAAS ortamına kurulu stand-alone bir
uygulama ile PAAS taki uygulamanızın birbiri ile basit bir folder share
üzerinden konuşmasını sağlayabilirsiniz. Bu senaryo özellikle kodu ve
mimarisi sizin yönetiminizde olmayan uygulamalarla entegrasyon için çok
anlamlı olabiliyor.

### REST üzerinden disk erişimi

SMB haricinde REST desteğinin de olduğundan bahsetmiştik fakat bu
noktaya kadar REST üzerinden Share Folder yaratmak dışında bir şey
yapmadık. REST üzerinden doğrudak diskteki tüm dosya ve klasörlere full
erişim hakkınız da var. Bu seçenek özellikle uygulamanız ayrı bölge
(region) içerisinde değilse süper işe yarayacaktır. Malum SMB için aynı
region içerisinde olmanız şart.

**[C\#]**

```cs
CloudStorageAccount account = CloudStorageAccount.Parse(this.connString);
CloudFileClient client = account.CreateCloudFileClient();
CloudFileShare share = client.GetShareReference("birklasor");
CloudFileDirectory rootDirectory = share.GetRootDirectoryReference();
CloudFile aCloudFile = rootDirectory.GetFileReference("deneme.txt");
Response.Write(aCloudFile.DownloadText());
```

Yukarıdaki kod içerisinde CloudFileShare'den root klasörü istedikten
sonra artık iş **CloudFileDirectory** ve **CloudFile** objeleri ile
çalışmaya kalıyor. Tüm buradaki objeler üzerinde yaptığınız işlemler
doğrudan REST API'lar üzerinden gerçekleştiriliyor. O nedenle bu şekilde
Azure Files'a erişen bir uygulamanınz SMB protokolüne bağımlılığı yok.

### Blob mu File Share mi?

İtiraf etmem gerek eğer bu soruyu soruyorsanız kafanız epey karışmış
demektir :) Ama bu çok da normal. Birincisi Azure Files'da SMB var. Bu
aslında servisin çözmeye çalıştığı sorun adına çok şey anlatıyor.
Buradaki amaç sadece bir storage sistemi sağlamak değil, hatta storage
sistemi sağlamak da değil. Buradaki amaç Bloblar gibi inanılmaz
ölçeklenebilirliliğe sahip storage yapılarını kullanamayan, migrate
edilemeyen senaryolarda bir çözüm sunabilmek. Daha açıkçası,
uygulamaların Cloud'a taşınmasını kolaylaştırmak ve (IAAS/PAAS) hybrid
uygulama yapılarını kolaylaştırmak. Teknik karşılaştırma isterseniz
eğer; bloblarda bir container 500TB olabilirken burada bir Share azami
5TB olabiliyor. Erişim olarak 60MBps storage'da blob başına gelirken
Azure Files'da Share başına geliyor. Diğer yandan bloblarda esasen
Directory diye bir konsept yokken burada var. Bloblarda isimler büyük,
küçük harf duyarlıyken burada değil. Bloblarda snapshow var, burada yok
:) Son olarak bloblarda byte başına para ödeniyor burada toplam size
için.

### Azure Disk ile Azure Files farkı nedir?

Azure Disk'i hatırlamayanlar için hemen ip ucu veriyim; bir VM
yarattığınızda ona ataçladığınız disk bir Azure Disk. Azure Disk'ler tek
bir VM tarafından kullanılabiliyorlar. Azure Files ile en büyük
farklılıkları aslında bu. Diğer yandan REST üzerinden erişim şansınız da
yok ama Azure Disk'ler aslında Blob'lar üzerinde yaşadığı için Snapshot
özelliği var. Azure Disk'lerde en büyük disk boyutu 1TB, Azure Files'da
Share boyutu 5TB. Veri erişimi hızı aynı olsa da 8KB IOps'da Azure Disk
500 verirken Azure Files 1000 veriyor.

Son olarak, yukarıdaki kodları hemen alıp test etmek isterseniz makaleyi
yazarken kullandığım test kodlarını
[Github'a](https://github.com/daronyondem/AzureOrnekler/tree/master/AzureFiles_Preview)
attım.

Görüşmek üzere.


