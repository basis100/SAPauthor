 SAPauthor
# BASIS终极授权の内部顾问参数文件的产生
需求：
我们需要达到一个授权目的： 


1 .让内部顾问有全部的权限对象。

2. 让内部顾问在10万个事物代码中，剔除“用户管理和系统管理”的几十个事务代码，其它都可以使用。




解决办法：

一、SAP_ALL是系统标准的全部权限的参数文件，在ECC6 EHP7新系统中由17个参数文件构成了。
T_CODES对象包含在&_SAP_ALL_16参数文件中。


二、参看&_SAP_ALL_16参数文件，有100多个对象，T_CODES对象也包含在其中。

新建一个角色ZSAP_ALL_16.SAP，使用这100多个对象。
其中对于T_CODES对象，里面放的就是全部的事物代码清单。	


三、拿掉不需要的事物代码PFCG，SPRO，SU01分段，比如T_CODES对象的值为：
/ASU/MAINTAIN ---PFAL  【排除位置39187 PFCG】
PFCP---------SPRM  	【排除位置44464 SPRO】
SPROJECT---------SU0  【排除44778 SU01】
SU3----------_IHC00


如果要排除的事物代码太多，为得到这个分段，可以用程序来实现。程序见最后。


四、最后我们产生了用于内部顾问的参数文件：ZNOBASPRO。
	这个参数文件加给用户，就实现了我们的需求。












五、历次版本更新调整记录：

1、2014.12.02-PRD生产系统--ZSAP_ALL_16角色，拷贝出一个ZSAP_ALL_16-2角色。

2、拿掉下列代码的的分段，只要是去掉开发和用户管理事物代码。

3、调整后保存对应的参数文件名称T-PD090095。

4、用该参数去替换原来在复合参数文件中的T-DV320002。



SCC1
SCC4
SCC7
SCC8
SCC9
SPRO
PFCG
 
SU01 
SU01D 
SU01_NAV 
SU01_OLD 
SU02 
SU03 
SU05_OLD 
SU1 
SU10 
SU10_OLD 
SU12 
SU2 
SU20 
SU20_BTCH 
SU21 
SU22 
SU22_HISTORY 
SU22_OLD 
SU24 
SU24_HISTORY 
SU24_OLD 
SU24_S_TABU_NAM 
SU25 
SU25_2A_SEL 
SU25_OLD 
SU26


SE09
SE10
SE11
SE15
SE16
SE16N
SE18
SE19
SE20
SE21
SE24
SE30
SE32
SE37
SE38
SE39
SE80








//---2014.7.25-PRD生产系统--ZSAP_ALL_16角色，对应的参数文件：ZNOBASPRO--业务A角，无basis权限,无spro等76个TC----

ZNOBASPRO参数文件适用的集团:300,220
除掉下面事物代码（共60个）,除掉开发代码（共16个）：


ME12 
ME13 
ME14 
ME15 
ME1A 
ME1E 
ME1L 
ME1M 
ME1P 
ME1W 
ME1X 
ME1Y 
ME22N 
ME23N 
ME28 
ME29N 
ME2B 
ME2C 
ME2L 
ME2M 
ME2N 
ME2W 
ME80FN 
ME81N 
MEMASSIN 
MES11 
MM02 
MM03 
MM60 
PFCG 
SE11 
SE16 
SE16N 
SPRO 
SU01 
SU01D 
SU01_NAV 
SU01_OLD 
SU02 
SU03 
SU05_OLD 
SU1 
SU10 
SU10_OLD 
SU12 
SU2 
SU20 
SU20_BTCH 
SU21 
SU22 
SU22_HISTORY 
SU22_OLD 
SU24 
SU24_HISTORY 
SU24_OLD 
SU24_S_TABU_NAM 
SU25 
SU25_2A_SEL 
SU25_OLD 
SU26 
SE09
SE10
SE11
SE15
SE16
SE18
SE19
SE20
SE21
SE24
SE30
SE32
SE37
SE38
SE39
SE80



********************************************************************
* 获得事物代码分段
* 功能:获得，去除不需要的事物代码后的事物代码分段,BASIS授权使用
* 作者：刘欣  Power By James All Rights Reserved.
* 2014-6-28 潍柴项目
********************************************************************


REPORT ZBA_R005 .


*定义str_code结构
TYPES : BEGIN OF str_code,
        item LIKE agr_tcodes-tcode,
        END OF str_code.


*定义str_code2结构
TYPES : BEGIN OF str_code2,
        low LIKE agr_tcodes-tcode,
        high LIKE agr_tcodes-tcode,
        END OF str_code2.


*定义big内表
DATA big TYPE STANDARD TABLE OF str_code WITH HEADER LINE .
*定义min内表
DATA min TYPE STANDARD TABLE OF str_code WITH HEADER LINE .


*定义out内表
DATA out TYPE STANDARD TABLE OF str_code2 WITH HEADER LINE .


*定义用户输入的不好的事物代码
DATA divcode LIKE agr_tcodes-tcode.


*//--------------定义SELECTION SCREEN----------------------
SELECTION-SCREEN BEGIN OF BLOCK block_body WITH FRAME.
SELECTION-SCREEN SKIP 1.
SELECTION-SCREEN COMMENT /1(57) t1 .
SELECTION-SCREEN SKIP 2.
SELECT-OPTIONS: div_itab FOR divcode NO INTERVALS.
SELECTION-SCREEN END OF BLOCK block_body.


*//--------------INITIALIZATION------------------
INITIALIZATION.

 t1 = '获得事物代码分段报表'.

*//-------------AT SELECTION-SCREEN--------------------------

AT SELECTION-SCREEN.
START-OF-SELECTION.

*取出11万个事物代码
SELECT TCODE INTO TABLE big FROM TSTC.

*按照坏代码换成haha
LOOP AT div_itab.
    LOOP AT big.
      IF big-item EQ div_itab-LOW.
         big-item = 'haha'.
         MODIFY big .
      ENDIF.
    ENDLOOP.
ENDLOOP.

*按haha为界分段，取每小段的第一个和最后一个代码。
LOOP AT big.
  IF big-item <> 'haha'.
     APPEND big-item to min.
      AT LAST.
       READ TABLE min INTO out-low INDEX 1.
       READ TABLE min INTO out-high INDEX lines( min ).
       APPEND out.
       CLEAR min[].
      ENDAT.
  ELSE.




    AT END OF item.
       READ TABLE min INTO out-low INDEX 1.
       READ TABLE min INTO out-high INDEX lines( min ).
       APPEND out.
       CLEAR min[].
    ENDAT.


  ENDIF.


ENDLOOP.


*输出结果

perform listshow.
end-OF-SELECTION.


*//-------form listshow-----------------


form listshow.
  DATA:
      G_REPID TYPE SY-REPID,
      IT_EVENTS TYPE SLIS_T_EVENT,
      IT_FIELD TYPE SLIS_T_FIELDCAT_ALV,
      WA_FIELD TYPE SLIS_FIELDCAT_ALV,
      IT_SORT TYPE SLIS_T_SORTINFO_ALV.
*
*G_REPID = SY-REPID.


********宏定义.
  DEFINE ADD_FIELD.
    WA_FIELD-FIELDNAME = &1.
    WA_FIELD-REPTEXT_DDIC = &2.
    WA_FIELD-NO_ZERO = 'X'.
    WA_FIELD-outputlen = 25. "列宽
    APPEND WA_FIELD TO IT_FIELD.
  END-OF-DEFINITION.


  ADD_FIELD 'low'  '分段下限'. "注意这里low要同内表的字段一致
  ADD_FIELD 'high'  '分段上限'.




  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
       EXPORTING
            I_CALLBACK_PROGRAM = G_REPID
            I_BACKGROUND_ID   = 'ALV_BACKGROUND'
*            I_GRID_TITLE      = ' 用户权限查询'
            IT_FIELDCAT        = IT_FIELD
*            IS_LAYOUT          = GS_LAYOUT
*            IT_SORT            = IT_SORT
            I_SAVE             = 'A'
            IT_EVENTS          = IT_EVENTS[]
       TABLES
            T_OUTTAB           = out
       EXCEPTIONS
            PROGRAM_ERROR = 1
            OTHERS        = 2.


endform.

