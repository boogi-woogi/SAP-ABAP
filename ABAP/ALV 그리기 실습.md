# 메인
```
*&---------------------------------------------------------------------*
*& Report ZTEST_KJU
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT Z_ALV_TEMPLETE.

INCLUDE Z_ALV_TEMPLETE_TOP.
INCLUDE Z_ALV_TEMPLETE_C01.
INCLUDE Z_ALV_TEMPLETE_PBO.
INCLUDE Z_ALV_TEMPLETE_PAI.
INCLUDE Z_ALV_TEMPLETE_F01.

INITIALIZATION.

AT SELECTION-SCREEN.

START-OF-SELECTION.

  PERFORM GET_DATA.

END-OF-SELECTION.

call screen 100.
```

### 실행순서
1. **INITIALIZATION** : Default값과 같은 것들을 지정해주는 section
      * 예시)
      ```
        INITIALIZATION.
          p_date = sy-datum.  " 오늘 날짜를 기본값으로
      ```
2. **AT SELECTION-SCREEN** : 프로그램을 실행하고 어떤 값을 넣고 엔터치거나 F8(execute) 누르는 순간 동작하는 이벤트.
    * 사용자가 선택 화면에서 입력을 마치고 실행버튼 눌렀을 때 호출되는 이벤트.
    * 검색할 때 사용자가 체크 박스(미상신, 결재중, 완료) 체크 안하고 검색할 때 경고창이 뜨는 케이스 -> 이런게 AT SELECTION-SCREEN에서 정의되어 있는 이벤트..
    * 입력값 검증과 특정 필드 입력 시 처리를 수행하는 이벤트가 주로 정의된다.
3. **START-OF-SELECTION** : 선택 화면에서 입력이 끝나고, 프로그램이 실행될 때 시작되는 메인 이벤트.
4. **END-OF-SELECTION** : START-OF-SELECTION 이후, 모든 데이터 처리가 끝난 후 실행.
    * 최종 결과 출력하는 단계이다. ( ALV / WRITE )
  
### 전체 실행 순서 요약
1. INITIALIZATION → 선택화면 초기값 설정
2. 사용자 입력 후 실행 클릭
3. AT SELECTION-SCREEN → 입력값 검증
4. START-OF-SELECTION → 데이터 조회 (PERFORM GET_DATA. 실행)
5. END-OF-SELECTION → 결과 출력
6. CALL SCREEN 100 → 화면 100 표시

# TOP
```
DATA : BEGIN OF GS_DISP,
EBELN TYPE EKKO-EBELN,       
BUKRS TYPE EKKO-BUKRS,       
BSART TYPE EKKO-BSART,      
	   END OF GS_DISP. 

DATA : GT_DISP LIKE TABLE OF GS_DISP.

//ALV 그리드를 위한 객체참조변수 2가지 선언

DATA : CON1_REF TYPE REF TO CL_GUI_CUSTOM_CONTAINER. 
DATA : g_grid TYPE REF TO Cl_GUI_ALV_GRID.

DATA : OK_CODE TYPE SY-UCOMM.

* ALV에 보여줄 컬럼(열) 정의 목록을 담는 내부 테이블. 타입 LVC_T_FCAT.
DATA : gt_fieldcat TYPE lvc_t_fcat.

* gt_fieldcat에 하나씩 채워서 APPEND 하는 작은 구조(한 컬럼 정의). 타입 LVC_S_FCAT.
DATA : gs_fieldcat TYPE lvc_s_fcat.

* ALV의 레이아웃/동작 설정(전체 그리드 설정) 을 담는 구조. 타입 LVC_S_LAYO.
DATA : gs_layout type lvc_s_layo. 
```

### 변수 설명
* **GS_DISP** : Global Structure의 약자로 alv에 표현할 데이터의 구조를 선언한 것
* **GT_DISP** : Global Table(=Internal table)의 약자로 alv에 표현할 데이터의 집합을 담는 변수(여러 행(레코드)을 담는 테이블형 변수, 그냥 배열이라고 생각하자...)
* **CON1_REF** : 레이아웃 편집기로 custom control을 배치하였는데 해당 컴포넌트를 참조하는 변수
    * custom contorl은 그냥 alv를 담는 container라고 생각하자.(안드로이드로치면 fragment담는 그 컴포넌트?)
    * 타입 : CL_GUI_CUSTOM_CONTAINER
* **gs_fieldcat** : internal table에 표현할 각각의 필드들을 정의하기 위한 변수
* **gt_fieldcat** : alv용 구조의 내부 테이블
    * **GT_DISP와의 차이는?** GT_DISP에는 실제 데이터들이 들어있다면 GT_FIELDCAT은 데이터를 화면에 어떻게 보여줄지 규정하는 데이터
  
  | 항목 | GT_DISP (예시) | GT_FIELDCAT |
  |------|----------------|--------------|
  | **목적** | 실제 데이터(행/레코드)를 담음 (출력할 레코드들) | 각 컬럼의 표시방법(라벨, 너비, 출력 타입 등)을 정의 |
  | **타입** | `LIKE TABLE OF GS_DISP` (업무 구조체 기반) | `TYPE lvc_t_fcat` 또는 `TYPE slis_t_fieldcat_alv` 등 ALV용 구조의 내부테이블 |
  | **내용 예** | `EBELN`, `BUKRS`, `BSART` 값들 (구체적 데이터) | `FIELDNAME = 'EBELN'`, `COLTEXT = '구매문서'`, `OUTPUTLEN = 10` 등 |
  | **ALV로 전달할 때** | `it_outtab = gt_disp` (데이터) | `it_fieldcatalog = gt_fieldcat` (메타정보) |
  | **비유** | 사람들의 명부(이름, 나이, 주소 등 실제 데이터) | 명부를 출력할 때의 ‘컬럼 제목/너비/정렬 규칙’ 설명서 |

   * **예시코드)**
  ```
  TYPES: BEGIN OF ty_disp,
         ebeln TYPE ekko-ebeln,
         bukrs TYPE ekko-bukrs,
         bsart TYPE ekko-bsart,
       END OF ty_disp.

  DATA: gs_disp TYPE ty_disp,
        gt_disp TYPE TABLE OF ty_disp.

  " fieldcat 관련
  DATA: gs_fieldcat TYPE lvc_s_fcat,
        gt_fieldcat TYPE lvc_t_fcat.

  " --- 데이터 예시 추가 ---
  gs_disp-ebeln = '450000001'.
  gs_disp-bukrs = '1000'.
  gs_disp-bsart = 'NB'.
  APPEND gs_disp TO gt_disp.
  CLEAR gs_disp.
  " ... (더 많은 레코드들)

  " --- 필드카탈로그 구성 (각 컬럼에 대해 한 레코드씩) ---
  gs_fieldcat-fieldname = 'EBELN'.        " 꼭 대문자, 데이터 테이블의 필드명
  gs_fieldcat-coltext   = '구매문서번호'.
  gs_fieldcat-outputlen = 10.
  APPEND gs_fieldcat TO gt_fieldcat.

  CLEAR gs_fieldcat.
  gs_fieldcat-fieldname = 'BUKRS'.
  gs_fieldcat-coltext   = '회사코드'.
  gs_fieldcat-outputlen = 6.
  APPEND gs_fieldcat TO gt_fieldcat.

  CLEAR gs_fieldcat.
  gs_fieldcat-fieldname = 'BSART'.
  gs_fieldcat-coltext   = '구매문서유형'.
  gs_fieldcat-outputlen = 4.
  APPEND gs_fieldcat TO gt_fieldcat.
  ```

