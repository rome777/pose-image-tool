# 테스트에 쓴 프롬프트 모음

노트(`pose_tool.ipynb`)에 그대로 들어 있는 문자열입니다.
이미지마다 시드·모델·스텝·가이던스까지 포함한 전체 기록은 `samples/*.json` 에 있습니다.

## 공통 설정

| 항목 | 값 |
|---|---|
| 본체 모델 | `stabilityai/stable-diffusion-xl-base-1.0` |
| ControlNet | `thibaud/controlnet-openpose-sdxl-1.0` |
| VAE | `madebyollin/sdxl-vae-fp16-fix` |
| dtype | `float16` |
| 스텝 | 25 |
| guidance_scale | 5.0 (증류 모델이 아니므로 0.0이 아님) |
| controlnet_conditioning_scale | 0.8 (10번 셀에서만 0.3 / 1.2 로 변경) |
| 크기 | 768 × 1024 |
| 시드 | 20260916 (전부 고정) |

네거티브 프롬프트는 전부 동일합니다. 짧게, 실제로 효과를 확인한 것만 넣었습니다.

```
cropped, out of frame, extra limbs, deformed hands, text, watermark
```

---

## 1. 참조 사진 프롬프트 (자세를 만들기 위한 것)

참조 사진은 **한 명 / 전신 / 팔다리가 몸에 겹치지 않는 것** 세 가지가 지켜져야 합니다.
관절이 가려지면 OpenPose가 못 찾고, 못 찾은 관절은 조건으로 들어가지 않습니다.

### pose_01 — 정면을 보고 선 자세

```
Full body studio photograph of one adult in a plain gray t-shirt and jeans, standing on a
seamless light gray backdrop, feet apart, left arm raised straight up above the head, right
arm extended horizontally to the side, facing the camera. Even soft studio lighting from the
front, 50mm lens, the whole body from head to shoes inside the frame. Photorealistic
reference photograph, neutral colors.
```

> **실제로 나온 것:** 두 팔을 내리고 정면을 보고 선 평범한 자세. **요청한 팔 동작이 무시됐습니다.**
> 이게 이 도구가 필요한 이유입니다. 자세는 프롬프트로 지시해도 잘 안 듣습니다.
> 그래도 실습에는 지장이 없어 나온 그대로 뼈대를 뽑아 썼습니다(`samples/pose_01.png`).

### pose_02 — 한쪽 무릎을 꿇은 자세

```
Full body studio photograph of one adult in a plain gray t-shirt and jeans, crouching on one
knee on a seamless light gray backdrop, right knee down, left forearm resting on the left knee,
head turned to the left, side three-quarter view. Even soft studio lighting from the front,
50mm lens, the whole body from head to shoes inside the frame. Photorealistic reference
photograph, neutral colors.
```

> **실제로 나온 것:** 요청한 대로 한쪽 무릎을 꿇고 앞쪽 무릎에 손을 얹은 자세.
> 뼈대도 깨끗하게 뽑혔습니다(`samples/pose_02.png`).

---

## 2. 실험 1 — 자세는 그대로, 프롬프트만 바꿈 (전부 pose_01)

세 프롬프트 모두 **동작 칸이 비어 있습니다.** 그 칸은 뼈대가 이미 채웠기 때문입니다.
자세를 글로 또 적으면 뼈대와 충돌합니다.

### output_01 — 공장 노동자 / 다큐멘터리 사진

```
A factory worker in a white safety helmet and navy work uniform inside a large manufacturing
plant. Medium full shot, static camera, 50mm lens. Soft morning light from tall windows on the
left throws long reflections on the polished concrete floor. Rows of idle machinery recede into
shallow depth of field. Photorealistic documentary photography, muted industrial color palette.
```

| 칸 | 채운 내용 |
|---|---|
| 피사체 | 안전모와 남색 작업복 차림의 공장 노동자 한 명 |
| 동작 | *(비움 — 뼈대가 담당)* |
| 카메라 | 미디엄 풀 샷 / 고정 / 50mm |
| 조명 | 왼쪽 큰 창에서 들어오는 부드러운 아침 빛 |
| 환경 | 대형 제조 공장, 뒤쪽에 놓인 기계들 |
| 스타일 | 다큐멘터리 사진, 채도 낮은 산업 팔레트 |

### output_01b — 우주 비행사 / 시네마틱

```
An astronaut in a white spacesuit on the dusty surface of Mars. Full shot, static camera,
35mm lens. Hard low sunlight from the right, long sharp shadow on red sand. Distant rocky
ridges under a pale orange sky. Photorealistic cinematic still, warm desaturated palette.
```

### output_01c — 한복 차림 선비 / 일러스트

```
A traditional Korean scholar in a pale blue hanbok standing in a temple courtyard. Full shot,
static camera, 85mm lens. Warm late afternoon light from the left, soft shadows on stone tiles.
Wooden pillars and a tiled roof behind, softly out of focus. Painterly illustration, ink and
watercolor texture.
```

**바꾼 칸:** 피사체 / 환경 / 조명 방향 / 렌즈 / 스타일. **안 바꾼 것:** 자세, 시드, 모델, 스텝, 가이던스.

**결과:** 세 장 모두 자세가 유지되고 인물·옷·배경·화풍만 바뀌었습니다.

---

## 3. 실험 2 — 프롬프트는 그대로, 자세만 바꿈

### output_02

프롬프트는 `output_01`과 **한 글자도 다르지 않습니다.** 시드도 같습니다.
바뀐 것은 ControlNet에 들어간 뼈대뿐입니다 (`pose_01` → `pose_02`).

```
A factory worker in a white safety helmet and navy work uniform inside a large manufacturing
plant. Medium full shot, static camera, 50mm lens. Soft morning light from tall windows on the
left throws long reflections on the polished concrete floor. Rows of idle machinery recede into
shallow depth of field. Photorealistic documentary photography, muted industrial color palette.
```

---

## 4. 실험 3 — 자세 강제 세기만 바꿈

프롬프트는 `output_01`, 뼈대는 `pose_01`, 시드도 동일.
`controlnet_conditioning_scale` 한 값만 바꿉니다.

| 파일 | scale | 관찰 |
|---|---|---|
| `scale_03.png` | 0.3 | **인물이 뒤로 돌아섰다.** 서 있다는 것만 맞고 방향·팔 위치가 뼈대와 어긋남 |
| `scale_08.png` | 0.8 | 절충점. 뼈대를 그대로 따르면서 화풍·조명 지시도 살아 있음 |
| `scale_12.png` | 1.2 | 자세는 정확하지만 **손이 무너지고** 손 자리에 주인 없는 기계 부품이 붙음 |

---

## 5. 재현성 확인

`repro_check.png` 는 `output_01` 과 **완전히 같은 조건**(프롬프트·시드·뼈대·스텝·가이던스)으로
한 번 더 뽑은 것입니다. 두 이미지의 픽셀 차이를 계산해 최대값이 0인지 확인합니다.

실제 결과는 **픽셀 차이 0**, 파일 해시까지 동일했습니다.

```
output_01.png    ef353293140b255f
repro_check.png  ef353293140b255f
scale_08.png     ef353293140b255f   (기본 scale 0.8 과 같은 조건이라 같은 파일)
output_02.png    f5bdc989f4e197c4   (뼈대만 다름)
```

시드를 문자열에서 만들 때는 파이썬 기본 `hash()` 를 쓰지 않습니다.
보안상의 이유로 프로그램을 새로 실행할 때마다 값이 달라져서, 오늘 뽑은 그림을 내일 다시 못 뽑습니다.
노트에서는 `hashlib.sha256` 으로 만듭니다.

---

## 6. 한국어 프롬프트에 대해

프롬프트는 전부 영어로 썼습니다. 학습 데이터의 캡션이 영어에 크게 치우쳐 있어 영어 쪽이 훨씬 안정적입니다.
한국어로 장면을 정리한 다음 영어로 옮겨 넣는 방식을 권합니다.
