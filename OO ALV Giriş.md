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


```cadence
TABLES : MARA
SELECTION-SCREEN BEGIN OF BLOCK B1
  SELECT-OPTIONS S_MATNR FOR MARA-MATNR
  SELECTION -SCREEN END OF BlOCK B1.
  
  DATA: it_mara TYPE TABLE OF mara,
        it_fcat TYPE STANDARD TABLE OF lvc_s_fcat,
        vg_container TYPE REF TO cl_gui_custom_container,
        obj_alv_grid TYPE REF to Cl_GUI_ALV_GRID.
        
TYPES: BEGIN OF ty_mara,
       metarial TYPE MARA-MATNR,
       spart TYPE MARA-SPART,
       maktl TYPE MARA-MATKL,
   END OF ty_ mara.
   
DATA: it_mars TYPE TABLE OF ty mara.

class cls_alv oo DEFINITION DEFERRED.
  DATA: obj alv oo TYPE REF TO cls_alv_oo.
```

Screen oluşturalım daha sonrada sırasıyla aşağıda ki işlemleri yapalım. Custom Control oluşturalım,Kontrolleri görüntüleyebileceğiniz ekran alanları. 

![image](https://github.com/sumeyyaakbulut/ABAP-OO-ALV/assets/62395974/1f23b153-83cd-4863-bc55-4041640225c5)

İkinci include PBO Process before output ekran açılmadan başlık ve status oluşması gerekmektedir.

![image](https://github.com/sumeyyaakbulut/ABAP-OO-ALV/assets/62395974/80cbecdf-2fd8-45e1-b136-9075c03b09bd)

Üçüncü include ise PAI process after input ekran açıldıktan sonra yapılan işlemleri gösterir.
![image](https://github.com/sumeyyaakbulut/ABAP-OO-ALV/assets/62395974/0155ef57-0bd7-4e50-b836-4c162354805e)

Dördüncü include içeriği için classlar kullanalım. İlk olarak cls_alv_oo ismin de local
Class oluşturalım. Definition kısmında public tanımlı üç method kullanalım.

![image](https://github.com/sumeyyaakbulut/ABAP-OO-ALV/assets/62395974/118b75b1-4583-4616-a459-9d7d39273a76)

Implementation yani class işlevsellik kazandırdığımız yer de method sırayla işlemleri yapalım.
Get data metodumuzun  içeriğine selection option ile kullanıcının girdiği değere matnr alanına eşit olan değerleri getirelim.
FieldCatalog ile structure oluşturalım, Fieldname ile kullanılan sütunun ismi yazılır.
Outputlen ile sütunun uzunluğu belirlenir.
SELTEXT değeri araç ipucu olarak da kullanılacak ve fareyi ızgaradaki sütunun üzerine getirdiğinizde görünecektir.
REF TABLE şeffaf tablo içindeki alanlara başvurabilirsiniz. Bu durum da
mara tablosunu ve bu alanı referans olarak almasını söylüyorum.
Bu şekilde hangi tablodaki hangi sütunların görünmesini istiyorsak fieldcatalog un structure üzerinde işlemler yapıp append ile de fieldcatalog internal tablosuna kaydedelim.

class cls_alv_oo IMPLEMENTATION.
  METHOD: get_data.
    SELECT * EROM mara
      into TABLE it_mara
      where matnr in s_matnr
   endmethod
METHOD: set fieldcat
data wa fcat TYPE lvc s fcat



