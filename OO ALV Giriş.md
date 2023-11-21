## OO ALV
OO ALV Screen ,Fieldcatalog ve Internal tablo üçünü de kendimizin oluşturması gerekmektedir.

İlk olarak include oluşturalım. 
Include; 
* 1.Veri tanımlanır, 
* 2.PBO(Process before output),
* 3.PAI(Process After Input)
* 4.Class ,bilgilerini tutan dört tane include oluşturalım.

1.Include
Select Option kullanacağımız tablo seçim yapmak için kullanalım. Select option kullanılacak olan tabloyu kullanabilmek için tables ile üst kısma yazmalıyız .Selection option dışına selection screen begin blok oluşturalım.

Kullanacağımız mara tablosundan internal tablo oluşturalım.
Hem tablo türü hem de standart tablo türü dahili tablo oluşturmak için type Standard table of kullanarak fieldcat internal tablo oluşturalım.

Class dan referans alınarak container oluşturulur.
Ekrana basabilmek için class type referans alarak  oluşturalım. 

Type begin of ile tip oluşturalım. Oluşturduğumuz bu tipi kullanarak internal tablo oluşturalım. 


