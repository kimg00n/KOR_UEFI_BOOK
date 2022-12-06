---
description: '파트 1: 숫자'
---

# 65. 더 많은 VFR 입력 요소들

지난 레슨에서는 checkbox VFR요소를 사용했었다. 사용자 입력에 사용할 수 있는 다른 VFR 요소들을 살펴보자.&#x20;

## efivarstore에서 사용자 정의 구조체를 사용하는 HII 애플리케이션 생성하기

`HIIFormCheckbox` 드라이버의 코드를 기반으로 하나의 체크박스 요소가 포함된 `HIIFormDataElements` 드라이버를 만들어보자.

이 레슨의 UEFI 변수는 폼에 있는 다른 VFR 요소의 데이터를 포함하므로 사용자 정의 구조체 유형 내부의 checkbox에 대한 UINT8 스토리지를 배치해야 한다.

typedef를 통해 사용자 정의 구조체를 만들기 위해 헤더 파일 `UefiLessonsPkg/HIIFormDataElements/Data.h` 를 생성한다. 변수 이름과 GUID 정의(우리의 경우 FORMSET\_GUID와 동일하다고 결정함)도 이 헤더파일로 이동시켜준다.

`UefiLessonsPkg/HIIFormDataElements/Data.h`

```c
#ifndef _DATA_H_
#define _DATA_H_

#define FORMSET_GUID  {0x531bc507, 0x9191, 0x4fa2, {0x94, 0x46, 0xb8, 0x44, 0xe3, 0x5d, 0xd1, 0x2a}}

#define UEFI_VARIABLE_STRUCTURE_NAME L"FormData"

#pragma pack(1)
typedef struct {
  UINT8 CheckboxValue;
} UEFI_VARIABLE_STRUCTURE;
#pragma pack()

#endif
```

다음은 `UefiLessonsPkg/HIIFormDataTemplate/HIIFormDataElements.c`에 적용해야 하는 수정 사항이다.

```c
...
#include "Data.h"			<--- 헤더 파일 추가하기

...

EFI_STRING      UEFIVariableName = UEFI_VARIABLE_STRUCTURE_NAME;   <--- UEFI 변수 이름에 대한 상수 추가


EFI_STATUS
EFIAPI
HIIFormDataElementsUnload (
  EFI_HANDLE ImageHandle
  )
{
  ...

  UEFI_VARIABLE_STRUCTURE EfiVarstore;		<--- `UINT8`대신 우리의 커스텀 스토리지 사용
  BufferSize = sizeof(UEFI_VARIABLE_STRUCTURE); <---

  Status = gRT->GetVariable(
                UEFIVariableName,				<--- UEFI 변수 이름으로 변수 사용
                &mHiiVendorDevicePath.VendorDevicePath.Guid,
                NULL,
                &BufferSize,
                &EfiVarstore);
  if (!EFI_ERROR(Status)) {
    Status = gRT->SetVariable(
                  UEFIVariableName,                             <--- UEFI 변수 이름으로 변수 사용
                  &mHiiVendorDevicePath.VendorDevicePath.Guid,
                  0,
                  0,
                  NULL);
  ...
}


EFI_STATUS
EFIAPI
HIIFormDataElementsEntryPoint (
  IN EFI_HANDLE        ImageHandle,
  IN EFI_SYSTEM_TABLE  *SystemTable
  )
{
  ...

  UEFI_VARIABLE_STRUCTURE EfiVarstore;        	<--- `UINT8`대신 우리의 커스텀 스토리지 사용
  BufferSize = sizeof(UEFI_VARIABLE_STRUCTURE); <---

  Status = gRT->GetVariable (
                UEFIVariableName,				<--- UEFI 변수 이름으로 변수 사용
                &mHiiVendorDevicePath.VendorDevicePath.Guid,
                NULL,
                &BufferSize,
                &EfiVarstore);
  if (EFI_ERROR(Status)) {
    ZeroMem(&EfiVarstore, sizeof(EfiVarstore));
    Status = gRT->SetVariable(
                  UEFIVariableName,                             <--- UEFI 변수 이름으로 변수 사용
                  &mHiiVendorDevicePath.VendorDevicePath.Guid,
                  EFI_VARIABLE_NON_VOLATILE | EFI_VARIABLE_BOOTSERVICE_ACCESS,
                  sizeof(EfiVarstore),
                  &EfiVarstore);
  ...
}
```

그리고 `UefiLessonsPkg/HIIFormDataElements/Form.vfr`의 수정사항

```c
#include <Uefi/UefiMultiPhase.h>
#include "Data.h"                             <--- 헤더 파일 추가하기

formset
  guid     = FORMSET_GUID,
  title    = STRING_TOKEN(FORMSET_TITLE),
  help     = STRING_TOKEN(FORMSET_HELP),

  efivarstore UEFI_VARIABLE_STRUCTURE,                                          <--- 사용자 정의 유형 사용
    attribute = EFI_VARIABLE_BOOTSERVICE_ACCESS | EFI_VARIABLE_NON_VOLATILE,
    name  = FormData,                                                           <--- 새 UEFI 변수 이름
    guid  = FORMSET_GUID;

  form
    formid = 1,
    title = STRING_TOKEN(FORMID1_TITLE);

    checkbox
      varid = FormData.CheckboxValue,                                           <--- 구조체 요소로 엑세스
      prompt = STRING_TOKEN(CHECKBOX_PROMPT),
      help = STRING_TOKEN(CHECKBOX_HELP),
    endcheckbox;
endformset;
```

드라이버를 빌드하고 모든 것이 제대로 작동하는지 확인해본다.

