### Faster Whisper - 디코딩/인코딩

![[image 12.png|image 12.png]]

### Faster Whisper - 전사옵션

|옵션명|타입|기본값|허용 범위|설명|
|---|---|---|---|---|
|`audio`|`str`, `BinaryIO`, `np.ndarray`|-|파일 경로, 바이너리, NumPy 배열|입력 오디오|
|`language`|`Optional[str]`|`None`|ISO 언어 코드|언어 미지정 시 자동 감지|
|`task`|`str`|`"transcribe"`|`"transcribe"`, `"translate"`|텍스트 전사 또는 영어 번역|
|`log_progress`|`bool`|`False`|`True`, `False`|처리 상태 로깅 여부|
|`beam_size`|`int`|`5`|`≥1`|디코딩 빔 탐색 폭 (클수록 정확)|
|`best_of`|`int`|`5`|`≥1`|샘플링 시 후보 수 (temperature > 0 일 때만)|
|`patience`|`float`|`1.0`|`≥1.0`|빔 탐색에서 더 나은 후보 기다림 정도|
|`length_penalty`|`float`|`1.0`|>0|긴 문장에 대한 패널티 조절|
|`repetition_penalty`|`float`|`1.0`|≥1.0|반복 억제용 패널티|
|`no_repeat_ngram_size`|`int`|`0`|≥0|반복되는 n-gram 억제 크기|
|`temperature`|`float` or `list[float]`|`[0.0, 0.2, 0.4, 0.6, 0.8, 1.0]`|0.0~1.0+|샘플링 다양성 조절|
|`compression_ratio_threshold`|`float`|`2.4`|≥0|압축률 초과 시 실패 처리|
|`log_prob_threshold`|`float`|`-1.0`|≤0|평균 로그확률 이하 시 실패 처리|
|`no_speech_threshold`|`float`|`0.6`|0.0~1.0|무음 판단 기준 확률값|
|`condition_on_previous_text`|`bool`|`True`|`True`, `False`|이전 텍스트를 다음 세그먼트 프롬프트로 전달|
|`prompt_reset_on_temperature`|`float`|`0.5`|≥0.0|온도 초과 시 프롬프트 리셋 기준|
|`initial_prompt`|`str`, `Iterable[int]`|`None`|-|첫 window에 사용할 프롬프트|
|`prefix`|`Optional[str]`|`None`|-|첫 window에 붙일 강제 텍스트|
|`suppress_blank`|`bool`|`True`|`True`, `False`|초기 blank 토큰 억제 여부|
|`suppress_tokens`|`List[int]` or `[-1]`|`[-1]`|-|억제할 토큰 ID 리스트 (`-1`: 기본 음성 토큰)|
|`without_timestamps`|`bool`|`False`|`True`, `False`|텍스트만 출력 (타임스탬프 없음)|
|`max_initial_timestamp`|`float`|`1.0`|≥0|첫 segment의 최대 시작시간 제한|
|`word_timestamps`|`bool`|`False`|`True`, `False`|단어별 타임스탬프 출력 여부|
|`prepend_punctuations`|`str`|`"'“¿([{-`|-|다음 단어에 붙일 구두점|
|`append_punctuations`|`str`|`"'.。,，!！?？:：”)]}、`|-|앞 단어에 붙일 구두점|
|`multilingual`|`bool`|`False`|`True`, `False`|segment별 언어 감지 여부|
|`vad_filter`|`bool`|`False`|`True`, `False`|Silero VAD로 무음 제거 여부|
|`vad_parameters`|`dict` or `VadOptions`|`None`|-|VAD 세부 파라미터 설정|
|`max_new_tokens`|`Optional[int]`|`None`|≥1 or `None`|chunk당 최대 토큰 생성 수|
|`chunk_length`|`Optional[int]`|`None`|≥1 or `None`|오디오를 나눌 단위 길이 (초 단위)|
|`clip_timestamps`|`str`, `List[float]`|`"0"`|"start,end,...", `[float,...]`|처리할 오디오 클립 시간 지정|
|`hallucination_silence_threshold`|`Optional[float]`|`None`|≥0 or `None`|단어가 환청처럼 나올 경우 무음 길이 제한|
|`hotwords`|`Optional[str]`|`None`|-|인식 정확도 향상을 위한 힌트 키워드|
|`language_detection_threshold`|`float`|`0.5`|0~1|언어 감지 확률 임계값|
|`language_detection_segments`|`int`|`1`|≥1|언어 감지에 사용할 segment 수|

### Faster Whisper - VAD누적 전사 시 단점

![[image 1 3.png|image 1 3.png]]

### Faster Whisper - initial prompt 길이

![[image 2 3.png|image 2 3.png]]