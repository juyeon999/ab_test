# Paper Review - Detecting Network Effects: Randomizing Over Randomized Experiments

## 링크드인 사례 - Detecting interference: An A/B test of A/B

A/B 테스트에서의 Network effect (interference)의 중요성을 알아보고 어떻게 탐지할 수 있는지에 대한 방법론 및 실험 결과를 담은 paper로 링크드인의 데이터셋을 활용한다.

## Network effect
A/B 테스트는 많은 곳에서 쓰이는 방법이다. 하지만 강한 가설을 가정하고 있다: 
특징 A와 특징 B에 대해 비교할 때, A를 사용하는 유저의 행동은 B를 사용하는 유저의 행동에 영향을 받지 않는다는 가정이다. 
즉 there is no interference, which is sometimes called 'network effect', between features.

- A/B testing without interference
![image](https://github.com/juyeon999/ab_test/assets/132811616/a780a6b4-fbc4-4c7c-a36b-b0cc2560993b)
- A/B testing with interference. When treatment leaks into control, we can no longer rely on computing mean(B) – mean(A).
![image](https://github.com/juyeon999/ab_test/assets/132811616/0dafbc87-11cd-41d6-b7d9-71dae3ae7d1d)
예를 들어, control group과 treatement group 간의 약 A, B에 대한 효과 차이라면 control group이 A의 약을 먹는 것이 treatment group에 아무런 영향을 미치지 않을 것이다.
하지만 메신저 인터페이스에 대한 테스트를 한다고 했을 때, 내 친구는 타겟 그룹이고 나는 아니라고 했을 때, 그녀의 향상된 메신저 인터페이스 경험 때문에 나에게 메신저를 더 자주 해서 덩달아 나도 더 많이 메신저를 쓰게 된다면 이게 바로 네트워크 효과이다. 나는 실험의 일부가 아님에도 실험에 영향을 받게 되는 것이다. 이게 바로 interference이다.


## 이게 왜 중요한가?
Network effect 때문에 평균에 차이가 생긴다면 실험 결과를 신뢰할 수 없다. 하지만 이를 인지(탐지)하지 못한다면, 잘못된 A/B 테스트 결과를 신뢰하여 잘못된 결론으로 이어질 수 있다. 

## 그래서 어떻게 실험?
- 링크드인 그래프 데이터를 만들어서 10,000개의 클러스터를 만든다.
- 그리고 클러스터를 2개의 그룹으로 쪼갠다:
- A) 개인 수준의 A/B 테스트로, members를 실험군과 대조군으로 랜덤하게 나눈다.
- B) 클러스터 수준의 A/B 테스트로, 개별 클러스터 하나가 통채로 실험군 또는 대조군으로 할당한다.

만약 network 효과가 없다면 두 실험 A, B에 대해서는 동일한 효과를 줘야한다.
![image](https://github.com/juyeon999/ab_test/assets/132811616/ca6488f5-9724-494e-9eeb-796ed1822365)

## 네트워크(graph) 데이터를 만들어 클러스터 만들기
클러스터를 만들기 위해서는 그래프 데이터가 필요하다. 그래프 데이터를 생성 할 때 유의할 점은, 한 member는 하나의 클러스터에만 들어가야 한다는 점이다. 이를 위해 클러스터 간의 절단(cuts)를 찾아, 랜덤화가 될 수 있게 가능한 많은 클러스터를 보유하면서도 클러스터 간의 차이가 있게 한다.  
reLDG 알고리즘이 가장 좋은 성능을 보였으며 grid search로 클러스터의 수를 정했습니다.
![image](https://github.com/juyeon999/ab_test/assets/132811616/8f527e10-c879-46a7-a894-41db997b3b86)

## 실험 A, B에 대해 control group, treatment group 만들기
위에서 만든 네트워크 데이터를 기반으로 클러스터의 50%는 개인 단위의 실험(A)에 할당하고 나머지 50%는 클러스터 단위의 실험(B)에 할당했다. 그리고 각 실험에서 50%는 treatment, 50%는 control group에 랜덤 할당 했다.  
그리곤 테스트 A, B를 각각 진행하고 결과를 비교했다. 
![image](https://github.com/juyeon999/ab_test/assets/132811616/cffffd2b-de83-425f-ad64-b33b67488a27)

## 결과 비교
Indiviual A/B test (A)와 cluster-based A/B test (B)간에 효과 차이가 있나 검증한다. 
- Individual A/B test: 평균 차이 $\mu_{BR}$, 표준편차 $\sigma_{BR}$
- Cluster-based A/B test: 평균 차이 $\mu_{CBR}$, 표준편차 $\sigma_{CBR}$
- 두 테스트간 효과 차이: $\Delta = \mu_{BR} - \mu_{CBR}$
- 가설 검정: $H_0: \Delta = 0$, $H_a: \Delta \neq 0$
![image](https://github.com/juyeon999/ab_test/assets/132811616/608c3c29-5fe0-41e8-8b1b-ec1e883a7ae2)
![image](https://github.com/juyeon999/ab_test/assets/132811616/209ae3c1-6ed7-451d-a184-0c2cb5f3c163)
만약 귀무가설을 기각하면 우리는 interference가 있다고 결론 내릴 수 있는 것이다. 

## 결과
Experiment 2을 보면 BR Treatment Effect($\tilda{\mu_{br}}$)의 post-treatment와 CBR Treatment Effect($\tilda{\mu_{cbr}}$)의 post-treatment간의 차이가 크다. 즉 Reject 귀무가설을 통해 interference가 있다고 결론 내릴 수 있다.

## 의의
Interference가 있는지 탐지 할 수 있는 테스트를 할 수 있게 되었다. 따라서 A/B 테스트를 수행할 때 해당 테스트를 통해 interference 효과가 있는지 테스트 해볼 수 있고 A/B 테스트의 효과에 대한 해석을 더 정확하게 할 수 있게 되었다.

## Reference
- https://engineering.linkedin.com/blog/2019/06/detecting-interference--an-a-b-test-of-a-b-tests
- https://dl.acm.org/doi/pdf/10.1145/3097983.3098192?casa_token=tXDsbTeb8ecAAAAA:_yETdgf1xsL8ADT4QYXA0C6o3kv9WRI8yLwlvkfx6h-QN_BPqyYOGe3wN0nuHg_3zQs2TVWZgB_zfA
