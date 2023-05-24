# README

최근작업한 내용인 Raytracing 과 Footprint 기능에 대한 간략한 개요 입니다.

작업내용은 모드 UE4를 수정하여 구현되었습니다.

## 구현 결과
-   `Raytracing`
    -   Reflection & Global illumination
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXAnimation.gif?raw=true" width=600 height=350>
    -   ParticleSystem with HybridRendering
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXParticleAnimation.gif?raw=true" width=600 height=350>

-   `Footprint`
    -   Character Footprint
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintAnimation.gif?raw=true" width=600 height=350>

## Raytracing
-   `Reflection`
    -   최적화를 위해 물체표면의 러프니스와 메탈릭 수치에 따라 픽셀당 광선의 반사 횟수를 0 ~ 3회 사이로 동적으로 변경되도록 수정.
    -   광선의 최대 반사 횟수를 넘을 경우 환경맵핑 수행.
    -   레이트레이싱에서 Additive Blend Mode 와 Modulate Blend Mode 를 표현 할수 있도록 수정.
    -   불투명표면과 반투명표면간 반사가 일어날 수 있도록 수정.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXTranslucent.jpg?raw=true" width=600 height=250>

-   `Global illumination`
    -   NVidia의 RTXGI플러그인 적용.
        -   https://developer.nvidia.com/rtx/ray-tracing/rtxgi
    -   BaseColor가 다소 어둡게 제작된 아트 에섯들이 GI에 효과적으로 영향을 받을 수 있도록 플러그인 수정.
    
-   `ParticleSystem`
    -   ParticleSystem 레이트레이싱 지원
        -   UE4는 CascadeParticleSystem의 레이트레이싱을 지원하지 않음.
        -   따라서 게임에서 가장 많이 사용 되는 MeshParticle과 SpriteParticle이 레이트레이싱 지원 가능 하도록 기능 구현.
            -   가속화구조를 위한 동적 버텍스버퍼 추가.(레이 충돌에 사용되는 Position Only 지오메트리)
            -   레이트레이싱 쉐이더에서 ParticleSystem의 머티리얼 정보(위치, 색상, UV 등)를 가져올 수 있도록 동적버퍼(UAV) 추가 및 MeshParticle과 SpriteParticle의 버텍스펙토리 수정.

-   `HybridRendering`
    -   다량의 반투명 입자를 사용하는 ParticleSystem은 입자들이 겹치는 부분에서 아티펙트 발생.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXParticle.jpg?raw=true" width=200 height=200>
        -   레이 충돌검사 시 self intersection 방지를 위한 TMin값에 의해 발생하는 현상.
    -   다량의 반투명 입자를 사용하는 ParticleSystem을 레이트레이싱에 그대로 사용 할 경우 게임성능이 심각하게 저하.
        -   레이트레이싱에서 ParticleSystem당 표현가능 한 최대 입자의 수를 매우 적게 제한하여 해결.
        -   하지만 카메라에 직접 보이는 ParticleSystem의 입자를 줄일 경우 품질저하 발생.
    -   위의 문제들을 해결하기 위해 하이브리드 렌더링 사용.
        -   기존의 레스터화 렌더링과 레이트레이싱을 함께 사용.     
        -   카메라에 직접 보여지는 ParticleSystem의 입자들은 기존의 레스터화 방식으로 렌더링.
        -   레스터화 렌더링 이 후 적은수로 제한된 ParticleSystem입자를 사용해 반투명 레이트레이싱 반사 계산.   
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXHybrid.jpg?raw=true" width=200 height=200> 

-   `DepthFade`
    -   반투명 재질의 경우 SceneDepth표현식을 사용하는 경우가 있다.
    -   하지만 UE4 레이트레이싱에선 SceneDepth표현식을 사용 할 수 없다.
        -   레이트레이싱에선 카메라의 옆이나 뒤 처럼 Depth 버퍼에 기록 할 수 없는 부분도 계산해야 하기 떄문.
    -   UE4 반투명 레이트레이싱은 최적화를 위해 가장 가까운 불투명 표면까지의 거리를 계산 하여 레이의 길이를 조절한다.
        -   이 때 계산된 불투명 표면까지의 거리를 반투명 레이트레이싱 Payload에 적제하여 반투명 레이트레이싱 게산동안 사용할 수 있도록 수정.
        -   단 이때 계산된 거리는 SceneDepth와는 동일하지 않다.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXSceneDepth.png?raw=true" width=450 height=400> 
        -   https://github.com/djarksnd/RecentWork/blob/main/images/RTXSceneDepth.png


## Footprint
-   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintAnimation.gif?raw=true" width=300 height=175>
-   `Footprint`
    -   UE4 ShadowDepthRendering 코드를 참조하여 구현.
    -   PrimitiveComponent에 프로퍼티를 추가하여 Footprint를 남기는 FootprintCaster와 Footprint가 표면에 남게 되는 FootprintReceiver로 구분.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintProperty.jpg?raw=true" width=200 height=250>
    -   Footprint영역(AABBox)에 들어온 FootprintCaster와 FootprintReceiver 컬링.
        -   최적화를 위해 Octree와 Multithread(PC에선 Parallel For 활용, Mobile에선 MultiThread 사용하지 않음)활용.
    -   FootprintCaster와 FootprintReceiver를 2Pass로 나누어 렌더링.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintExp.png?raw=true" width=246 height=167>
        -   Depth & Stencil Test를 이용하여 FootprintCaster와 FootprintReceiver가 겹치는 영역 마스킹.
    -  마스킹결과를 이전 프레임의 FootprintMaskBuffer와 블렌딩하여 현재 프레임의 FootprintMaskBuffer생성.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintMask.png?raw=true" width=246 height=167>
    -  현재 프레임의 FootprintMaskBuffer를 이용하여 FootprintTangentBuffer 생성
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintNormalMap.png?raw=true" width=246 height=167>
    -  FootprintMaskBuffer와 FootprintTangentBuffer를 사용하여 최종 결과물 생성.    
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/Footprint.jpg?raw=true" width=246 height=167>
    -  아트팀에서 Footprint를 간단하게 사용 할 수 있도록 머티리얼 에디터에서 FootprintMask노드와 TransformFootprintTS노드를 제공.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintMaterialNode.png?raw=true" width=400 height=270>
