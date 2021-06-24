# CI와 CD
CI와 CD는 애자일 개발 방법이 대세가 되는 요즘에 더욱 중요한 개념으로, 보통 DevOps 개발자들이 하는 일들이다. 

CI/CD는 어플리케이션을 개발할 때 (코드의 통합 및 테스트 → 배포 자동화)를 이뤄주며, 보통 두 과정이 자동화 되어 연결되며, 이를 CI/CD 파이프라인이라고 부른다. 

## CI

CI는 Continuous Integration을 의미하며, 지속적인 통합(코드)이라는 의미를 가진다. 

여러 개발자가 하나의 어플리케이션에서 작업할 때 충돌을 피하는것이 정말 중요하다.

이때, CI를 사용하면 각 개발자가 작성한 코드들이 하나의 공유 레포에 자동으로 빌드 및 테스트를 거치면서 충돌을 배제하게 통합된다. 

즉, 코드의 자동적인 빌드 → 테스트 → 통합(병합)이 이루어진다. 

이때 CI의 핵심은 '자동화'에 있으며 이는 다양한 툴들(ex. Jenkins)로 이루어질 수 있다. 

## CD

CD의 약자는 다음 두 가지의 뜻을 가지고 있을 수 있다. 

- Continuous Delivery
    - 지속적인 서비스 제공
- Continuous Deployment
    - 지속적인 배포
- 두 의미의 핵심적인 차이는 고객들이 사용하는 프로덕션 환경까지의 진행이 자동화 되어있는지 여부에 있다.

의미와 상관없이 CD는 CI가 선행된 이후 진행되는 단계다. 

### Continuous Delivery

CI 단계에서 개발자들이 작성한 코드들이 빌드, 테스트, 병합을 마친다면 

해당 코드가 깃헙같은 공유 레포까지 자동으로 업로드가 되는 것을 의미한다. 

다만, 이렇게 수정된 코드를 프로덕션 환경에까지 배포하는 것은 수동으로 해줘야 한다. 

### Continuous Deployment

Continuous Delivery와 의미는 같으며, 지속적 서비스 제공의 의미에서 확장되어

추가적으로 고객들의 프로덕션 환경에까지 자동으로 배포되는 것을 의미한다. 

정말 이상적인 사례라면 코드를 수정하고 몇분 내로 수정된 어플리케이션을 자동으로 실행해볼 수 있는 것을 의미한다. 

## CI & CD 파이프라인

이처럼 CI와 CD는 뗄래야 뗄 수 없는 관계이며, 보통 CI는 CD에 선행해서 이루어져야 한다. 

이 두 과정을 자동화 하여 하나로 이어놓은 것을 파이프라인이라고 한다. 

아래 그림에 잘 나와 있다. 

![https://www.redhat.com/cms/managed-files/ci-cd-flow-desktop_1.png](https://www.redhat.com/cms/managed-files/ci-cd-flow-desktop_1.png)

출처: [https://www.redhat.com/ko/topics/devops/what-is-ci-cd](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)
