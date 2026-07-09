[![](https://blogfiles.pstatic.net/MjAyMjEwMjlfNDAg/MDAxNjY3MDM2NjA2MzA5.KX5DyUQKHdtjPMp-sHIy5kTFiQ7-FD8kB8wiSd-ilSog.SMjkD-SMR0Jre7nHZ1UuR8Ddr37nHZVBxBlksdP8Yccg.PNG.getinthere/image.png)](https://blogfiles.pstatic.net/MjAyMjEwMjlfNDAg/MDAxNjY3MDM2NjA2MzA5.KX5DyUQKHdtjPMp-sHIy5kTFiQ7-FD8kB8wiSd-ilSog.SMjkD-SMR0Jre7nHZ1UuR8Ddr37nHZVBxBlksdP8Yccg.PNG.getinthere/image.png)


## 1. Java 언어선택


## 2. 세팅하기

단축키는 tt

탭키 누를때마다 이동되는 곳이 $1, $2 순서

```json
{
	"Junit Test Method": {
		"prefix": "tt",
		"body": [
			"@Test",
			"public void $1_test() throws Exception {",
			"    // given",
			"    $2",
			"",
			"    // when",
			"",
			"",
			"    // then",
			"",
			"}"
		],
		"description": "Junit Test Method"
	},
	"Log": {
		"prefix": "logd",
		"body": [
			"log.debug(\"디버그 : $1\"$2);"
		],
		"description": "logd"
	},
	"Sysout": {
		"prefix": "syst",
		"body": [
			"System.out.println(\"테스트 : $1\"$2);"
		],
		"description": "sysout"
	},
	"ReturnMapping": {
		"prefix": "rr",
		"body": [
			"return new ResponseEntity<>(new ResponseDto<>(1, \"\", null), HttpStatus.OK);",
		],
		"description": "ResponseMapping"
	},
	"ErrorMapping": {
		"prefix": "err",
		"body": [
			"return new ResponseEntity<>(new ResponseDto<>(-1, \"\", null), HttpStatus.BAD_REQUEST);",
		],
		"description": "ResponseMapping"
	},
	"GetMapping": {
		"prefix": "getm",
		"body": [
			"@GetMapping(\"/$1\")",
			"public ResponseEntity<?> $2(){",
			"    $3",
			"return new ResponseEntity<>(new ResponseDto<>(1, \"\", null), HttpStatus.OK);",
			"}",
		],
		"description": "Mapping"
	},
	"PostMapping": {
		"prefix": "postm",
		"body": [
			"@PostMapping(\"/$1\")",
			"public ResponseEntity<?> $2(){",
			"    $3",
			"    return new ResponseEntity<>(new ResponseDto<>(1, \"\", null), HttpStatus.CREATED);",
			"}",
		],
		"description": "Mapping"
	},
	"PutMapping": {
		"prefix": "putm",
		"body": [
			"@PutMapping(\"/$1\")",
			"public ResponseEntity<?> $2(){",
			"    $3",
			"    return new ResponseEntity<>(new ResponseDto<>(1, \"\", null), HttpStatus.OK);",
			"}",
		],
		"description": "Mapping"
	},
	"DeleteMapping": {
		"prefix": "delm",
		"body": [
			"@DeleteMapping(\"/$1\")",
			"public ResponseEntity<?> $2(){",
			"    $3",
			"    return new ResponseEntity<>(new ResponseDto<>(1, \"\", null), HttpStatus.OK);",
			"}",
		],
		"description": "Mapping"
	},

	"Logger": {
		"prefix": "logf",
		"body": [
			"private final Logger log = LoggerFactory.getLogger(getClass());"
		],
		"description": "Logger Field"
	},
	"MapToList": {
		"prefix": "mapToList",
		"body": [
			"$1.stream().map((e)->e).collect(Collectors.toList());"
		],
		"description": "MapToList"
	},
	"AssertThatEquals": {
		"prefix": "asse",
		"body": [
			"assertThat($1).isEqualTo($2);"
		],
		"description": "AssertThatEquals"
	},
}
```

## 3. 사용법

[![](https://blogfiles.pstatic.net/MjAyMjEwMjlfMjg5/MDAxNjY3MDM2NTM3NjQ5.AJXbguMCS_fTZgoz5l0y_C9Nff9X7BaO-1HKzJlbXKIg.qpjVberxE37y_WQERPApYOVNHSO-rbDP-7Vru5Y3Ckog.PNG.getinthere/image.png)](https://blogfiles.pstatic.net/MjAyMjEwMjlfMjg5/MDAxNjY3MDM2NTM3NjQ5.AJXbguMCS_fTZgoz5l0y_C9Nff9X7BaO-1HKzJlbXKIg.qpjVberxE37y_WQERPApYOVNHSO-rbDP-7Vru5Y3Ckog.PNG.getinthere/image.png)

[![](https://blogfiles.pstatic.net/MjAyMjEwMjlfMTUz/MDAxNjY3MDM2NTQ3MDk1.xtl6TT75qb81VmtwPLHDRzA2283-G1Yc5z5SHwPQjckg.FI2fbrBHt_auReOsUSsO87E_f5WEYkhfBcqkwxpB2Ywg.PNG.getinthere/image.png)](https://blogfiles.pstatic.net/MjAyMjEwMjlfMTUz/MDAxNjY3MDM2NTQ3MDk1.xtl6TT75qb81VmtwPLHDRzA2283-G1Yc5z5SHwPQjckg.FI2fbrBHt_auReOsUSsO87E_f5WEYkhfBcqkwxpB2Ywg.PNG.getinthere/image.png)

## 관련 문서

- 상위 목차: [[Vscode 스프링 환경 세팅]]
- 이전 문서: [[4강 VsCode Junit 세팅]]
