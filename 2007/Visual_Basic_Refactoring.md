---
FallbackID: 1899
Title: Visual Basic Refactoring
PublishDate: 28/12/2007
EntryID: Visual_Basic_Refactoring
IsActive: True
Section: software
MinutesSpent: 0
Tags: ASP.NET 3.5, Visual Basic 2005, Visual Basic 2008, Visual Studio 2005, Visual Studio 2008, Visual Basic .NET
old.EntryID: 89a98349-9c27-460b-91a3-744e0725897f
---
Bugün sizlere biraz **Refactoring'den** bahsetmek isriyorum.
Refactoring'i kaba bir şekilde tanımlamak gerekirse herhangi bir kodun
işlevini değiştirmeden yazılışı değiştirmek ve hedef olarak da kodun
okunuşu kolaylaştırmaktır diyebiliriz. Refactoring konusunda sektörde
bir çok araç, üçüncü parti uygulamalar satılıyor. Visual Studio
içerisinde de C\# için hazır bazı ufak tefek Refactoring araçları
bulunuyor. Tabi ben bir VB programcısı olarak olayın VB kısmından
bahsedeceğim ve sizlere ücretsiz bir Visual Studio eklentisi olan
Refactor'u tavsiye edeceğim.

<http://msdn2.microsoft.com/en-us/vbasic/bb693327.aspx>

Yukarıdaki adresten indirebileceğiniz yazılımın normal sürümü ücretsiz
ve hem Visual Studio 2005 hem de 2008 ile uyumlu. Yazılımın daha çok
özelliklere sahip bir sürümü de "Refactor Pro" adı altında satılıyor.
Biz şimdilik ücretsiz sürümle yetinelim :)

Hemen bir iki örnek ile neler yapabileceğimize bakalım.

    <span style="color: blue;">Protected</span> <span
style="color: blue;">Sub</span> Page\_Load(<span
style="color: blue;">ByVal</span> sender <span
style="color: blue;">As</span> <span style="color: blue;">Object</span>,
<span style="color: blue;">ByVal</span> e <span
style="color: blue;">As</span> System.EventArgs) <span
style="color: blue;">Handles</span> <span
style="color: blue;">Me</span>.Load

        <span style="color: blue;">Dim</span> ds <span
style="color: blue;">As</span> <span style="color: blue;">New</span>
Data.DataSet

 

        <span style="color: blue;">Using</span> cnn <span
style="color: blue;">As</span> <span style="color: blue;">New</span>
SqlConnection(ConfigurationManager.ConnectionStrings(<span
style="color: #a31515;">"CNN"</span>).ConnectionString)

            <span style="color: blue;">Using</span> cmd <span
style="color: blue;">As</span> <span style="color: blue;">New</span>
SqlCommand(<span style="color: #a31515;">"SELECT \* from TABLO"</span>,
cnn)

                <span style="color: blue;">Dim</span> da <span
style="color: blue;">As</span> <span style="color: blue;">New</span>
SqlDataAdapter(cmd)

                da.Fill(ds)

            <span style="color: blue;">End</span> <span
style="color: blue;">Using</span>

        <span style="color: blue;">End</span> <span
style="color: blue;">Using</span>

 

        GridView1.DataSource = ds

        GridView1.DataBind()

    <span style="color: blue;">End</span> <span
style="color: blue;">Sub</span>

Yukarıdaki kod herhangi bir asp.net web sayfasının **Page.Load**
durumunda çalışıyor olsun. Gördüğünüz gibi veritabanına bir **Select**
göndererek gelen veriyi **GridView'e** bağlıyoruz. Eğer **Select** ile
aldığımız veriyi sayfada başka yerlerde de almamız gerekecekse aslında
bunu harici bir **function** olarak yazmamız daha faydalı olacaktır.
Hemen aşağıdaki şekilde **function** içerisine almak istediğim bölümü
seçiyorum.

![Kodumuzu harici bir function içerisinde
alıyoruz.](media/Visual_Basic_Refactoring/27122007_1.png)\
*Kodumuzu harici bir function içerisinde alıyoruz.*

"**Extract Method**" komutu verdiğimizde Refactor bize kodu **class**
yapısı içerisinde nereye yazdırmak istediğimizi soruyor sonrasında da
Function'ımızı aşağıdaki gibi otomatik olarak yaratıyor.

<span style="color: blue;">Partial</span> <span
style="color: blue;">Class</span> \_Default

    <span style="color: blue;">Inherits</span> System.Web.UI.Page

 

    <span style="color: blue;">Private</span> <span
style="color: blue;">Shared</span> <span style="color: blue;">Sub</span>
**Page\_LoadExtracted**(<span style="color: blue;">ByVal</span> ds <span
style="color: blue;">As</span> Data.DataSet)

        <span style="color: blue;">Using</span> cnn <span
style="color: blue;">As</span> <span style="color: blue;">New</span>
SqlConnection(ConfigurationManager.ConnectionStrings(<span
style="color: #a31515;">"CNN"</span>).ConnectionString)

            <span style="color: blue;">Using</span> cmd <span
style="color: blue;">As</span> <span style="color: blue;">New</span>
SqlCommand(<span style="color: #a31515;">"SELECT \* from TABLO"</span>,
cnn)

                <span style="color: blue;">Dim</span> da <span
style="color: blue;">As</span> <span style="color: blue;">New</span>
SqlDataAdapter(cmd)

                da.Fill(ds)

            <span style="color: blue;">End</span> <span
style="color: blue;">Using</span>

        <span style="color: blue;">End</span> <span
style="color: blue;">Using</span>

    <span style="color: blue;">End</span> <span
style="color: blue;">Sub</span>

    <span style="color: blue;">Protected</span> <span
style="color: blue;">Sub</span> Page\_Load(<span
style="color: blue;">ByVal</span> sender <span
style="color: blue;">As</span> <span style="color: blue;">Object</span>,
<span style="color: blue;">ByVal</span> e <span
style="color: blue;">As</span> System.EventArgs) <span
style="color: blue;">Handles</span> <span
style="color: blue;">Me</span>.Load

        <span style="color: blue;">Dim</span> ds <span
style="color: blue;">As</span> <span style="color: blue;">New</span>
Data.DataSet

 

        **Page\_LoadExtracted**(ds)

 

        GridView1.DataSource = ds

        GridView1.DataBind()

    <span style="color: blue;">End</span> <span
style="color: blue;">Sub</span>

<span style="color: blue;">End</span> <span
style="color: blue;">Class</span>

Gördüğünüz gibi artık aynı **Function'ı** kullanarak sürekli aynı datayı
istediğimizde bir **DataSet** içerisine doldurtabiliyoruz. Fakat burada
sizi rahatsız edeceğinden emin olduğum bir şey var; Refactor'un
yarattığı function'ın adı çok anlamsız :) Visual Studio içerisinde
yaratılan fonksiyonun adını değiştirirseniz **Refactor** sizin için söz
konusu fonksiyonun kullanıldığı yerlerdeki referansları da otomatik
olarak değiştirecektir.

Gelelim bir diğer örneğe. Varsayalım ki yeni bir sınıf yapısı
oluşturmaya başladınız. Oluşturduğunuz ilk sınıfın bir sürü **Private**
değişkeni var ve bunlara da uygun **Property'lerin** tanımlanması
gerekiyor. **Set** ve **Get** leri tek tek her biri için ayarlıyor olmak
gerçekten işkenceye dönüşebilir. Gelin aşağıdaki koda bir bakalım.

    <span style="color: blue;">Public</span> <span
style="color: blue;">Class</span> Deneme

        <span style="color: blue;">Private</span> Ozellik <span
style="color: blue;">As</span> <span style="color: blue;">Integer</span>

        <span style="color: blue;">Private</span> Ozellik2 <span
style="color: blue;">As</span> <span style="color: blue;">Integer</span>

 

        <span style="color: blue;">Sub</span> <span
style="color: blue;">New</span>()

 

        <span style="color: blue;">End</span> <span
style="color: blue;">Sub</span>

 

        <span style="color: blue;">Sub</span> <span
style="color: blue;">New</span>(<span style="color: blue;">ByVal</span>
Ozellik <span style="color: blue;">As</span> <span
style="color: blue;">Integer</span>, <span
style="color: blue;">ByVal</span> Ozellik2 <span
style="color: blue;">As</span> <span
style="color: blue;">Integer</span>)

            <span style="color: blue;">Me</span>.Ozellik = Ozellik

            <span style="color: blue;">Me</span>.Ozellik2 = Ozellik2

        <span style="color: blue;">End</span> <span
style="color: blue;">Sub</span>

    <span style="color: blue;">End</span> <span
style="color: blue;">Class</span>

Yukarıdaki sınıfımızda sadece iki adet özellik bulunuyor bunların
**Property'lerini** oluşturmamız gerekiyor. **Private** değişkenlerin
üzerine sağ tuşu ile tıkladıktan sonra "**Refactor / Encapsulate
Field**" komutu veriyorum ve kodun yerleştirileceği yeri de seçtikten
sonra Refactor benim için **Property** kodlarını otomatik olarak
yazıyor.

  <span style="color: blue;">Public</span> <span
style="color: blue;">Class</span> Deneme

        <span style="color: blue;">Private</span> Ozellik <span
style="color: blue;">As</span> <span style="color: blue;">Integer</span>

        <span style="color: blue;">Private</span> Ozellik2 <span
style="color: blue;">As</span> <span style="color: blue;">Integer</span>

 

        <span style="color: blue;">Sub</span> <span
style="color: blue;">New</span>()

 

        <span style="color: blue;">End</span> <span
style="color: blue;">Sub</span>

 

        <span style="color: blue;">Sub</span> <span
style="color: blue;">New</span>(<span style="color: blue;">ByVal</span>
Ozellik <span style="color: blue;">As</span> <span
style="color: blue;">Integer</span>, <span
style="color: blue;">ByVal</span> Ozellik2 <span
style="color: blue;">As</span> <span
style="color: blue;">Integer</span>)

            <span style="color: blue;">Me</span>.Ozellik1 = Ozellik

            <span style="color: blue;">Me</span>.Ozellik21 = Ozellik2

        <span style="color: blue;">End</span> <span
style="color: blue;">Sub</span>

        <span style="color: blue;">Public</span> <span
style="color: blue;">Property</span> Ozellik1() <span
style="color: blue;">As</span> <span style="color: blue;">Integer</span>

            <span style="color: blue;">Get</span>

                <span style="color: blue;">Return</span> Ozellik

            <span style="color: blue;">End</span> <span
style="color: blue;">Get</span>

            <span style="color: blue;">Set</span>(<span
style="color: blue;">ByVal</span> value <span
style="color: blue;">As</span> <span
style="color: blue;">Integer</span>)

                Ozellik = value

            <span style="color: blue;">End</span> <span
style="color: blue;">Set</span>

        <span style="color: blue;">End</span> <span
style="color: blue;">Property</span>

        <span style="color: blue;">Public</span> <span
style="color: blue;">Property</span> Ozellik21() <span
style="color: blue;">As</span> <span style="color: blue;">Integer</span>

            <span style="color: blue;">Get</span>

                <span style="color: blue;">Return</span> Ozellik2

            <span style="color: blue;">End</span> <span
style="color: blue;">Get</span>

            <span style="color: blue;">Set</span>(<span
style="color: blue;">ByVal</span> value <span
style="color: blue;">As</span> <span
style="color: blue;">Integer</span>)

                Ozellik2 = value

            <span style="color: blue;">End</span> <span
style="color: blue;">Set</span>

        <span style="color: blue;">End</span> <span
style="color: blue;">Property</span>

    <span style="color: blue;">End</span> <span
style="color: blue;">Class</span>

Refactor tarafından tamamlanan yukarıdaki kodda dikkatinizi çektiyse
**Sub New** kodundaki değerlerin aktarıldığı değişkenlerin isimleri de
otomatik olarak **Property** isimleri ile değiştirilmiş durumda. Bir
önceki örnekte olduğu gibi burada da eğer elle **Property** isimlerini
değiştirirseniz Refactor otomatik olarak sınıf içerisinde diğer
referansları da değiştiriyor.

Tüm bu işlemleri kod yazarken yapıyor olmak kolaylık sağlayabilir fakat
esas mesele elinizde hazır kodları tamamlanmış bir proje varsa
gerçekleşiyor. Projeyi inceleyerek sadece **Refactoring** araçlarını
kullanarak daha okunabilir bir kod yaratmaya çalışıyorsunuz, hatta çoğu
zaman kodun işleyişini değiştirmeden kodu kısaltabiliyorsunuz bile.

Refactoring'e giriş yapmanızı sağlayacak Refactor eklentisi ne kadar
ücretsiz olsa da daha fazla reklamını yapmayacağım :) **Refactoring**
dünyasını keşfetmek artık size kalmış.

Hepinize kolay gelsin.


