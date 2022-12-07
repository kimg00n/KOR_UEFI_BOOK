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

## 숫자형 VFR 요소

숫자 VFR 요소를 조사해보자. 이를 통해 사용자는 HII 폼에서 숫자 값을 입력할 수 있다. [https://edk2-docs.gitbook.io/edk-ii-vfr-specification/2\_vfr\_description\_in\_bnf/211\_vfr\_form\_definition#2.11.6.6.1-vfr-numeric-statement-definition](https://edk2-docs.gitbook.io/edk-ii-vfr-specification/2\_vfr\_description\_in\_bnf/211\_vfr\_form\_definition#2.11.6.6.1-vfr-numeric-statement-definition)

새 요소에는 스토리지가 필요하므로 UEFI 변수 구조에 `UINT8 NumericValue` 필드를 추가한다.

```c
typedef struct {
  UINT8 CheckboxValue;
  UINT8 NumericValue;
} UEFI_VARIABLE_STRUCTURE;
```

또한 HII폼에 필요한 몇 가지 문자열을 `UefiLessonsPkg/HIIFormDataElements/Strings.uni`에 추가한다.

```
#string NUMERIC_PROMPT         #language en-US  "Numeric prompt"
#string NUMERIC_HELP           #language en-US  "Numeric help"
```

요소에 대한 최소한의 VFR 코드는 다음과 같다.

```
numeric
  varid = FormData.NumericValue,
  prompt = STRING_TOKEN(NUMERIC_PROMPT),
  help = STRING_TOKEN(NUMERIC_HELP),
  minimum = 5,
  maximum = 20,
endnumeric;
```

`minimum`과 `maximum`필드는 숫자 요소에 필수이다. 이 예제에서는 값을 `5..20` 범위로 제한한다.

UEFI 변수의 크기를 변경했으므로 드라이버를 로드하기 전에 제거하는 것이 좋다.(unload 명령어 사용)

```
FS0:\> dmpstore -guid 531bc507-9191-4fa2-9446-b844e35dd12a -d
FS0:\> load HIIFormDataElements.efi
```

우리의 폼을 로드하면 다음과 같다.

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

보다시피 기본적으로 값은 0이다.(`ZeroMem(&EfiVarstore, sizeof(EfiVarstore))`을 사용하기 때문) 이는 값이 제한 범위(5..20)를 벗어남을 의미한다. 요점은 HII 폼은 초기 값을 제어할 수 없다는 것이다. 최대값 및 최소값은 오직 사용자의입력에 대한 제한이다.

범위를 벗어난 값(예: 4)을 입력하려고 하면 폼에서 허용하지 않는다.

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

폼 엔진은 허용되지 않는 입력을 입력하는 것까지만 허용한다. 예를 들어 첫 번째 기호로 `3`을 입력하면 폼 엔진은 다른 숫자 기호를 추가로 입력하는 것을 허용하지 않는다. 값이 사용 가능한 범위를 벗어나기 때문이다.&#x20;

그러나 허용 범위에 있으면 값을 성공적으로 입력하고 저장(`F10`)할 수 있다.

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

`dmpstore` 명령을 사용하여 값이 성공적으로 설정되었는지 확인할 수 있다. 우리의 경우 값은 저장소의 두 번째 바이트에 있다.(`0x0f = 15`)

```
Shell> dmpstore -guid 531bc507-9191-4fa2-9446-b844e35dd12a
Variable NV+BS '531BC507-9191-4FA2-9446-B844E35DD12A:FormData' DataSize = 0x02
  00000000: 00 0F                                            *..*
```

