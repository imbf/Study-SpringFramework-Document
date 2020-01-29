## 1.9 Annotation-based Container Configuration

> #### Spring 설정에 XML 보다 annotation이 더 좋나?
>
> annotation-based configuration 의 소개에서는 XML 보다 Annotation 기반의 configuration이 더 좋느냐 하는 질물을 일으켰습니다. 우리는 "때에 따라 다르지라고" 짧게 대답할 수 있습니다. 좀 더 길게 답하자면 각각의 접근법은 장점과 단점을 보유하고 있습니다. 보통, 어떠한 전략이 더 적합한지 결정하는 것은 개발자에게 달려(be up to)있습니다. **개발자가 정의하는 방식에 따라서, annotation들은 선언에 많은 수의 context를 제공하며 짧고 간결한 설정을 제공합니다.** **반면에 XML은 소스코드를 조작하거나 다시 컴파일 할 필요없이 components 간에 연결짓는데 더 훌륭합니다.** <u>일부 개발자들은 소스에 가까운 wiring을 선호하는 반면에 다른 사람들은 주석이 달린 클래스가 더이상 POJO가 아니며, 구성이 분산되어 제어하기 더 어렵다고 주장합니다.</u>
>
> 어느 선택을 하던간, Spring은 두가지의 스타일과 심지어는 같이 섞어논 스타일까지도 수용할(accommodate) 수 있습니다. Spring의 JavaConfig option 을 통해서 지적할만한 가치가 있습니다. Spring은 annotations이 비침투 방법으로 target 요소 소스코드를 건드리지 않고 tooling 측면에서 사용되어질 수 있도록 한다. 모든 설정 스타일은 Spring Tool Suite로부터 지원되어진다.



