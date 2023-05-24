# README

최근작업한 내용인 Raytracing 과 Footprint 기능에 대한 간략한 개요 입니다.

작업내용은 모드 UE4를 수정하여 구현되었습니다.

## 인게임 구현 결과
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
    -   Reflection
        -   최적화를 위해 물체표면의 러프니스와 메탈릭 수치에 따라 픽셀당 광선의 반사 횟수를 0 ~ 3회 사이로 동적으로 변경.
        -   광선의 최대 반사 횟수를 넘을 경우 환경맵핑 수행.
        -   레이트레이싱에서 Additive Blend Mode 와 Modulate Blend Mode 를 표현 할수 있도록 수정.
        -   불투명표면과 반투명표면간 반사가 일어날 수 있도록 수정.
            -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXTranslucent.jpg?raw=true" width=600 height=250>    

-   `Global illumination`
    -   NVidia의 RTXGI플러그인 적용.
    -   https://developer.nvidia.com/rtx/ray-tracing/rtxgi
    -   BaseColor가 다소 어둡게 제작된 아트 에섯들이 GI에 효과적으로 영향을 주거나 받을 수 있도록 플러그인 코드수정.
    
-   `ParticleSystem with HybridRendering`
    -   ParticleSystem 레이트레이싱 지원
        -   UE4는 CascadeParticleSystem의 레이트레이싱을 지원하지 않음.
        -   따라서 게임에서 가장 많이 사용 되는 MeshParticle과 SpriteParticle이 레이트레이싱 지원 가능 하도록 기능 구현.
            -   가속화구조를 위한 동적 버텍스버퍼 추가.
            -   레이트레이싱 쉐이더에서 ParticleSystem의 머티리얼 정보(위치, 색상, UV 등)를 가져올 수 있도록 동적버퍼(UAV) 추가 및 MeshParticle과 SpriteParticle의 버텍스펙토리 수정.
    -   HybridRendering
        -   ㄹ

## Footprint
-   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintAnimation.gif?raw=true" width=300 height=175>
-   `구현`
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
