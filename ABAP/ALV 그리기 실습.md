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
