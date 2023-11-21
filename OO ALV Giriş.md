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

```cadence
class cls_alv_oo IMPLEMENTATION.
  METHOD: get_data.
    SELECT * EROM mara
      INTO TABLE it_mara
      WHERE matnr in s_matnr.
   endmethod.
METHOD: set_fieldcat.
  data wa fcat TYPE lvc_s_fcat.
```

OO ALV ekrana basabilmek için de show metodunun içerisinde işlemler yapalım.
Tüm sütunlar üzerinde değişiklik yapacaksak eğer layout kullanalım .Layout ile sütunların bir satır açık renk bir satır daha koyu bir renk görünebilmesi için zebra diyerek ya x yada abap_true yazarak kullanılabilir.

Daha sonra is not bound , nesne referanslarına özeldir, o yüzden bu içeriği görünce otomatik olarak bir nesneden bahsedildiğini anlarız.
İs initial kullanırsak değişken internal tablo veya nesne referansı olabileceğinden dolayı  neden bahsettiğimiz konusunda belirsizlik vardır.
referans tipine sahip olanlar için  ve  nesneler için kullanılır,bu içeriği kullanırsak kod okurken bu ifadenin direk olarak referans nesnesi olduğunu anlarız.( vg_container TYPE REF TO cl_gui_custom_container,)
Vg_container boş değilse , obje oluşturalım.

Obje oluştururken noktaya basmadan önce ctrl space basılarak alanlar getir.Container name kısmana screen 100 de oluşturduğumuz container ismi verilir.
Obj alv grid den obje oluşturalım, patern kısmana container alır ama direk  olarak container ismini yazıp kullanamayız o yüzden ilk başta vg_container dan nesne oluşturup bu oluşturduğumuz nesneyi de pattern de ismini yazarız.

Ekranda görüntülemek için de obj alv grid set tab içeriğini kullanarak tanımladığımız layout u mara tablosundan oluşturduğumuz internal tabloyu ve fieldcatalog veririz.

Container içeriği boş ise de aynı işlemleri tekrarlamak için obj alv grid  referans içerik çağırılır.

```cadence
METHOD : SHOW ALV.
   DATA: VL_LAYOUT TYPE LVC_S_LAYO.
   VL LAYOUT-ZEBRA = 'X'.
   IF vg_container IS NOT BOUND.
      CREATE OBJECT VG CONTAINER
        EXPORTING
          CONTAINER NAME ='CC_ALV'.
      CREATE OBJECT OBJ ALV GRID
        EXPORTING
          I PARENT = VG_CONTAINER.

      CALL METHOD SET_FIELDCAT.

      CALL METHOD OBJ_AL_GRID->SET_TABLE_FOR_FIRST_DISPLAY
      EXPORTING
        IS LAYOUT = VL LAYOUT
        CHANGING
           IT OUTTAB       = IT MARA
           IT FIELDCATALOG = IT FCAT.
    ELSE.
       CALL METHOD OBJ_ALV_GRID->REFRESH_TABLE_DISPLAY.
    ENDIF.
ENDMETHOD
```
Daha sonra da main programın içerisine, start of selection yazarak  oluşturduğumuz methodları
Çağıralım .Get data ve show alv methodlarımızı çağıralım. En son da ekranımı çağırmak için call screen kullanalım.

```cadence
INCLUDE ZP_P095_I003. "TOP
INCLUDE ZP_P095_I001. "PBO
INCLUDE ZP_P095_I002. "PAI
INCLUDE ZP_P095_I004. "CLASS

START-OF-SELECTION.

CREATE OBJECT OBJ_ALV_OO.
call METHOD OBJ_ALV_OO->GET_DATA.
call METHOD OBJ_ALV_OO->SHOW ALV.

call screen 0100.
```

* 2.Yol eğer tablomuzdaki tüm sütunlar gelsin ve fieldcatalog üzerinde her satırı yazmamak için aşağıdaki şekilde kullanabiliriz.
Pattern ile lvc fieldcatalog merge çağıralım .Structure kısmını hangi tablo alanlarını kullanacaksak eğer o tablonun ismi yazılır.

```cadence
CALL FUNCTION 'LVC_FIELDCATALOG_MERGE'
  EXPORTING
      I_STRUCTURE_NAME      = 'MARA'
  CHANGING
      CT_FIELDCAT           = IT FCAT
  EXCEPTIONS
     INCONSISTENT INTERFACE = 1
     PROGRAM _ERROR         = 2
     OTHERS                 = 3
          .
IF SY-SUBRC <> 0.

ENDIF.
```
