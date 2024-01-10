# README

-   최근작업한 기능들에 대한 간략한 개요 입니다.
-   UE4를 이용해 구현되었습니다.

## 구현 결과
-   `Raytracing` [PC Only]
    -   [Reflection & Global illumination], [ParticleSystem Raytracing]
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXAnimation.gif?raw=true" width=400 height=250><img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXParticleAnimation.gif?raw=true" width=400 height=250>

-   `Footprint (지형에 흔적 남기기)` [PC & Mobile]
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintAnimation.gif?raw=true" width=400 height=250>

-   `FoliageInteraction` [PC & Mobile]
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteraction.gif?raw=true" width=250 height=250>
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteractionRoar.gif?raw=true" width=300 height=250><img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteractionWhirlwind.gif?raw=true" width=300 height=250>

-   `ScreenSpaceAfterimage (화면공간 잔상효과)` [PC & Mobile]
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/SSAI_Skill_1.gif?raw=true" width=400 height=250>
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/SSAI_Skill_2.gif?raw=true" width=400 height=250>

-   `FocalShadow (캐릭터에 초점을 맞춘 그림자)` [PC & Mobile]
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FocalShadow_Intro.jpg?raw=true" width=700 height=250>
    
## Raytracing
-   `Reflection`
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXReflection.png?raw=true" width=900 height=150>    
    -   최적화를 위해 물체표면의 러프니스와 메탈릭 수치에 따라 픽셀당 광선의 반사 횟수를 0 ~ 3회 사이로 동적으로 변경되도록 수정.
    -   광선의 픽셀당 최대 반사 횟수를 넘을 경우 환경맵핑 수행.
    -   UE4 레이트레이싱에서 Additive Blend Mode 와 Modulate Blend Mode 를 표현 할수 있도록 수정.
    -   불투명표면과 반투명표면간 반사가 일어날 수 있도록 수정.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXTranslucent.jpg?raw=true" width=350 height=150>

-   `Global illumination`
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXGI.png?raw=true" width=600 height=150>    
    -   NVidia의 RTXGI플러그인 적용.
        -   https://developer.nvidia.com/rtx/ray-tracing/rtxgi
    -   BaseColor가 다소 어둡게 제작된 아트 에섯들이 GI에 효과적으로 영향을 받을 수 있도록 플러그인 수정.
    
-   `ParticleSystem`
    -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXParticleReflection.jpg?raw=true" width=150 height=270> 
    -   ParticleSystem 레이트레이싱 지원
        -   UE4는 CascadeParticleSystem의 레이트레이싱을 지원하지 않음.
        -   따라서 게임에서 가장 많이 사용 되는 MeshParticle과 SpriteParticle이 레이트레이싱 지원 가능 하도록 기능 구현.
            -   가속화구조를 위한 동적 버텍스버퍼 추가.(레이 충돌에 사용되는 Position Only 지오메트리)
            -   레이트레이싱 쉐이더에서 ParticleSystem의 머티리얼 및 입자의 정보(라이프타임, 위치, 색상, UV 등)들을 가져올 수 있도록 동적버퍼(UAV) 추가 및 MeshParticle과 SpriteParticle의 버텍스펙토리 수정.

-   `HybridRendering`
    -   다량의 반투명 입자를 사용하는 ParticleSystem은 입자들이 겹치는 부분에서 아티펙트 발생.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXParticle.jpg?raw=true" width=200 height=200>
        -   레이 충돌검사 시 self intersection 방지를 위한 TMin값에 의해 발생하는 현상.
    -   다량의 반투명 입자를 사용하는 ParticleSystem을 레이트레이싱에 그대로 사용 할 경우 게임성능이 심각하게 저하.
        -   레이트레이싱에서 ParticleSystem당 표현가능 한 최대 입자의 수를 매우 적게 제한하여 해결.
        -   하지만 카메라에 직접 보이는 ParticleSystem의 입자를 줄일 경우 품질저하 발생.
    -   위의 문제들을 해결하기 위해 하이브리드 렌더링 구현.
        -   기존의 레스터화 렌더링과 레이트레이싱을 함께 사용.     
        -   카메라에 직접 보여지는 반투명 ParticleSystem의 입자들은 기존의 레스터화 방식으로 렌더링(눈에 직접 보이는 부분).
        -   레스터화 렌더링 이 후 적은수로 제한된 ParticleSystem입자를 사용해 반투명 레이트레이싱 반사 계산(거울, 수면 등에 반사되는 부분).   
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXHybrid.jpg?raw=true" width=200 height=200> 

-   `SceneDepth & PixelDepth Expression`
    -   반투명 재질의 경우 SceneDepth 와 PixelDepth 표현식을 사용하는 경우가 많다.
    -   UE4 레이트레이싱에선 재질의 SceneDepth 와 PixelDepth 등의 스크린기반 표현식을 사용 할 수 없다.
        -   레이트레이싱에선 카메라의 옆이나 뒤 처럼 시야(스크린영역)를 벗어나는 부분도 계산해야 하기 때문.
    -   하지만 UE4 반투명 레이트레이싱은 최적화를 위해 가장 가까운 불투명 표면까지의 거리를 계산 하여 레이의 길이를 조절한다.
        -   이 때 계산된 불투명 표면까지의 거리를 반투명 레이트레이싱 Payload에 적제하여 반투명 레이트레이싱 계산동안 SceneDepth표현식으로 사용할 수 있도록 쉐이더코드에서 기존 표현식 레핑.
        -   PixelDepth 표현식은 레이트레이싱 쉐이더에서 HLSL의 RayTCurrent 함수를 사용하도록 쉐이더코드에서 기존 표현식 레핑.
        -   위의 표현식 레핑으로 기존의 머티리얼을 수정 없이 사용가능.
        -   단 이때 계산된 SceneDepth 와 PixelDepth 표현식의 값은 레스터화 렌더링에서의 값과 약간의 차이가 있다.
            -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXSceneDepth.png?raw=true" width=600 height=400> 
    -   수정 내용을 적용해 반투명 레이트레이싱에서 SceneDepth와 PixelDepth 표현식을 이용한 DepthFade 처리(수면과 지면이 닿는 부분).
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/RTXTranslucentFade.jpg?raw=true" width=300 height=200>    


## Footprint
-   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintAnimation.gif?raw=true" width=300 height=175>
-   `Footprint`
    -   UE4 ShadowDepthRendering 코드를 참조하여 구현.
    -   PrimitiveComponent에 프로퍼티를 추가하여 Footprint를 남기는 FootprintCaster와 Footprint가 표면에 남게 되는 FootprintReceiver로 구분.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FootprintProperty.jpg?raw=true" width=200 height=250>
    -   컬링거리에 들어온 FootprintCaster와 FootprintReceiver 컬링.
        -   최적화를 위해 Octree와 Multithread(PC에선 Parallel For 활용, Mobile에선 MultiThread 사용하지 않음)활용.
    -   FootprintCaster와 FootprintReceiver를 2 Pass로 나누어 렌더링.
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

## FoliageInteraction
-   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteraction.gif?raw=true" width=250 height=250><img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteractionRoar.gif?raw=true" width=300 height=250><img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteractionWhirlwind.gif?raw=true" width=300 height=250>
-   `FoliageInteraction`
    -   FoliageInteractionComponent 클래스를 제작하여 구현.
    -   ShadowDepthPass 이전(폴리지의 흔들림이 적용된 그림자를 그리기 위해 ShadowDepthPass 이전에 렌더링) FoliageInteractionBufferPass 를 추가하여 FoliageInteractionComponent의 정보(FoliageInteractionSceneProxy)를 탑뷰 시점에서 렌더링하여 FoliageInteractionBuffer 생성. 
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteractionBuffer.jpg?raw=true" width=550 height=250>
    -   Material에서 FoliageInteraction노드(전용 MaterialExpression 추가)를 이용해 FoliageInteractionBuffer의 정보를 가져와 폴리지의 움직임을 시각적으로 구현.
         -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteractionNode.jpg?raw=true" width=500 height=168>
    -   상호작용 강도와(Force) 형태를(Direction Intensity, Angle) 쉽게 제어할수 있도록 구현.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteractionComponent.jpg?raw=true" width=280 height=200>
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FoliageInteractionProps.png?raw=true" width=400 height=500>

## ScreenSpaceAfterimage
-   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/SSAI_Skill_1.gif?raw=true" width=350 height=200>
-   `ScreenSpaceAfterimage`
    -   UE4의 CustomDepthStencil기능 과 Postprocess를 활용하여 구현.
    -   PrimitiveComponent에 DrawScreenSpaceAfterimage 프로퍼티를 추가하여 잔상을 남길지 여부 판단.
    -   CustomDepthStencil기능을 이용해 특정 시간마다(0.333초) DrawScreenSpaceAfterimage가 활성화된 PrimitiveComponent의 Stencil 기록.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/SSAI_Stencil.jpg?raw=true" width=300 height=200>
    -   Stencil이 기록된 부분에 해당하는 픽셀을 SceneTexture에서 읽어와 이전 프레임의 ScreenSpaceAfterimage 버퍼에 복사.
    -   시간에 따른 잔상의 FadeOut처리를 위해 2장의 ScreenSpaceAfterimageBuffer(Current And Prev)를 매 프레임 서로 스왑하여 사용.
    -   매 프레임 마다 이전 프레임의 ScreenSpaceAfterimageBuffer에서 픽셀을 읽어 DeltaTime을 이용해 FadeOut처리를(AlphaBlend) 한후 현재 프레임의 ScreenSpaceAfterimageBuffer에 기록.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/SSAI_AttBuffer.jpg?raw=true" width=300 height=200>
    -   현재 프레임의 ScreenSpaceAfterimageBuffer와 SceneTexture를 섞어 최종결과물 생성.
        -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/SSAI_Result.jpg?raw=true" width=300 height=200>

## FocalShadow
-   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FocalShadow_Intro.jpg?raw=true" width=700 height=250>
-   `구현 아이디어`
    -   게임을 플레이중인 유저의 시점은 게임캐릭터주위 또는 화면중앙에 머무르게 된다.
    -   따라서 게임캐릭터주위 또는 화면중앙의 그림자만 더 뚜렷하게 표현할 수 있다면 ShadowDepthMap의 해상도를 올리지 않고 그림자의 품질을 높일 수 있다.
-   `구현 방법`
    -   기존의 CSM 방식은 뷰프러스텀 영역을 거리에 따라 나누어 ShadowDepthMap을 표현한다.
          -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/CSM_Normal.jpg?raw=true" width=400 height=300>
    -   기존의 CSM 방식을 수정하여 첫 번째 CSM(CSM 1)은 초점영역(게임캐릭터 주위 또는 화면중앙)만 표현하여 ShadowDepthMap이 표현해야 할 범위를 좁혀 ShadowDepthMap을 더 뚜렷하게 그린다.      
          -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/CSM_Focal.jpg?raw=true" width=350 height=300>
    -   그림자를 렌더링 하는 쉐이더코드를 수정하여 초점영역(게임캐릭터 주위 또는 화면중앙) 주위픽셀은 첫 번째 CSM에서 ShadowDepth를 읽어와 그림자를 처리하면 초점영역(게임캐릭터 주위 또는 화면중앙)의 그림자를 더 뚜렷하게 표현할 수 있다.
    -   단 초점영역(게임캐릭터 주위 또는 화면중앙)을 벗어난 부분은(CSM 2) CSM이 표현해야할 범위가 넓어지기에 기존 CSM방식보다 그림자가 조금 뭉개진다. 
    -   아래의 이미지는 초점영역(게임캐릭터 주위 또는 화면중앙)을 시각화한 모습이다.
          -   <img src="https://github.com/djarksnd/RecentWork/blob/main/images/FocalShadow_Sphere.jpg?raw=true" width=420 height=300> 

            
