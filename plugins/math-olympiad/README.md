# math-olympiad

적대적 검증(adversarial verification)을 활용한 경시 수학 솔버입니다.

## 문제점 (The problem)

자가 검증(self-verification)은 속기 쉽습니다. 풀이 과정을 함께 보는 검증기(verifier)는 풀이에 동의하는 방향으로 편향됩니다. arXiv:2503.21934 ("Proof or Bluff")에 따르면 자가 검증을 통한 IMO 성공률 85.7%가 사람의 채점 하에서는 5% 미만으로 급감하는 것으로 나타났습니다.

## 해결 접근법 (The approach)

- **컨텍스트 격리 검증 (Context-isolated verification)**: 검증기는 풀이 과정 흔적(reasoning trace)을 보지 못하고 오직 정돈된 증명 내용만 확인합니다.
- **패턴 무장 적대적 체크 (Pattern-armed adversarial checks)**: "이것이 맞습니까?"가 아닌 "이 증명이 우연히 리만 가설(RH)까지 증명해 버리지 않는가?" / "일반적인 보조정리를 추출하고, 2×2 반례를 찾으라"와 같은 적대적 확인을 수행합니다.
- **조정된 기권 (Calibrated abstention)**: 억지 답변을 내놓기보다 "확신할 수 있는 솔루션이 없음"을 솔직하게 제시합니다.
- **프레젠테이션 단계 (Presentation pass)**: 검증을 통과한 후 깔끔한 LaTeX/PDF 결과물을 생성합니다.

## 검증 결과 (Validation)

2025년 IMO 및 Putnam 기출문제 18개 중 17개를 해결하였으며, 잘못 제안된 답변(false positive)은 0건, 신규 증명 2건을 발견했습니다. 스킬의 평가 데이터는 [anthropic monorepo](https://github.com/anthropics/anthropic/tree/staging/sandbox/sandbox/ralph/math_skills/eval_harness)에서 확인할 수 있습니다.

## 설치 (Install)

```
/plugin install math-olympiad@claude-plugins-official
```

## 사용법 (Use)

```
> Solve this IMO problem: [statement]
```

이 스킬은 "IMO", "Putnam", "olympiad", "verify this proof" 등의 단어가 입력되면 자동으로 트리거됩니다.
